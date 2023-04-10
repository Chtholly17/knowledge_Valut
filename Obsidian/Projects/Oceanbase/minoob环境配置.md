# gitclone
gitclone中出现的问题：![[Pasted image 20221018011202.png]]
采用本方法，无法成功clone submodlue
解决方案为先clone miniob，之后进入deps![[Pasted image 20221018011256.png]]
仅clone miniob后，deps为空，这时去找到对应的module，手动clone到deps下![[Pasted image 20221018011404.png]]
# Build
之后分别进入每个子目录进行cmake和make![[Pasted image 20221018011525.png]]三个submodule make结束后，对miniob进行build即可![[Pasted image 20221018011552.png]]
- mac : unkonwn identifier append_history()
宏定义未知原因失效，解决方法为在对应源码中注释掉可能出现问题的部分
怀疑是mac系统中的重复定义，或者mac的readline中没有对应函数![[Pasted image 20221114190525.png]]
# Test
进入build目录下的bin文件夹，执行observer程序，使用参数-f指定初始化文件observer.ini（放置在minion/etc中。![[ca72ab390dad33e230896a9ca293037.png]]
（新建一个终端）打开客户端，同样在bin目录下，执行obclient程序，尝试输入一些命令：![[f2a84530e5d4525880b6b2c8b49d970.png]]