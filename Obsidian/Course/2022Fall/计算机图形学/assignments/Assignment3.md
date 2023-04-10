## 1、What are the difference between Local illumination model and Global illumination model?
* Local illumination model：仅模拟直接的光照，仅仅关注被光源直接照射到的表面的光照情况，不考虑由一个平面反射或折射的光对其他平面的影响，也不考虑反射光与光源发出的光的混合，当使用局部光照模型时，没有被光直接照到的平面不存在漫反射光和镜面反射光，只展现环境光的效果。此算法比较简单，计算速度较快。
* Global illumination model：考虑直接的光源和间接的光源（反射光与折射光），描述了光在多个平面上的反射效果，可以产生更加真实的光照效果，但算法比较复杂，时间消耗较大。
## 2、What is the purpose of material attribute?
材质的属性定义了物体表面对同一种光源的不同反射情况，通过参数定义对漫反射光、环境光、镜面反射光的反射强度，可以模拟出显示生活中不同材质对光照的反射效果，同时表面的材质还包含了基础的颜色信息。通过结合颜色信息和光线的反射效果，可以展现出比较真实的材质特性。
## 3、Can you approve Blinn-Phong is an approximation of Phong reflection model?
Blinn-Phong在计算镜面反射光的过程中，使用了近似$L_sK_s(R\cdot V)^{\alpha}\to L_sK_s(N\cdot H)^{\alpha}$，如下图所示，将各个向量在运算之前单位化，相当于使用近似$cos\theta _ 1=cos\theta _2$，Phong反射模型需要先使用向量N与L计算出向量R，再与向量V进行点乘，并且R的计算有比较大的运算量，在计算每个像素时重复需要消耗较大的时间。而Blinn Phong的近似方法只需要额外计算H向量，H向量的计算相对于R而言比较简单，可以节约大量的时间，并且此近似在数学结果上非常接近，效果相差不大。![[fa267f53d390f9c09ad5ee65ad366f1.jpg]]

## 4、Why Phong shading produce better result than Guround shading?
Guround Shading使用在标准图像流水线中，用每一个顶点周围与其相交的所有多边形的法向量求平均计算这个顶点的法向量。之后先计算顶点的光照情况，计算顶点的颜色，再分别将顶点的颜色属性使用二维线性内插，内插到每个像素上。而Phong shading直接将顶点的法向量内插到每个像素上，对于每个像素，分别计算这个像素的光照、颜色情况。
这导致Phong shading的计算量远远大于Guround shading，但其光照表现更加细腻，每个像素都具有单独计算的颜色信息，因此表现更好，但由于需要内插法向量，无法直接用在现有的渲染流水线中。
## 5、What information can be stored in a Texture?
纹理可以存储任何几何体的表面信息，包括几何体表面的形状、颜色，这些信息可以帮助显卡在渲染过程中更加真实描述出物体的外观。同时，纹理中也存储了几何体对光线的反射率、法向量偏移量等信息，可以帮助显卡显示出真实的物体材质、光照表现。
总而言之，建模完成后，几何体表面显示所需要的所有信息，除了来自于顶点的属性内插之外，都存储在纹理之中。
## 6、Why the Texture coordinates require perspective correction?
在texture mapping过程中，由于需要将以图象形式存储的2D纹理投射到3D的空间平面上，而3D空间平面存在深度信息，因此，在纹理图像上的坐标在空间平面上由于深度不同可能会发生变化，这个问题是由于3D空间中的透视导致的，因此需要在texture mapping的过程中消除透视产生的影响，也就是perspective correction。
通过数学计算，消除深度信息对texture坐标产生的影响，才可以在空间平面上显示出我们希望的纹理。
## 7、What is the cause of aliasing in Textures?
由于3D空间中存在深度信息与透视，相同大小的屏幕区域，在近处可能包含的texel较少，可能需要多个pixel来表示一个texel，而远处可能包含大量的texel，一个像素上可能包含多个texel，这会导致显卡不知道应该在这个pixel上显示哪一个texel的信息。也可以解释为，当纹理上的信号频率过高，而pixel数量较少，相当于采样频率小于目标信号的一半，不满足采样定律，最后导致混叠现象的出现。
## 8、What are the rendering equation and reflection equation?
- Reflection equation：定义了入射光线在某个点上某一方向上所有反射能量的总和，可以用以下公式表示
$$
L_0(\vec p,w_0)=\int_{H^2}f_r(\vec p,\omega_i\to \omega_0)L_i(\vec p,w_0)cos(\theta )d\omega_i
$$
$L_r(p,\omega _r)$表示反射或出射的光线（或辐射度），$\vec p,\omega _i,\omega _0$分别表示了着色点空间位置，入射方向和出射方向。$f_r$表示BRDF，即双向反射分布函数，能表示着色点 $\vec p$ 在各个方向上的反射方式，$L_i$表示入射光线，最后$cos(\theta )$表示入射光线在法线方向上的投影。

- Rendering equation：在反射方程的基础上加入自发光项$L_e$，在上半球进行积分就得到渲染方程。渲染方程中，入射光理解可以理解为从其他方向来的反射光的递归表达式，递归渲染方程使用算子简写后，可以表示为形成多次光线弹射的叠加光线，即全局光照。$$
L_0(\vec p,w_0)=L_e(p,\omega _r)+\int_{\Omega ^+}f_r(\vec p,\omega_i\to \omega_0)L_i(\vec p,w_0)cos(\theta )d\omega_i
$$
$$
L=E+KE+K^2E+K^3E+\cdots 
$$
# programing
- sky-box和星球纹理贴图上次已经实现
- 本次添加光照，包括一个位于太阳中心的电光源和位于屏幕中心的spotlight，类似于一个手电筒，方便大家进行太阳系的探索
- 为不同星球设置了不同的材质