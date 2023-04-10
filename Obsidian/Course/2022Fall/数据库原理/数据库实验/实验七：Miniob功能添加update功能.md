# 分析
- 首先，查看其他指令的执行函数（比如insert与delete等），发现主要由executor完成，根据当前的操作类型，可以选择运行的函数，当前处理udate的函数为空
```cpp
switch (stmt->type()) {  
case StmtType::SELECT: {  
  do_select(sql_event);  
} break;  
case StmtType::INSERT: {  
  do_insert(sql_event);  
} break;  
case StmtType::UPDATE: {  
  //do_update((UpdateStmt *)stmt, session_event);  
} break;  
case StmtType::DELETE: {  
  do_delete(sql_event);  
} break;  
default: {  
  LOG_WARN("should not happen. please implenment");  
} break;  
}
```
## executor.do_delete()
- UPDATE操作与DELETE操作具有相似性，都需要使用where语句确定某一特定的记录，因此首先参考delete函数的实现
- 以下部分为executor的固定入口参数，输入之后可以获取一系列状态量
```cpp
RC ExecuteStage::do_delete(SQLStageEvent *sql_event)  
{  
  //update operation has similar parameter
  //get needed instance
  Stmt *stmt = sql_event->stmt();  
  SessionEvent *session_event = sql_event->session_event();  
  Session *session = session_event->session();  
  Db *db = session->get_current_db();  
  Trx *trx = session->current_trx();  
  CLogManager *clog_manager = db->get_clog_manager();  
  
  //nullptr error processing
  if (stmt == nullptr) {  
    LOG_WARN("cannot find statement");  
    return RC::GENERIC_ERROR;  
  }  
```
- 创建一个delete_stmt，该类继承于stmt，可以从其中取得table与filter，为了工程性，我们也需要为update操作创建一个update_stmt
```cpp
  //a stmt for update is also needed
  DeleteStmt *delete_stmt = (DeleteStmt *)stmt;  
  TableScanOperator scan_oper(delete_stmt->table());  
  PredicateOperator pred_oper(delete_stmt->filter_stmt());  
  //条件过滤操作有一个子操作（扫描整个table）
  pred_oper.add_child(&scan_oper);  
  ```
