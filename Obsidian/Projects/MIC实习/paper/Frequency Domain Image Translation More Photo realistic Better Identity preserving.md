# 概述
## 背景
- Image-to-Image Translation：synthesize new images based on the source and reference images
- GAN-based method
## 问题
- identity preserving：synthesized image can over-adapt to the reference domain and lose the original identity characteristics（失去identity）![[Pasted image 20230310152334.png]]
- generation process may lose important fine-grained details, leading to suboptimal visual quality（生成的图片质量下降）
## 贡献
- FDIT
	- 将图片分为高频和低频成分![[Pasted image 20230310152223.png]]
	- 在训练过程中保持频域的一致性
# 具体工作
- Framework![[Pasted image 20230310155244.png]]
	- 使用高斯滤波分解出图像的低频成分，原图像减去低频成分得到高频成分![[Pasted image 20230310155629.png]] ![[Pasted image 20230310155641.png]]
- pixel space loss
	- Reconstruction Loss in pixel space：minimize distance between x and G(E(x))![[Pasted image 20230310155745.png]]
	- Translation matching loss in pixel space![[Pasted image 20230310155912.png]]
- Fourier Frequency Space loss
	- Reconstruction loss in the Fourier space![[Pasted image 20230310160023.png]]
	- Translation matching loss in the Fourier space![[Pasted image 20230310160038.png]]
	- Frequency mask: 将频域图移动，使低频成分位于频域图的中间区域，以下公式中的MH就是high-frequency mask![[Pasted image 20230310160248.png]]
- Overroll loss![[Pasted image 20230310160334.png]]
# 实验
- 定性实验![[Pasted image 20230310161117.png]]
- 定量实验：和GAN based method 对比![[Pasted image 20230310163313.png]]
# 启发
- **之前的实验只考虑了时域以及对时域的分解，接下来可以考虑添加频域的约束
- 频域与时域是否需要区别对待？
- 高频与低频成分是否需要区别对待？
