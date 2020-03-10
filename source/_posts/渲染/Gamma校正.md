---
title: Gamma校正
categories: 渲染
date: 2018-07-19
---

### 一、Gamma的由来
早期的CRT（阴极射线管）显示器显示出来的亮度和电压的不是线性关系，而是幂次的关系：$ V_{out} = V_{in}^\gamma$输入电压得到一个亮度，但是想要将这个亮度提高一倍，却无法通过将电压提高一倍来实现。输入的电压和输出的亮度呈下图的关系。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fw7t9fviekj208w05gdfv.jpg)
由于CRT的这个物理特性，显示图像的亮度要比图像中存储的数值的亮度低。为了弥补这个问题，需要对图像采集设备做补偿，将图像中存储的亮度做一个逆幂次变换。经过这两次幂次变换，图像采集的图像与CRT显示的图像能够保持一致。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fw7t9fvf1ej208c06qgm3.jpg)
对颜色施加幂次变换的过程称为Gamma校正。
- 一般CRT显示器的γ值为2.5，这个值叫**display gamma**，这个操作称为**gamma compression**。
- 对图像捕捉设备做的补偿γ为display gamma的倒数，叫做**encoding gamma**，这个操作称为**gamma expansion**。

接下来从人感受的角度来分析，这颜色：黑色、中灰、白色。在下面的图中，最左边的为黑色，记为0；最右边的颜色为白色，记为1；正中间的颜色为人眼感受到的中灰（感觉上的黑白的中间点），记为0.5。
<img src="http://ww1.sinaimg.cn/large/c5c3a364ly1fw6eowio2yj20dw046dgr.jpg" width=512 height=128 />
定义黑色的物理反射率为0%，白色的物理反射率为100%，然而 经过物理测量之后，中灰的物理反射率并不是直觉上的50%，而是20%左右。
以物理反射率从0%，100%线性变化，做出亮度变化的图：
<img src="http://ww1.sinaimg.cn/large/c5c3a364ly1fw6fgdia0gj20k00463ye.jpg" width=512 height=128 />
中灰的地方在20%的位置，而图片的大部分都被亮色覆盖了。在这张图中很容易能得出结论：人眼对不同亮度的敏感程度也是不同的，对暗部变化的敏感要高于亮部。

而早期图像的存储空间有限，常用的RGBA32格式，其每个通道只有8位，也就是说只能存储256种亮度，所以基于人眼的特性，用更多的空间存储亮部，能够更好的利用图像的存储空间。经过科学家基于人眼特性的测量，用0.45的encoding gamma对输入的亮度进行编码，得到一张图像，显示的时候再经过$\frac{1}{0.45}$的display gamma进行解码，就可以显示原本的亮度值。
多么棒的巧合！CRT的物理特性和人眼对亮度的敏感曲线都是幂次。后来微软、爱普生、惠普共同提供了一套标准方法来定义色彩，也就是现在设计师常用的sRGB（standard Red Green Blue）。在这个标准下，display gamma值为2.2。

### 二、为什么需要Gamma校正
游戏的真实渲染是为了尽可能的模拟显示的自然环境的光、雾、阴影、折射、反射等效果，如果我们忽视了Gamma的效应，会导致渲染并不具备物理上的正确性。

设计师做的一张亮度存储为0.5的基于sRGB标准的图像，在经过游戏引擎的光照后，将亮度提升2倍，输出到显示器。如果不考虑Gamma，游戏引擎读取图像后亮度为0.5，经过光照处理后亮度为1.0，再经过显示器的display gamma（2.2）处理，其输出的亮度是1.0。但是这并不符合物理上的规律。由于sRGB标准的图像需要对输入的亮度做encoding gamma($\frac{1}{2.2}$)的处理，所以存储数据为0.5对应的物理亮度应当是0.218（$0.5^{2.2}$）。经过光照处理后为0.436。这才是正确的亮度。

sRGB空间是一个非线性空间，我们在这里称之为**Gamma空间**，而自然界的光照是**线性空间**的运算。CRT显示器的输出是非线性的（**gamma expansion**），要与**encoding gamma**配合使用。所以sRGB空间下图像可以直接显示出原本的样子。但是如果对贴图进行光照运算，必须要把贴图中的值转换到线性空间，线性空间下计算出来的光照必须要经过**encoding gamma**，再由CRT显示器显示才能保证结果的正确性。下面来看一下Gamma空间和线性空间下渲染的区别。

