- 不要在迭代访问容器的过程中对其进行增删操作，可能会使迭代器错误，以下代码就存在这个问题
```cpp
for(auto k:update){
if(k.second> currentTime){
	cnt++;
}
else{
	tmp.push_back(k.first);
}

```