- 创建一个delete_oper，关键的删除操作在此中完成（open中），并且为其添加一个子操作，就是刚才的条件过滤（predicate操作）,我们也需要添加一个类似的update_oper，同样需要具有添加子操作和open的功能
```cpp
  //a opertor for update is also needed
  DeleteOperator delete_oper(delete_stmt, trx);  
  delete_oper.add_child(&pred_oper);  
  RC rc = delete_oper.open();  
```
- 以下操作就是根据delete操作的返回状态rc，决定输出到屏幕的提示，update操作可以采用类似的处理
```cpp
  if (rc != RC::SUCCESS) {  
    session_event->set_response("FAILURE\n");  
  } else {  
    session_event->set_response("SUCCESS\n");  
    if (!session->is_trx_multi_operation_mode()) {  
      CLogRecord *clog_record = nullptr;  
      rc = clog_manager->clog_gen_record(CLogType::REDO_MTR_COMMIT, trx->get_current_id(), clog_record);  
      if (rc != RC::SUCCESS || clog_record == nullptr) {  
        session_event->set_response("FAILURE\n");  
        return rc;  
      }  
  
      rc = clog_manager->clog_append_record(clog_record);  
      if (rc != RC::SUCCESS) {  
        session_event->set_response("FAILURE\n");  
        return rc;  
      }   
  
      trx->next_current_id();  
      session_event->set_response("SUCCESS\n");  
    }  
  }  
  return rc;  
}
```
## delete_oper类
### 成员变量stmt与trx
- 从executor中继承，需要进行类型转换，转换为delete_stmt，trx直接传入即可
```cpp
public:  
  DeleteOperator(DeleteStmt *delete_stmt, Trx *trx)  
    : delete_stmt_(delete_stmt), trx_(trx)  
  {}
  
private:  
  DeleteStmt *delete_stmt_ = nullptr;  
  Trx *trx_ = nullptr;
```
### 1、add_child()
- 继承自opertor类，无需自己重新定义
```cpp
void add_child(Operator *oper) {  
  children_.push_back(oper);  
}
```
### 2、open()
- 进行子操作，对子操作数量不对、子操作执行open失败的情况需要给出错误提示
```cpp
RC DeleteOperator::open()  
{  
  if (children_.size() != 1) {  
    LOG_WARN("delete operator must has 1 child");  
    return RC::INTERNAL;  
  }  
  
  Operator *child = children_[0];  
  RC rc = child->open();  
  if (rc != RC::SUCCESS) {  
    LOG_WARN("failed to open child operator: %s", strrc(rc));  
    return rc;  
  }
```
- 从stmt中获取table，管理当前的表
```cpp
Table *table = delete_stmt_->table();
```
- 由于child操作是条件过滤操作，因此child->next()就是获取下一个符合要求的tuple，假如获取错误需要输出信息，update操作的child也是条件过滤，因此可以采用相同的写法
```cpp
while (RC::SUCCESS == (rc = child->next())) {  
  Tuple *tuple = child->current_tuple();  
  if (nullptr == tuple) {  
    LOG_WARN("failed to get current record: %s", strrc(rc));  
    return rc;  
  }
```
- 进行以下的语句，获取对应tuple的record
```cpp
  RowTuple *row_tuple = static_cast<RowTuple *>(tuple);  
  Record &record = row_tuple->record();
```
- 获取record之后，可以在table中调用delete_record()方法，删除这个记录，并且对这个操作是否成功进行检查，如果失败着输出信息。
```cpp
rc = table->delete_record(trx_, &record);  
    if (rc != RC::SUCCESS) {  
      LOG_WARN("failed to delete record: %s", strrc(rc));  
      return rc;  
    }  
  }  
  return RC::SUCCESS;  
}
```
## table类
### 1、delete_record()方法
- 这部分涉及index的删除，与update操作没太大关系
```cpp
RC Table::delete_record(Trx *trx, Record *record)  
{  
  RC rc = RC::SUCCESS;  
  rc = delete_entry_of_indexes(record->data(), record->rid(), false);  // 重复代码 refer to commit_delete  if (rc != RC::SUCCESS) {  
    LOG_ERROR("Failed to delete indexes of record (rid=%d.%d). rc=%d:%s",  
                record->rid().page_num, record->rid().slot_num, rc, strrc(rc));  
    return rc;  
  }
```
- 这一部分，调用recordhandler（是一个RecordFileHandler，用于管理record文件），进行delete操作，若失败着输出错误提示
```cpp
rc = record_handler_->delete_record(&record->rid());  
if (rc != RC::SUCCESS) {  
  LOG_ERROR("Failed to delete record (rid=%d.%d). rc=%d:%s",  
              record->rid().page_num, record->rid().slot_num, rc, strrc(rc));  
  return rc;  
}
```
- 下面这一块先不管，设计trx与clog，实现事件回滚和日志记录，应该不会影响update功能，不深入研究了
```cpp
if (trx != nullptr) {  
  rc = trx->delete_record(this, record);  
  CLogRecord *clog_record = nullptr;  
  rc = clog_manager_->clog_gen_record(CLogType::REDO_DELETE, trx->get_current_id(), clog_record, name(), 0, record);  
  if (rc != RC::SUCCESS) {  
    LOG_ERROR("Failed to create a clog record. rc=%d:%s", rc, strrc(rc));  
    return rc;  
  }  
  rc = clog_manager_->clog_append_record(clog_record);  
  if (rc != RC::SUCCESS) {  
    return rc;  
  }  
}
```
## recordFileManager类
### 1、updateRecord方法
- 这个类中除了deleteRecord方法外，居然也有updateRecord()方法，阅读这一部分源码，发现其只不过是将记录的内容进行重现写入，没有进行任何更新
- 但我们可以参考
```cpp
RC RecordPageHandler::update_record(const Record *rec)  
{  
  if (rec->rid().slot_num >= page_header_->record_capacity) {  
    LOG_ERROR("Invalid slot_num %d, exceed page's record capacity, page_num %d.",  
         rec->rid().slot_num, frame_->page_num());  
    return RC::INVALID_ARGUMENT;  
  }  
  
  Bitmap bitmap(bitmap_, page_header_->record_capacity);  
  if (!bitmap.get_bit(rec->rid().slot_num)) {  
    LOG_ERROR("Invalid slot_num %d, slot is empty, page_num %d.",  
         rec->rid().slot_num, frame_->page_num());  
    return RC::RECORD_RECORD_NOT_EXIST;  
  } else {  
    char *record_data = get_record_data(rec->rid().slot_num);  
    memcpy(record_data, rec->data(), page_header_->record_real_size);  
    bitmap.set_bit(rec->rid().slot_num);  
    frame_->mark_dirty();  
    // LOG_TRACE("Update record. file_id=%d, page num=%d,slot=%d", file_id_, rec->rid.page_num, rec->rid.slot_num);  
    return RC::SUCCESS;  
  }  
}
```
### 2、insertRecord方法
- 真正需要写入数据的方法应该是数据的插入，它提供了一个char* 用来指向需要写入的数据。
- 以下部分为进行内存管理，找到空闲位置，这部分在update中不用管
```cpp
RC RecordPageHandler::insert_record(const char *data, RID *rid)  
{  
  if (page_header_->record_num == page_header_->record_capacity) {  
    LOG_WARN("Page is full, page_num %d:%d.", frame_->page_num());  
    return RC::RECORD_NOMEM;  
  }  
  
  // 找到空闲位置  
  Bitmap bitmap(bitmap_, page_header_->record_capacity);  
  int index = bitmap.next_unsetted_bit(0);  
  bitmap.set_bit(index);  
  page_header_->record_num++;
```
- 以下部分，将data写入file中对应的位置，update的操作应该类似于这个
```cpp
// assert index < page_header_->record_capacity  
char *record_data = get_record_data(index);  
memcpy(record_data, data, page_header_->record_real_size);  
  
frame_->mark_dirty();
```
- 接下来的部分，设置page_num与slot，update不用管，返回RC::SUCCESS即可
### deleteRecord方法
- 主要是参考如何找到对应的记录，update采用的方法应该相同，以下为初始化bitmap并检查slot是否合理
```cpp
if (rid->slot_num >= page_header_->record_capacity) {  
  LOG_ERROR("Invalid slot_num %d, exceed page's record capacity, page_num %d.",  
     rid->slot_num, frame_->page_num());  
  return RC::INVALID_ARGUMENT;  
}  
  
Bitmap bitmap(bitmap_, page_header_->record_capacity);
```
- 取得bit并进行校验，假设函数返回0说明错误读取，需要给出错误提示
```cpp
bitmap.get_bit(rid->slot_num)
```
```cpp
else {  
  LOG_ERROR("Invalid slot_num %d, slot is empty, page_num %d.",  
     rid->slot_num, frame_->page_num());  
  return RC::RECORD_RECORD_NOT_EXIST;  
}
```
- 如果以上校验通过，直接使用slot_num作为index，get record_data之后使用insert相同的方式写入并返回成功标志即可
- 最后的问题：insert record中的char * data是如何获取的？可以直接调用record的data方法获取即可。

