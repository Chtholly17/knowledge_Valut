* [[Image Quality Assessment：From Error Visibility to Structual Similarity]]
	SSIM，具有标准参考图片的IQA问题，使用与原始图像结构上的相似性评价图像质量
* [[Perceptual Image Quality Assessment with Transformers]]
	IQT，将Transformer应用到IQA问题上，研究输入信息、采样方法与embedding
* [[Image Quality Assessment：Unifying Structure and Texture Similarity]]
	DISTS，使用SSIM的思想，使用CNN提取feature map并计算structure和texture特征，对细微的纹理改变不敏感
- [[The Unreasonable Effectiveness of Deep Feature as a Perceptual Metric]]
	LPIPS，进行了deep feature用于表达感知信息的多个实验，计算feature map之间的$l_2$ distance
- [[Generating Natural images with direct Patch Distributions Matching]]
	GPDM，使用Sliced Wasserstein Distance(SWD)计算图像patch distribution之间的distance，使用随机CNN将patch投影到高维空间的向量
- [[The Contextual Loss for Image Transformation with Non-Aligned Data]]
	CX，提出一种计算图像之间Contextual距离的loss function，可用于非对齐的数据
- [[Generalized Sliced Wasserstein Distances]]
	GSW，提出了一种Generalize的SW计算方法，使用不同的超平面进行投影
- [[Wasserstein Generative Adversarial Networks]]
	WGAN，使用Wasserstein-1 Distance替代传统GAN中的JR散度，探讨Wassertein Distance与其他距离的数学性质区别
- [[Projected Distribution Loss for Image Enhancement]]
	PDL，使用SWD计算边缘分布，提出另一种计算SWD Loss的思路
- [[Pooling by Sliced-Wasserstein Embedding]]
	PSWE，使用SWD进行pooling，投影的参数可训练
- [[SLOSH： Set LOcality Sensitive Hashing via Sliced Wasserstein Embeddings]]
	SLOSH：结合SWE（使用SWD Embedding）与LSH对structred-set（picture or point cloud）进行hash
- [[Distributional Sliced - Wasserstein And Applications To Generative Modeling]]
	DSW：对SWD的优化，使用网络在参数空间对采样范围进行约束，获取更有意义的投影方向
- [[Subspace Robust Wasserstein Distances]]
	PRW和SRW：相比SWD投射到更高维的空间上，使用计算前k大特征值的和取代计算trace
- [[Deep Self-Dissimilarities as Powerful Visual Fingerprints]]
	DSD：在scale层面上计算图片与自己的downscale结果gram之间的差。可以看作是对图像texture information的一种表示
- [[GANSpace Discovering Interpretable GAN Controls]]
	在GAN space上进行PCA（主成分分析），不同层级不同的的principal direction可能代表了图片不同的特征信息
- [[Closed-Form Factorization of Latent Semantics in GANs]]
	SeFa：提出一种在latent space上进行分析的方法，独立于模型与训练数据，可以提取出latent space中较为“重要”的方向。
-  [[Unsupervised Discovery of Interpretable Directions in the GAN Latent Space]]
	在latent space上通过同时训练K个方向组成的矩阵A和constructor，找到独立性更强的方向，这样的方向往往更具有语义上的可解释性
- [[Contrastive Learning for Unpaired Image-to-Image Translation]]
	CUT，采用cycleGAN的思路使用PatchNCE对图像的patch进行Contrastive learning，对空间变化有一定容忍度
- [[Locally Adaptive Structure and Texture Similarityfor Image Quality Assessment]]
	A-DISTS，在DISTS基础上增加了对texture或者structure可能性的计算作为权重，更好描述结构和纹理方面的差别
- [[Deterministic Feature Decoupling by Surfing Invariance Manifolds]]
	一种利用normalization在manifold上进行特征解耦的方法，偏向理论
- [[Self-Supervised Representation Learning by Rotation Feature Decoupling]]
	一种无监督学习方法，可以用于解耦图像中对旋转敏感的特征和对旋转不敏感的特征。
- 