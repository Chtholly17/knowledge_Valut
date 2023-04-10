好的，我可以用latex给出数学公式的表达。以下是一些WD数学性质的数学表达：

-   WD是一种度量空间，它满足
    -   非负性：$W(\mu,\nu)\geq 0$，当且仅当$\mu=\nu$时，$W(\mu,\nu)=0$
    -   对称性：$W(\mu,\nu)=W(\nu,\mu)$
    -   恒等性：$W(\mu,\mu)=0$
    -   三角不等式：$W(\mu,\nu)\leq W(\mu,\lambda)+W(\lambda,\nu)$
-   WD是一种拓扑空间，它具有
    -   完备性：如果${\mu_n}$是一个Cauchy序列，那么存在一个极限分布$\mu$，使得$W(\mu_n,\mu)\to 0$
    -   紧致性：如果${\mu_n}$是一个有界序列，那么存在一个子序列${\mu_{n_k}}$和一个极限分布$\mu$，使得$W(\mu_{n_k},\mu)\to 0$
    -   连续性：如果$\mu_n\to\mu$和$\nu_n\to\nu$，那么$W(\mu_n,\nu_n)\to W(\mu,\nu)$
-   WD是一种几何空间，它具有
    -   曲率：如果$X$是一个Riemann流形，那么$W_p(\mu,\nu)\leq d^p(x,y)$，其中$x$和$y$是$\mu$和$\nu$的重心，$d$是$X$上的测地距离
    -   梯度：如果$f$是一个Lipschitz连续函数，那么$W_p(f_\#\mu,f_\#\nu)\leq L^p W_p(\mu,\nu)$，其中$L$是$f$的Lipschitz常数，$f_\# \mu$表示$f$将$\mu$映射到的分布
    -   散度：如果$X$是一个Euclidean空间，那么$$W_p(\mu,\nu)=\left(\int_{X\times X}|x-y|p d\pi(x,y)\right){1/p}$$其中$\pi$是在$X\times X$上的联合分布，满足$\pi$的边缘分布是$\mu$和$\nu$