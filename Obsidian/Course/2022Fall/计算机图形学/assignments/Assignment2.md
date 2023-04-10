# 2053764 吴俊成
## 1. Proof the composed transformations defined in global coordinate frame is equivalent to the composed transformations defined in local coordinate frame but in different composing order.

证明：
	使用数学归纳法，变化之前某一点的位置为$p$，假设一共进行n次变换，之后的位置向量为$p_n$，
	每次的变换分别为$T_1$、$T_2$...... $T_n$，这里的变换在世界坐标系下与局部坐标系下可能对应不同的矩阵，但具有同样的语义。比如：都表示在当前坐标系下顺时针旋转30度，或者在当前坐标系下进行某一特定的Translation，由于坐标系不同，因此实际上对应的矩阵数值不同。
	1. n=1
		世界坐标系下：$p_1=T_1p$
		局部坐标系下：$p_1=T_1p$
	2. n=2
		世界坐标系下：$p_2=T_2T_1p$
		局部坐标系下：
		（1）首先进行第一次变换，$p_1=T_1p$
		（2）接下来进行第二次变换，由于局部坐标系已经改变，因此需要修正回原来的坐标系（也就是进行$T_1$的逆变换$T_1^{-1}$，再进行新的变换，之后再将坐标系修正回来，也就是再次进行$T_1$变换，这个三个变换可以合并为$(T_1T_2T_1^{-1})$，因此最终在局部坐标系下的变换为$p_2=(T_1T_2T_1^{-1})T_1p=T_1T_2p$ 
	3. n>2
		世界坐标系下：$p_n=T_nT_{n-1}p_{n-2}=T_nT_{n-1}\cdots T_1p$  
		局部坐标系下：与之前同理，$p_n=T_{n-1}T_np_{n-2}, p_{n-2}=(T_1T_2\cdots T_{n-2})p$，令$T_a=T_{n-1}T_n, T_b=T_1T_2\cdots T_{n-2}$ ，由于在未进行任何变换之前，世界坐标系与局部坐标系是重叠的，因此以上变化相当于在世界坐标系下，在p上先进行变化$T_b$再进行变化$T_a$，与n=2的情况相同，此时在局部坐标系下$p_n=T_bT_ap=T_1T_2\cdots T_np$
	证毕 
## 2. Describe the differences between orthographic and perspective 3D viewing processes? (Draw the view volume of the above two viewings)
正则3D viewing过程会忽视顶点的深度信息，将几何体直接投射到一个长方形的视景体中，进行clipping操作以及正则标准化过程后（Orthogonal normalization）之后得到NDC，因此，几何体中各个线段之间的逻辑关系（平行、垂直等）在最终的渲染结果中同样存在，不会随着camera的位置改变而发生变化，因此适合在CAD工程制图等软件中使用。正则标准化通过正则矩阵完成，可以理解为Translation和Scale两个步骤，首先通过Translation进行中心对齐，接下来在几何体的每个轴上进行缩放，得到最终的NDC。其视景体如下图所示：![[微信图片_20221020203606.jpg]]
透视3D viewing过程会考虑顶点的深度信息，其视景体是一个四棱台，首先通过标准透视变换将几何体投射到视景体中，进行clipping操作，之后进行透视除法，转化到NDC空间。最后渲染的结果符合近大远小的物理规律，适合用在游戏、电影等需要考虑建模真实性的场景中，其视景体如下图所示：![[微信图片_20221020203613.jpg]]
## 3. Which one defines the default NDC? Why?
$glm::ortho(-1., 1., -1., 1., -1., 1.)$是默认的NDC，由于在NDC空间中默认的坐标系为左手系，与Graphic Pipeline中其他绝大部分情况采用右手系不同。因此深度轴z的指向far方向，因此默认的NDC中near为-1.0，而far为1.0
## 4. What is the difference between the clip space and NDC?
裁切空间（clip space）是在顶点进行投影之后所在的空间（在opengl中，vertex shader的输出就在clip space中），在这里采用四元齐次坐标系。GPU需要在裁切空间中裁切掉Clip Space范围之外的顶点，接下来GPU会进行透视除法，将四元坐标除以w，转化到笛卡尔坐标，进入NDC空间。NDC空间的意义就是顶点最后要输出到的屏幕空间
## 5. Why does clipping performed in the clip space?
cliping的执行时间可以有三种选择，第一种是在进行投影变化（Projection transform）之前，需要使用视景体几个面的方程判断顶点是否需要被裁切，比较符合人们的认知，但相对而言计算量很大，难以实现。第二种是在透视除法之后，在NDC之中进行裁切，可这样的问题在于如果存在深度小于或者等于摄像机位置的顶点，会导致在进行透视除法的过程中出现除数为0的情况，或者导致NDC中包含位于摄像机后方的顶点，这部分顶点不应该被显示。第三种方法就是在clip space中进行裁切，之后再进行透视除法转化到NDC空间，综合考虑正确性、实现难度和效率，最终采用在clip space中进行clipping的方式。
## 6. What is the cause of Z-fighting? And can we solve the Z-fighting?
原因：
	确定了顶点的深度之后，需要内插顶点之间点的深度信息，理论上而言这样内插的点应该均匀，但由于距离不同，会导致实际上在靠近camera处内插更为密集，而远离camera处则较为稀疏，使深度与真实情况存在一定差距。这样的差距加上z-buffer的精度问题，导致在很靠近或者很远离一个物体时，前后表面的深度信息会差距很小，甚至被遮挡的物体会略微更靠近camera，导致本来应该被遮挡的物体被渲染到屏幕上，导致z fighting的出现。
解决：
	可以通过设置更高精度级别的z缓冲，或者避免在投影过程中near与far之间的差距过大导致深度信息的精度产生较大差异。也可以在建模阶段避免将模型之间靠得太近，进一步减少z frighting出现的可能性。
## 7. Programming
实现了简单的太阳系，包括以下功能：
* 简单的天空盒
* 各个星球的纹理贴图
* 增加了简单的光照
* 使用wasd以及鼠标可控制视角移动
![[Pasted image 20221030160303.png]]
