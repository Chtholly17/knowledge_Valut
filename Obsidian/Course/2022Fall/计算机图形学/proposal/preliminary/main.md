## 阴影生成算法

通用方法：

首先产生一个shadow mapping来表征光源能照到的着色点->光源视角下的depth buffer

接着在眼睛视角下确认某个点是否在阴影中

问题：锯齿，走样，尖锐->只有纯黑和不黑两种

产生软阴影(柔和阴影)方法：

PCF与PCSS算法

PCF (percentage closer filtering)：

对阴影边缘进行加权平均，得到柔和阴影

PCSS(percentage closer soft shadows):

根据障碍物到阴影之间的距离计算阴影

距离越近越硬，越远越软