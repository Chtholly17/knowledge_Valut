# 概要
## 背景
- nearest neighbor searching：
	- set size N is large
	- similarity evaluation is expensive
	- solution：ANN
- ANN：approximate nearest neighbor，a method to learning from set-structured data
	- distance calculating between sets
	- data structure for nearest neighbor searching
- LSH：Locality Sensitive Hashing
	- H：a set of Hash Function（Rd->U)，U代表hash值的空间
	- (R,c,P1,P2)-sensitive：对于任意的d维度向量uv，选取H中的任意一个Hash Function h，都满足以下两条性质：![[Pasted image 20221201164801.png]]
		- 其中P1>P2，c>1
	- 原本的LSH，使用Euclidean Distance测量uv的距离，采用以下方程进行hash：![[Pasted image 20221201165305.png]]
		- 其中a是一个从p-stable distribution（比如正态分布）中随机采样的向量，b的范围在[0,omega]（位于hash backet中），omega是hash bucket的宽度（hash bucket可以理解为哈希值可能存在的区间？）
	- uv hash value相等的概率为：![[Pasted image 20221201164931.png]]
		- 其中，r是uv之间的Lp距离，fp(t)代表用于采样a的p-stable distribution的density function
	- 我们想要P1与P2之间的差距尽量大，通常随机在H中选取k个hash函数进行计算，并结合他们的结果，使用以下的g作为最终的hash函数：                  ![[Pasted image 20221201203734.png]]
	- 实际应用中，可能使用多个上述的g（g1、g2……gT），但uv之间的distance为R（此处的distance 为 Euclidean Distance），uv在以上T个hash函数中任意至少一个中发生冲突的概率为                                                                                      ![[Pasted image 20221201204127.png]]
- GSW：![[Pasted image 20221201175410.png]]
	- 投影
	- 计算对两个一维序列l2 distance
	- 参数空间上积分
	- 蒙特卡洛方法![[Pasted image 20221201175831.png]]
- SWE：Sliced Wasseretein Embedding![[Pasted image 20221201161805.png]]
- reference Set：用于embedding，对于不同的input set，采用相同的reference Set，其通常具有确定的分布,在具体应用中具有不同的取值方式
- Input Set：包含结构特征的分布，比如SWD中投影到低维的点集。
- 首先将input与reference投射之后升序排序
- 计算input set与reference之间的monge coupling（可以看作OT问题中的transport plan），用以下公式计算，这一步可以看作在解决ANN问题                             ![[Pasted image 20221201201421.png]]
	- 其中，F表示累积分位函数（CDF），Fu(x)表示在概率分布u中，数值小于等于x的概率之和
	- F-1是CDF的反函数，但CDF连续时（本情况下符合此假设），认为F-1是分位函数，也就是，以下公式中Q代表F-1                                 ![[Pasted image 20221201201733.png]]
	- reference set的CDF如下，pi代表排序，pi-1代表排序前对应的元素                                            ![[Pasted image 20221201202016.png]]
	- 综上所述，以上公式的含义为：对于reference set中的每一点，计算其对于reference distribution的CDF，我们将这个值称为a，将a带入input set distribution的分位函数，可以在input set中找到唯一一个点与reference set中的点（下标为m）对应
	- 我们计算这两个点之间的L1距离。                    ![[Pasted image 20221201202157.png]]
	- 以上过程重复L次，每次产生一个长度为M（M为reference set中元素的个数）的向量，我们依次将他们连接在一起          ![[Pasted image 20221201202252.png]]
	- 最终![[Pasted image 20221201202310.png]]就是input set embedding的结果，当两个input set采用同样的reference进行embedding时，我们可以认为两个embedding之间的距离就是两个input set之间的GSW![[Pasted image 20221201202457.png]]
	- 以上公式采用积分的形式，在投影参数空间内积分，实际上我们采用蒙特卡洛方法来近似这个结果![[Pasted image 20221201202610.png]]
## 问题
- many ANN methods are designed for objects living in Hilbert Space，但在现在的很多应用场景中，输入的data不是一个set（point cloud processing），此时需要将输入数据分解为set
- 现有的方法采用Hungarian Algorithm或者Wasserstein Distance，时间复杂度在二次或者三次
- SWE实际上已经解决了以上两个问题，本文提出的SLOSH方法利用了SWE
## 贡献
- SLOSH：使用SWD进行embedding
	- data- independent
	- non-parametric
# 具体工作
- SLOSH：
	- 将输入的图片看作以下向量的集合（将图片分割为多个patch）![[Pasted image 20221201205140.png]]
	- 采用首先对set进行SWE，使用GSW代替经典LSH中的Euclidean距离，这样，对于一个Hash函数集合H，假设它满足(R,c,P1,P2)-sensitive性质，那么就有（其中phi(Xi)是embedding的结果，一个长度为LM的向量）：![[Pasted image 20221201204426.png]]
		- 就像之前提到的，我们会采用k个hash函数组成一个序列作为最终的hash结果，如下，一个输入的structred set hash的结果是长度为k的向量![[Pasted image 20221201204558.png]]
		- 假设使用T个g函数生成hash code，两个input set发生冲突的概率为![[Pasted image 20221201205913.png]]
# 实验
- FSPooling：Data dependent，可以看做本文方法的一种特殊情况（L=d，且Theta L被当作一个identity matrix，Theta L是L个投影向量的集合                                   ![[Pasted image 20221201205532.png]]
- 对所有方法，统一hash code的长度为1024
- 采用Point cloud MINIST 2D、ModelNet40、Oxford Building Dataset数据集
- Precision@K：即预测正确的相关结果占返回的所有结果的比例![[Pasted image 20221201210651.png]]
	- 其中k表示：只考虑预测结果中概率前k大的数值[[https://blog.csdn.net/zgqj1/article/details/89430374]]
## 定量结果
- 对hash值进行retrieve，计算ANN（具体方法不清楚），预测结果的Precision@K结果如下：![[Pasted image 20221201211220.png]]
## 定性结果
对于L>d情况下，在上个dataset retrieve的结果![[Pasted image 20221201211421.png]]
## 消融实验
- 对hash code长度的研究，可以看到SWE表现很好，并且128左右是最合适的编码长度![[Pasted image 20221201211508.png]]
- Slice Num（随机投影的数目L）对SLOSH效果的影响，过多的L不一定会提升hash的效果![[Pasted image 20221201211746.png]]
- reference set X0：随机取值，从上到下分别为：![[Pasted image 20221202151454.png]]
	- 对数据集中所有sets进行Kmeans计算得到
	- dataset中一个随机的set（可以看成其中的一张图片）
	- 从各个sets中均匀分布取得（可能是每个sets中取了几个set）
	- 从各个set中正态分布取得
- 都随着reference set长度的增加效果有所上升，上升到64左右后继续上涨效果提升不明显
# 启发
- 本文提出的是一种embedding并hash的算法，得到的结果进行retrieve后的precision也许一定程度上反映了hash或embedding的结果对原来的structured set中distribution信息保留的程度
- 从第三个消融实验中，我们可以发现：对于离散点的集合，采用k-means与正态分布这一类更具有统计意义的采样方式获取的reference set，更能体现出离散的structure。但是对于真实的图像（Oxford Building Dataset），本身每张图片的patch就包含了自己的分布信息，因此采用uniform distribution采样或者以某个sets作为reference的方法就更能在结果中保留distribution的信息，我们是否也可以采用一个reference set先进行embedding再计算set？这个reference set是否可以采用与图片有关，采用图片down sample或者blurring后的结果作为reference