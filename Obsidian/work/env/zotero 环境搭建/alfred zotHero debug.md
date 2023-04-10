- 无法显示搜索结果，但是可以找到文章，说明文件路径是不存在问题的![[Pasted image 20230327155102.png]]
- 就是这里出了问题，继续看traceback，最后发现是这里出错了![[Pasted image 20230327155659.png]]
- 看self.entry
```python
def entry(self, key):
	"""Return Entry for key."""
	sql = ITEMS_SQL + 'AND key = ?'
	row = self.conn.execute(sql, (key,)).fetchone()
	if not row:
		return None
  
	return self._load_entry(row)
```
果然，返回了NONE，触发了相应的输出，可能是由于未知原因，输入了不存在的KEY导致的，确实，在storage文件夹中也没有发现这个key![[Pasted image 20230327155949.png]]
为什么会这样呢？不想仔细研究了，可能和zotero的数据库管理机制有关，感觉是无底深坑啊
干脆先这样改，看看能不能对![[Pasted image 20230327160322.png]]
出新错误了![[Pasted image 20230327160406.png]]
那干脆在返回None的时候什么也不做![[Pasted image 20230327160506.png]]
解决了！![[Pasted image 20230327160559.png]]

