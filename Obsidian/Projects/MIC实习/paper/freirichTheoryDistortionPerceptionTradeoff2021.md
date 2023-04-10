# A Theory of the Distortion-Perception Tradeoff in Wasserstein Space
## Metadata
* Item Type: [[Conference paper]] 
* Authors: [[Dror Freirich]], [[Tomer Michaeli]], [[Ron Meir]] 
* Date: [[2021]] 
* Date Added: [[2023-03-29]] 
# 
## Zotero Links
* PDF Attachments
	- [Freirich et al. - 2021 - A Theory of the Distortion-Perception Tradeoff in .pdf](zotero://open-pdf/library/items/GBFM4ES2)
* [Local library](zotero://select/items/1_D7YCZIQE) 
## Inroduction
### Background 
- perceptual-distortion tradeoff****
### Drawbacks
- a open question: **what is the minimal distortion that can be achieved under a given perception constraint?**![[Pasted image 20230405222655.png]]
- the tradeoff in the paper [[blauPerceptionDistortionTradeoff2018]] is obtained by GAN
### Contribution
- this paper discovered the following properties of DP tradeoff:
	- Proved the **DP(Distortion-Perceptual) function is always quadratic, regardless of the underlying distribution**
		- estimators on the DP curve **form a geodesic in Wasserstein space(i.e. a curve that minimizes the Wasserstein distance between any two points in Wassertein space) **
		- For general distributions, these estimators can be constructed from the estimators at the two extremes of the tradeoff
			- global MSE minimizer
			- a minimizer of the MSE **under a perfect perceptual quality constraint**. it can be obtained as a **stochastic transformation of the global MSE minimizer**
	- In the Gaussian setting, provide a closed form expression for optimal estimators and for the corresponding DP curve
## Porposed Method
- problem setting
	- DP function(using L2 distance and Wasserstein-2 distance as representation of Distortion and perceptual loss respectivily):![[Pasted image 20230406215040.png]]
		- some general properties of DP function:![[Pasted image 20230406215437.png]]
			- X* is the output of the restoration network **aims at minimize MSE only**
				- D* and P* is the correspond distortion and perceptual quality(measured by W2 distance)
	- Wassertein Distance anein Distance anein Distance anein Distance and Gelbrich Distance
		- Wassertein Distance: ![[Pasted image 20230406220650.png]]
			- if U and V are Gaussian Distributions: ![[Pasted image 20230406223219.png]]
				- $T_{1\to 2}$ is the optimal transformation between $N(0,\Sigma _1)$ and $N(0,\Sigma _2)$
		- Gelbrich Distance: measure the **distance between two Gaussian distributions with different means and covariances**![[Pasted image 20230406222349.png]]
			- m is a d-dimensional vector
			- $\Sigma$ is a d-dimensional matrix
		- property between Wasserstein Distance and Gelbrich Distance:![[Pasted image 20230406223620.png]]
- Theorem 1: DP function is given by:($(x)_+=min(0,x)$) $$D(P)=D^*+[(P^*-P)_+]^2$$
	- where D* and P* is the distortion and perceptual quality **attainded by the Minimum MSE estimator X*(X* is the output image)
	- there is a lower bound on the DP function for any P: where G* refer to the corresponding Gelbrich Distance between X and X*![[Pasted image 20230407120941.png]]
- Theorem 2:![[Pasted image 20230407122553.png]]
- **Theorem 3**: $\hat{X}_0$ is the estimator achieving perceptual quality 0 and distortion D(0)![[Pasted image 20230407123152.png]]
	- Having a model that leads to low MSE and one that leads to good perceptual quality, **it is possible to construct any other estimator on the DP tradeoff**
- Theorem 4: 
	- In Gaussian Setting: from theorem2 and theorem3 we have![[Pasted image 20230407133440.png]] and![[Pasted image 20230407133501.png]]
	- Optimal estimators in the Gaussian case: X and Y are zero-mean Gaussion random vectors(I denotes the Identity Matrix)![[Pasted image 20230407133622.png]]
- Theorem 5:![[Pasted image 20230407133735.png]]
## Experimental
- Numerical illustration: ![[Pasted image 20230407134316.png]]
## Conclusions
- the paper revealed some closed-form properties of Perceptual-Distrotion tradeoff in Wasserstein Space