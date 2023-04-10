- 如何确定SWD计算时每张图片分解的patch（每个patch分别投影到一维，一同组成一个一维的分布）
- SWD投射的过程中，由于进行了随机的向低维投射，在此过程中丢失了位置信息，当输入的图片的distribution与ground truth差别过大时，使用梯度下降的方法可能导致生成的图片上的texture具有不同的位置。
## scale的确定
- 如何修正这一问题？也许可以考虑改变图像分解是的sclae，也就是更改patch的大小，
	- 越大的patch关注更global的信息，感受野越大。但是由于global的信息会导致local texture信息的丢失，导致局部特征的不准确
	- 更小的patch，可以获取更local 的特征，但太小会对geometry difference过于敏感，在现实问题中并不适用。
## WD的projection过程，投影向量的确定
- 找出最合适的投影方向
- MAX-WD：梯度下降选择使projection产生的结果之间distance最大的投影向量
- WD：多次投影，取平均
- MAX的Distance不一定最好，能否优化出最佳的Projection产生最合适的Distance？
## 可以进行的实验
![[Pasted image 20221122152002.png]]