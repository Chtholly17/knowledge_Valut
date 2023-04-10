## Bresenham
1、基本原理：
如下图，在 0≤x≤y 的 1/8 圆周上，像素坐标 x 值单调增加，y 值单调减少，设第 i 步已确定$(x_i,y_i)$是圆上的像素点，那么$(x_{i+1},y_{i+1})$只能是$H(x_i+1,y_i)$与$D(x_i+1,y_i-1)$中的一个。关键在于比较$d_h$与$d_D$的大小
![[Pasted image 20221026231708.png]]
2、近似
下图中D点应该是$y_i-1$
![[Pasted image 20221026231854.png]]![[Pasted image 20221026231911.png]]
这里只需要推出圆心为(0,0)的情况，当圆心不为0时，在绘制每个点的时候直接进行坐标变换即可。
判别式如下：
$p_i=d_H-d_D=2(x_i+1)^2+2y_i^2-2y_i-2R^2+1$
3、判别式的递推关系
判别式存在递推关系：
$p_{i+1}=2(x_{i+1}+1)^2+2y_{i+1}^2-2y_{i+1}-2R^2+1$
我们只研究第一象限$x=y$上方的八分之一圆，之后通过对称关系，就可以画出整个圆，在这里有$x_{i+1}=x_i+1$，因此有以下递推公式：
$p_{i+1}-p_i=4x_i+6+2(y_{i+1}^2-y_i^2+y_i-y_{i+1})$
	![[Pasted image 20221027014811.png]]
起始点为(x,y+R)，带入可得$p_0=3-2R$
需要注意的是，Bresenham算法中所有的运算都是整数，因此之前首先需要将半径r转为近似的整形后才送入，画出八分之一圆的代码如下：
```cpp
void drawHalfQuarterCircle(const Point& c, int r, int quadrant, Buffer& buff) {
        //圆心的x、y的坐标
        int x = c.x;
        int y = c.y;
        //初始的xi与yi
        int xi = 0;
        int yi = r;
        int pi = 3 - 2 * r;


        int x1 = 0, y1 = 0;
        for (; xi <= yi; xi++) {
            //为了实现对称性，需要依据编号选择实际的坐标
            switch (quadrant)
            {
                case 1:         //一象限上方部分
                    x1 = xi;
                    y1 = yi;
                    break;
                case 2:
                    x1 = yi;
                    y1 = xi;
                    break;
                case 3:
                    x1 = yi;
                    y1 = -xi;
                    break;
                case 4:
                    x1 = xi;
                    y1 = -yi;
                    break;
                case 5:
                    x1 = -xi;
                    y1 = -yi;
                    break;
                case 6:
                    x1 = -yi;
                    y1 = -xi;
                    break;
                case 7:
                    x1 = -yi;
                    y1 = xi;
                    break;
                case 8:
                    x1 = -xi;
                    y1 = yi;
                    break;
                default:
                    break;
            }
            //进行坐标变换
            buff.putPixel(x1 + x, y1 + y);
            //进行pi的递推
            if (pi >= 0) {
                pi += 4 * (xi - yi) + 10;
                yi--;
            }
            else {
                pi += 4 * xi + 6;
            }
        }
    }
```
改进方法，直接一次绘制出8个对称圆弧，不需要调用八次，节约时间
```cpp
void drawCircle(const Point& c, int r, Buffer& buff) {
        //圆心的x、y的坐标
        int x = c.x;
        int y = c.y;
        //初始的xi与yi
        int xi = 0;
        int yi = r;
        int pi = 3 - 2 * r;
        for (; xi <= yi; xi++) {
            //进行坐标变换，对称地绘制八个圆弧
            buff.putPixel(xi + x, yi + y);
            buff.putPixel(yi + x, xi + y);
            buff.putPixel(yi + x, -xi + y);
            buff.putPixel(xi + x, -yi + y);
            buff.putPixel(-xi + x, -yi + y);
            buff.putPixel(-yi + x, -xi + y);
            buff.putPixel(-yi + x, xi + y);
            buff.putPixel(-xi + x, yi + y);

            //进行pi的递推
            if (pi >= 0) {
                pi += 4 * (xi - yi) + 10;
                yi--;
            }
            else {
                pi += 4 * xi + 6;
            }
        }
    }
```
## DDA
假设第一个像素从(x,y+r)开始，关键在于确定某一个像素之后，如何推出下一个像素的坐标。可以近似认为，下一个像素应该也是这个像素所在切线上的下一个点。
假设当前像素为$(x_i,y_i)$，那么当前像素所在弦的斜率为：$k_1=\frac {y_i-y}{x_i-x}$因此切线的斜率为$k_2=-\frac{1}{k_1}=\frac{x-x_i}{y_i-y}$
我们采用与bresenham算法相同的思路，只绘制第一象限上半部分的八分之一圆周，在这里可确保k1不等于0，之后我们使用对称的思想绘制全部八个圆弧即可。在这八分之一圆弧上，对于下一个像素点$x_{i+1}$，一定有$x_{i+1}=x_i+1$、$y_{i+1}=round(y_i+k_2)$，这样就可以用递推的方式绘制出全部的像素，具体代码如下，DDA算法直接使用浮点运算，在绘制像素时取整即可。
```cpp
void drawCircle(const Point& c, double r, Buffer& buff) {
        int x = c.x;
        int y = c.y;
        double x1 = 0.0;
        double y1 = r;
        //初始时k1不存在，k2为0
        double k2 = 0, k1;
        while(true) {
            int xi = (int)round(x1), yi = (int)round(y1);
            if (xi > yi)
                break;
            // 进行坐标变换，对称地绘制八个圆弧
            buff.putPixel(xi + x, yi + y);
            buff.putPixel(yi + x, xi + y);
            buff.putPixel(yi + x, -xi + y);
            buff.putPixel(xi + x, -yi + y);
            buff.putPixel(-xi + x, -yi + y);
            buff.putPixel(-yi + x, -xi + y);
            buff.putPixel(-yi + x, xi + y);
            buff.putPixel(-xi + x, yi + y);
            x1 += 1;
            y1 += k2;
            k1 = y1 / x1;       //计算法线斜率
            k2 = -1 / k1;       //更新k2
        }
    }
```
## 性能比较
我使用测试程序分别对两种算法的性能进行了测试，测试的结果如下图所示：
![[Pasted image 20221027160451.png]]
可以发现，两种算法的性能差距并不大，绘制同样的圆时，消耗的时间基本相同，可能的原因在于，两种算法的时间复杂度基本相同，区别仅仅在于每次进行的是浮点运算还是整形运算，在当前的硬件环境下，GPU对于浮点运算可能存在硬件级别的加速，与整形运算的时间差异不大，因此这两种算法的时间基本相同。
同时，与其他同学比较，发现在逻辑基本相同的情况下，我的这两种算法运行时间明显长于其他同学，可能的原因在于我在windows环境下VS 2019集成环境中使用debug模式运行代码，而其他同学使用release模式或者在linux平台上运行，可能是运行模式和硬件性能的差异导致了运行时间的较大差异，与算法本身不存在太大的关系。