#### 光照

设想游戏渲染一个这样的场景，平行光照向一个球体，我们从垂直于光照方向的视线观察。在观察空间中球的最右侧的点**A**，经过光照后其亮度为1.0，输出到显示器上的亮度也是1.0。球体上与光线成60°夹角的点**B**，经过光照后的亮度为0.5。经过CRT显示器的display gamma处理，得到最终显示出来的亮度为0.218。这样的话，**A**的亮度是**B**的亮度的$\frac{1}{0.218}$倍，并不是自然界中的二倍，渲染出来的效果是图**b**中的效果。
图a中的效果是考虑了显示器的**display gamma**，在显示器显示之前，对计算出来的亮度做了**encoding gamma**，得到亮度值为0.73，再由CRT显示器显示为0.5的亮度。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fw7u8hjtd6j20dw05lt8x.jpg)
对比现实世界中的足球，显然图**a**更真实一些。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fw7x5otbscj207q05pac8.jpg)

#### 滤波
一个sRGB下两个相邻的像素红和绿，对这两个像素进行滤波，.
如果在Gamma空间下滤波，$ (1 + 0) * 0.5 = 0.5$，$0.5 * 255 = 128$，得到的结果是(128, 128, 0)。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fw7yesngv6j208f03l741.jpg)
但是符合物理规律下的滤波应该在线性空间下运算，
1. $(1, 0)$由$pow(x, 2.2)$ 转换到线性空间，得到(0, 1)
2. 线性空间中进行滤波，$(1 + 0) * 0.5 = 0.5$
3. 线性空间中运算的结果转换到sRGB空间下，$pow(0.5, \frac{1}{2.2}) = 0.73$
所以正确的结果是(187, 187, 0)。

![](http://ww1.sinaimg.cn/large/c5c3a364ly1fw7yefmyahj208f03l741.jpg)
在MipMap中会进行到线性插值，所以带mipmap的图片在Gamma空间滤波后，显示的结果要比正确的结果暗。

对一个蓝绿色底色和各个颜色的圆盘的图高斯模糊，对比Gamma空间和线性空间下的模糊的结果。在Gamma空间下红色和黄色的圆盘边界处出现了不正常的黑色、绿色。在线性空间下的结果是符合物理规律的。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fw9u69x9lgj20lc0e8ag0.jpg)
在gamma空间Blend出现的异常现象跟上面讲述的是一个原因，不再赘述。

### 三、怎么使用Gamma校正
使用Gamma校正有两个关键点：
- 将读取图像后的空间转换到线性空间
- 输出到CRT显示器之前，对线性空间运算的颜色encoding gamma

当然，Gamma校正是对非线性空间的图像和CRT显示器进行的。而直接存储物理亮度的浮点纹理（HDR图像）和HDR显示器，是不需要做以上操作的。

#### 空间转换
sRGB空间是非线性空间，γ为2.2。在shader进行幂次运算是将其转换到线性空间最直接的做法。
``` hlsl
float4 color = pow(layer0.Sample(sampler0, uv), 2.2);
```
硬API件一般都支持sRGB格式的图像输入，如D3D11的DXGI_FORMAT_R8G8B8A8_UNORM_SRGB，将图像指定为sRGB格式，就不需要在shader做这个转换了。

#### 输出到CRT显示器
CRT显示器的输入如果是线性空间下的，其本身的display gamma会导致显示器显示的颜色是非线性的。所以需要对线性空间的输入做encoding gamma，将其转换到非线性空间，再经过display gamma可得到线性的显示输出。
``` hlsl
Out.color = pow(color, 1 / 2.2);
return Out;
```
同样的，设置RenderTarget或者BackBuffer的格式是sRGB的，就可以省去shader里的运算，由硬件完成。

### 四、总结
当下的游戏渲染中HDR和PBR是必备的功能，保证光照在线性空间运算，进行gamma校正，是正确渲染的前提。如果没有gamma校正，HDR中的bloom光晕很小，PBR材质的颜色偏暗，细节不明显，美术需要大费周章的去调节其他参数来弥补这个错误。

### 参考
1. https://zhuanlan.zhihu.com/p/36581276
2. https://en.wikipedia.org/wiki/Gamma_correction
3. https://www.zhihu.com/question/27467127
4. https://blog.csdn.net/candycat1992/article/details/46228771
5. https://developer.nvidia.com/gpugems/GPUGems3/gpugems3_ch24.html