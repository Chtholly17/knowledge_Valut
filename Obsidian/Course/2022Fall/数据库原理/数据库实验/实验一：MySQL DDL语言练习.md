# 查看表的结构：
使用下列查询语句
```mysql
DESC table1
```
# 查询一个表上所有的约束
查看当前table中存在的所有约束：
```Mysql
SELECT table_name, constraint_name, constraint_type
FROM information_schema.TABLE_CONSTRAINTS
WHERE table_name=<表名>;
```

