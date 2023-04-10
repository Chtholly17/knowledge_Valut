- 参考资料
	- [[https://blog.csdn.net/Mrqiang9001/article/details/80820321]]
	- https://cloud.tencent.com/developer/article/1679861
- 首先查看虚拟机的IP地址，由于使用NAT模式，虚拟机和本机的ip地址应该是一样的，保险起见以后还是在虚拟机上查看![[Pasted image 20230329002827.png]]
- 然后设置NAT主机映射，流程可以参考[[https://blog.csdn.net/Mrqiang9001/article/details/80820321]]
	- 需要注意，这一步虚拟机ip地址需要用刚才查到的![[Pasted image 20230329003013.png]]
	- 端口，虚拟机和主机都填写22，在虚拟机sshd_config中可以设置虚拟机监听的端口号，但是默认是22，为了避免麻烦就不改了
- 这样正常来说用以下指令就可以连接了
```bash
ssh -p 22 username@localhost
```