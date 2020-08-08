Graphics Process Unit，图形处理器，是一种专门在个人电脑、工作站、游戏机和移动设备上做制图像和图形运算的微处理器。GPU减轻了对CPU的依赖，将部分属于CPU的工作转移到GPU上。GPU可以高速完成以下功能：

- 图形绘制
- 物理模拟
- 海量计算
- AI运算
- 其他复杂计算如音视频解码、加密解密、科学计算、离线渲染等

GPU能为我们提供更加真实的渲染画面。
![621d39ef-7288-4dc8-af56-9d878f85ced2](https://raw.githubusercontent.com/silence394/PicBed/PicGO/621d39ef-7288-4dc8-af56-9d878f85ced2.jpg)
PC上的GPU是长这个样子的。
![0a53ecbb-c184-4d96-b612-2a6dfa83052a](https://raw.githubusercontent.com/silence394/PicBed/PicGO/0a53ecbb-c184-4d96-b612-2a6dfa83052a.jpg)

而我们经常是显卡。显卡，则是由GPU、显存、电路板，BIOS固件、散热器等组成的。GPU是显卡的核心部件。

![30bcf7fd-2e40-47a0-aab1-993717021e8a](https://raw.githubusercontent.com/silence394/PicBed/PicGO/30bcf7fd-2e40-47a0-aab1-993717021e8a.png)

上面展示的是独立显卡，它以独立板卡的形式存在，插在主板的AGP和PCI-E插槽上。独立显卡配备独立的内存，不占用系统内容，能够提供良好的显示效果和运行性能。

还有一种显卡是集成显卡，它集成在了主板上，与CPU公用一块系统内存，一般都是紧挨着CPU芯片，以使CPU与集成GPU能够更快地进行数据交换。手机上CPU和GPU集成在SOC（System on Chip）上。一般来说，集成显卡要占用内存作为显存，占用系统的带宽，影响系统的整体性能，所以在手机上常常会碰到带宽的性能瓶颈。

### GPU的发展历程

file:///H:/FuckDownLoad/%E5%9B%BE%E5%BD%A2%E5%A4%84%E7%90%86%E5%99%A8%E7%9A%84%E5%8E%86%E5%8F%B2%E7%8E%B0%E7%8A%B6%E5%92%8C%E5%8F%91%E5%B1%95%E8%B6%8B%E5%8A%BF.pdf

https://en.wikipedia.org/wiki/Graphics_processing_unit#1970s

https://wenku.baidu.com/view/af266f85cc22bcd126ff0cd3.html?re=view

目前的GPU生产商，在PC端有Nvidia和AMD，在移动端有Imagination、ARM、高通等公司。


### PC端GPU的几大厂商

### 移动端GPU的几大厂商


- GPU是什么？用来做什么的？
- GPU的发展史
- GPU有哪些种？
- PC和移动端的GPU架构有什么区别？
- 如果有共有的概念，那就讲一下。比如可编程渲染管线。

Graphics Process Unit，图形处理器，是专门用于绘制图形和处理图元数据的微处理器。
它使Nvida公司在1998年发表GeForce 256时提出，在此之前电脑中处理影像输出的单元，很少被视为独立的运算单元。

减少了对CPU的依赖，并承担原来有CPU处理的工作，尤其是三维绘制，比CPU的处理速度更快。

GPU能做什么？
- 图形绘制
- 物理模拟
- 海量计算
- AI运算
- 其他复杂计算如音视频解码、加密解密、科学计算、离线渲染等

最早的时候是什么样的芯片，
然后Nvida在1998年发布GeForce256时提出了GPU的概念。而还有一个公司ATI也制作图形芯片，之前他们把图形芯片成为VPU，后来被AMD公司收购正式采用了GPU的名字。

由于中央处理器辅助的即时三维图形越来越常见于电脑和电视游戏中，从而使大众对3D图形的要求增加。个人电脑、PlayStation、任天堂等主机对图形处理器的变得越来越高，推动了GPU的加速发展。

随着硬件种类的增加，在各种各样的图形硬件上开发软件逐渐变得有大量工作的重复。为了解决这个问题，把这些工作封起来，方便客户端程序更方便地操作GPU，出现了跨平台的OpenGL、WIndows的DirectX，随着GPU的发展，OpenGL和DirectX也在更新迭代，也出现了能更好利用GPU的能力的苹果公司的metal和OpenGL的维护组织（Khronos Group）发表的Vulkan。

现在GPU已经遍布在我们生活中了，个人电脑、笔记本、平板电脑、手机，游戏主机如PlayStation、
Nintendo Switch。现在PC端的GPU制造商有Intel、Nvida和AMD，手机上的制造厂商有Imagination、Mali、高通。

当下PC主流的GPU，有Nvida的10系列和20系列，AMD的570、580、590系列，而手机端的CPU和GPU一般是放在一起说的，现在高端GPU的有华为的麒麟990，高通的adreno 865、855、845等，三星的Exynos 9820、9810，苹果的A13、A12、A11。

由于GPU的发展太多，无法一一介绍，所以在PC上介绍下具有代表性的Nvida的tuling架构，PC上介绍PowerVR的xxx架构。

tuling物理架构图

一些指标
什么意思。

而移动端GPU没能找到物理架构的东西，从文档中找到的更多是渲染架构的介绍。由于PC和移动平台明显的特征区别，导致二者要解决的瓶颈问题是不一样的。
比如手机上的TBR和TBDR。

二者都有相应的渲染管线，在后面会介绍到。
