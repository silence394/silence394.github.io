- GPU历史
  - GPU的名字
  - 发展史
  - 现在有多少厂商，都用来做什么样的的显卡。
- 现代GPU能用来做什么？
  - 对GPU的应用
- GPU的进化节点
  - GPU的软件架构，渲染管线进化之路
  - 统一渲染和分离式渲染
  - 固定功能渲染和可编程渲染
  - 立即渲染和基于tile的渲染
- 桌面GPU和移动GPU的分化
  - 二者所在平台的特点决定了对GPU不同的要求，从而决定了GPU的不同发展方向。
  - 续航、发热


### GPU概念
GPU，全程为Graphics Process Unit，即图形处理器，是一种专门在个人电脑、工作站、游戏机和移动设备上做制图像和图形运算的微处理器。GPU减轻了对CPU的依赖，将部分原本属于CPU的工作转移到GPU上，比如显示图像、文字等。

我们经常提到的N卡、A卡是PC上的独立显卡。它是由GPU、显存、电路板，BIOS固件、散热器等组成的，而GPU是显卡的核心部件。它以独立板卡的形式存在，插在主板的插槽上。独立显卡配备独立的内存，不占用系统内存，能够提供良好的显示效果和运行性能。

![30bcf7fd-2e40-47a0-aab1-993717021e8a](https://raw.githubusercontent.com/silence394/PicBed/PicGO/30bcf7fd-2e40-47a0-aab1-993717021e8a.png)

它能为我们提供更加真实的渲染画面。
![621d39ef-7288-4dc8-af56-9d878f85ced2](https://raw.githubusercontent.com/silence394/PicBed/PicGO/621d39ef-7288-4dc8-af56-9d878f85ced2.jpg)

在独立显卡中的GPU大概是这样的，
![0a53ecbb-c184-4d96-b612-2a6dfa83052a](https://raw.githubusercontent.com/silence394/PicBed/PicGO/0a53ecbb-c184-4d96-b612-2a6dfa83052a.jpg)

除了独立显卡之外还有集成显卡，简称集显，一般CPU自带一块集显，手机上用的也都是集显。它集成在了主板上，与CPU共用一块系统内存，一般都是紧挨着CPU芯片，这样CPU与GPU能够更快地进行数据交换。手机上CPU和GPU集成在SOC（System on Chip）上。

现代GPU具有高并发的特性，可以胜任以下工作：
- 图形绘制
- 物理模拟
- 海量计算
- AI运算
- 其他复杂计算如音视频解码、加密解密、科学计算、离线渲染等

### GPU的发展历程
一般认为GPU的发展历史是从Nvidia公司在1998年发布GeForce256开始，但是我认为早期的图形显示硬件也属于GPU的范畴。按照年代顺序介绍下GPU的发展。

1970年代，ANTIC和CTIA芯片为Atari-8位电脑提供硬件控制的图形和文字混合模式，以及其他视频效果的支持。ANTIC芯片是一个特殊用途的处理器，用于映射文字和图形数据到视频输出。

1980年代，第一个用以处理图形显示的大规模集成电路的单独芯片NEC 7220上市，实现了低消耗、高性能，是1980年代最著名的图形芯片之一。它支持高达1024x1024的分辨率，并为新兴的PC图形市场奠定了基础。

1984年，日立发布了ARTC HD63484，这是第一款用于PC的主要CMOS图形处理器。ARTC 在单色模式下能够显示高达4K的分辨率，并在1980年代后期用于许多PC图形卡和终端。

1990年代初期，Microsoft Windows的崛起引发人们对高性能、高清晰度二维位图运算（UNIX工作站和苹果公司的Macintosh原本是此领域的领导者）的兴趣。在个人电脑市场上，Windows的优势地位意味着台式机图形厂商可以集中精力发展单一的编程接口，图形设备接口。
windows

Nvidia

移动

https://en.wikipedia.org/wiki/Graphics_processing_unit#1970s

https://wenku.baidu.com/view/af266f85cc22bcd126ff0cd3.html?re=view

目前的GPU生产商，在PC端有Nvidia和AMD，在移动端有Imagination、ARM、高通等公司。


一般来说，集成显卡要占用内存作为显存，占用系统的带宽，影响系统的整体性能，所以在手机上常常会碰到带宽的性能瓶颈。

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


移动平台相比于PC平台，还是有很多不同的。从本质上看，是由功耗和体积两方面限制的，对于图形处理来说，主要是两点：第一，是有限的带宽。实际上，要增加计算能力，在功耗允许的情况下，堆核心并不是一件难事，事实上我们也看到了不少SOC集成了四核乃至“16核”GPU。但是难点在于，需要有足够的带宽去满足这颗强大的GPU，避免其出现“饿死”的情况。在左图的移动平台中，CPU、GPU和总线被共同集成在一颗芯片上，称之为SOC。整个SOC，包括其中的CPU和GPU，共享有限的内存带宽。即使是相对高端的，采用64bit内存位宽的一些SOC，如三星4412，高通8064等，也只是6.4 - 8.5GB/s的带宽，相比起PC平台主内存十几GB/s的带宽，和PC GPU GDDR5显存动辄几十，不少都超过100GB/s的带宽，只能说是少的可怜。相对另类的苹果在iPad4中，给A6X芯片搭配了128bit的LPDDR2-1066，带宽达到了17GB/s，用以喂饱强大的SGX 554 MP4 GPU，但相比PC平台依旧是小巫见大巫。因此，移动平台要在有限的带宽下实现合理的性能，在不少时候，瓶颈可能并不在于计算能力上，而在于带宽上。第二，相比PC平台的CPU，移动平台的CPU浮点较弱，在Cortex-A9开始虽然有所好转，但64bit的NEON跟桌面128bit甚至256bit的SIMD还是有显著差距，外加主频的差别。因此更多的计算也依赖硬件Vertex Shader去完成。因此，移动平台的GPU相对于PC平台，也会有一些不同。

移动GPU和桌面baiGPU最大的不同在于，移动GPU在设计上是有着明确的资源限制的：功耗和面积。其次才是性能功耗比。这样也导致移动GPU无论是拼性能还是性能功耗比，都远不如桌面GPU。
另外，对于现代GPU来说，无论是桌面还是移动，能看的指标只有两个：一个是FLOPS（Half Precision和Full Precision），一个就是Memory Bandwidth。至于核数之类的，那都是Market Techniques。此外性能和指标的关系，虽然是有，但没那么密切。

渲染架构https://www.gpuinsight.com/gpu_terms/

http://www.igao7.com/news/201406/1217-vv-gpu.html
https://www.zhihu.com/question/20720436
https://gameinstitute.qq.com/community/detail/103959
https://zhidao.baidu.com/question/564339480544098684.html
https://www.cnblogs.com/herenzhiming/articles/7183309.html