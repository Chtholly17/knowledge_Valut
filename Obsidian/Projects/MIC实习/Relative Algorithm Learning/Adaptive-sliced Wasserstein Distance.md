假设SWD进行N次投影，第i次投影计算的SWD为$$sw_i=W^p_p(\pi _{\theta_i}\# \mu,\pi _{\theta_i}\# v) $$
使用蒙特卡洛方法，进行N次从某一分布上随机采样的方向进行投影，计算的SWD为$$\overline {sw}_N = \frac{1}{N}\sum \limits^N \limits_{i=1}(sw_i-\overline{sw}_N)^2$$
![[Pasted image 20230306210232.png]]
ASW就是在保证以下不等式不成立的情况下，每次增加s次蒙特卡洛采样并更新结果![[Pasted image 20230306210350.png]]
	其中，k的值设定为2，表示一下关系成立，也就是估计的偏差小于一定值（标准正态分布-k到k的概率），epsilon可以看作对容忍率的调整，是一个常数
	![[Pasted image 20230306210500.png]]![[Pasted image 20230306210435.png]] ![[Pasted image 20230306210443.png]]
最终整体的算法如下：最多可以进行M次采样![[Pasted image 20230306210716.png]]