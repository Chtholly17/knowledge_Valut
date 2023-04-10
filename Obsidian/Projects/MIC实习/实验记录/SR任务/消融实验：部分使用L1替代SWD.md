在之前使用FFT替换了高斯滤波的基础上，进行了一系列消融实验，使用L1替换SWD，总体而言相较于之前有一定性能下降，但仍然高于baseline，全部使用l1的方法也好于不分解使用SWD的方法
**值得注意的是：在urban100数据集上，lpips性能有明显的提升
- 全部换成L1![[FDIT_l1.jpg]]
- 频域l1，spatial swd![[freq_l1_spatial_swd.png]]
- 频域swd，spatial l1![[freq_swd_spatial_l1.png]]
- lpips![[lpips.png]]
- swd（baseline）![[SWD.png]]