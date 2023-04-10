- Pytorch 实现fft，新旧版本对应问题
```python
import torch
input = torch.randn(1, 3, 64, 64)
### 旧版 ###
# 参数normalized对这篇文章的结论没有影响，加上只是为了跟文章开头同步
# 时域=>频域
output_fft_old = torch.rfft(input, signal_ndim=2, normalized=False, onesided=False)
# 频域=>时域
output_ifft_old = torch.irfft(output_fft_old , signal_ndim=2, normalized=False, onesided=False)

### 新版 ###
# 时域=>频域
output_fft_new = torch.fft.fft2(input, dim=(-2, -1))
output_fft_new_2dim = torch.stack((output_fft_new.real, output_fft_new.imag), -1)  # 根据需求将复数形式转成数组形式
# 频域=>时域
output_ifft_new = torch.fft.ifft2(output_fft_new, dim=(-2, -1))                   # 输入为复数形式
output_ifft_new = torch.fft.ifft2(torch.complex(output_fft_new_2dim[..., 0],      # 输入为数组形式
                                                output_fft_new_2dim[..., 1]), dim=(-2, -1))    
# 注意最后输出的结果还是为复数，需要将虚部丢弃
output_ifft_new = output_ifft_new.real
print((output_ifft_new - input).sum())   # 输出应该趋近于0（因为存在数值误差）
```
- 本次实验：首先应用新版本的torch实现fft，由于结果为虚数，因此需要计算幅值
```python
def calc_fft(image):
	'''image is tensor, N*C*H*W'''
	fft_new = torch.fft.fft2(image, dim=(-2, -1))
	fft = torch.stack((fft_new.real, fft_new.imag), -1)
	fft_mag = torch.log(1 + torch.sqrt(fft[..., 0] ** 2 + fft[..., 1] ** 2 + 1e-8))
	return fft_mag
```
- 接下来，使用Gaussian Kernel进行滤波，分离得到低频成分
```python
def get_gaussian_blur(x, k, stride=1, padding=0):
	res = []
	x = F.pad(x, (padding, padding, padding, padding), mode='constant', value=0)
	for xx in x.split(1, 1):
		res.append(F.conv2d(xx, k, stride=stride, padding=0))
	return torch.cat(res, 1)

def get_low_freq(im, gauss_kernel):
	padding = (gauss_kernel.shape[-1] - 1) // 2
	low_freq = get_gaussian_blur(im, gauss_kernel, padding=padding)
	return low_freq
```
- 最后，使用低频成分和原本图片分别计算得到灰度，然后相减得到高频成分![[Pasted image 20230310103213.png]]
- 本次实验操作存在一定问题，实际上是对fft的结果进行了高频低频成分分解，最后的结果，**lpips效果较于其他分解方法有一定改进，但dists变差了![[SWD_freq.png]]
- 同时，**减少了边缘上的绿色问题

