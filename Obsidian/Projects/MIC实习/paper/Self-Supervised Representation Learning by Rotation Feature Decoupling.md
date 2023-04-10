# 概述
## 背景
- Unsurprised- Learning
- Predict image-rotations：按照这种方法学习，得到的feature在许多问题上具有不错的表现
- Positive Unlabeled learning（PU）：通常将unlabeled data视为negative[[https://zhuanlan.zhihu.com/p/42435966]]
## 问题
- 现有很多方法不关注提取的特征的性质，不关注是否提取出的特征有利于生成现实生活中的图片
- 目前基于Predict image-rotations方法学习的feature对于具有旋转不变性的任务不适用（旋转不变性也是一种位移不变性，和我们目前研究的方向吻合
- 有些图像经过旋转后的结果不会影响我们的观测，这类图像的旋转难以观测（不给原图的情况下看不出来是否经过了旋转）![[Pasted image 20230208173437.png]]
## 贡献
- 提出了一种方法，将predict rotation任务看作一个PU问题可以解耦rotation prediction task和instance discrimination task，这种方法学习到的feature具有两种梯度，与旋转相关和与旋转无关的梯度
	- Rotation discriminative features：通过predict image-rotation学习，原图被看作positive instance。旋转后的图像如果不能和原图明显分辨，则会被当作unlabeled instance中的positive instance
	- Rotation unrelated features：通过对经过不同程度旋转的统一图片之间的distance进行惩罚
# 具体工作
- Image rotation prediction（RotNet）
	- S是一个具有N张图片的训练集，对每张图片定义K种旋转（旋转K种不同的角度），$X_{i,y}$表示对第i张图片采用第y种旋转![[Pasted image 20230208180040.png]]
	- 训练一个网络，用于分辨每张图和它旋转后的结果，这个网络输出的是它认为图片旋转的角度，目标是使预测结果和实际上的旋转方式y之间的差距最小。l采用交叉熵loss![[Pasted image 20230208180251.png]]
	- 为了识别旋转，网络必须识别和定位图片中显著的组成部分（这部分的旋转往往意味着整张图片的旋转）
- Reduce influence of noisy rotation labels：
	- 问题出现的原因：有些图片的旋转并不影响人的观察，具有rotate unrelate的性质
	- 首先，对于原图中所有的图片label为positive，经过一定角度旋转后的图片（注意，旋转后可能和原图没有差别，比如旋转了一圈）全部作为unlabeled，应该被当作negative。
	- pretrain一个二分类网络，预测一张图片是否旋转过。在每个交叉熵loss前添加一个权重，如以下公式表示，其中gamma是一个可调整的参数，![[Pasted image 20230208205851.png]]表示图片被预训练网络网络认为是positive（没有旋转过）的概率，当y=1时，图片没有经过旋转 ![[Pasted image 20230208205838.png]]这样修改之后，网络的训练目标如下，可以限制noise labeled的影响（旋转不敏感的图片，权重很小，造成的影响减小）：![[Pasted image 20230208210043.png]]
- Feature Decoupling
	- Framework![[Pasted image 20230208211244.png]]
	- 网络输出的向量被分割为两部分，分别作为对旋转敏感和不敏感的feature![[Pasted image 20230208212305.png]]
	- Rotate discriminative feature，被用作一个rotate classification任务，送入一个分类器预测旋转的类型，形成Lc![[Pasted image 20230208212331.png]]
	- Rotate unrelated feature，将原图进行K种旋转后分别送入网络，得到的所有unrelated feature取平均，要求各个旋转后的feature和平均feature之间差距最小                      ![[Pasted image 20230208212526.png]] ![[Pasted image 20230208212533.png]]
	- Image instance classification：以上Lr计算方法可能导致网络始终输出一个常向量以满足以上Lr最小（找到了trivial solution）解决方法为Non-parametric classification（无参数学习）
		- 第二部分特征应该和所有旋转情况的平均结果接近，而和其他图片产生的第二部分特征有差异。
		- 以下公式表示一张图片被认为是dataset中第i个instance 的概率，ˆ f is the L2-normalized version of ¯f![[Pasted image 20230208213656.png]]
		- 优化的目标如下：![[Pasted image 20230208213803.png]]
		- 为了简化上述的计算，使用以下loss近似Ln![[Pasted image 20230208213907.png]]
	- 最终的模型由3部分组成，目标如下，需同时优化特征采集网络和分类器网络                                             ![[Pasted image 20230208214201.png]]
- 最终分解得到与分别与旋转相关和无关的特征$f^{(1)}$和$f^{(2)}$
# 实验
## 定性实验
- nearest neighbor问题，红框出的方法为明显不合理的，本文在具有旋转不变性图片上的表现明显好于现有的RotNet![[Pasted image 20230208215011.png]]
- 卷积核可视化，本文提出的方法可以捕捉到深度特征![[Pasted image 20230208215233.png]]
## 定量实验
- 分别使用不同校验集进行Top-1 linear classfication，使用不同layer的feature进行计算，本文效果最好![[Pasted image 20230208215545.png]]
## 消融实验
- loss的组成部分和进行分解的feature来源哪一层进行了研究![[Pasted image 20230208220419.png]]
# 启发
- 可以将这种解耦方法用在目前的工作上
- 这种思路是否可以用在其他几何改变上，分离出对位移敏感和不敏感的特征？