---
title: 移动GPU主流架构
categories: 性能
---

在TBR GPU上，将像素重新写到memorybuffer上的操作可以省略，但是shadering 还是要执行。

PowerVR的GPU上不存在这个问题，因为会先画深度。也叫TBDR.


Single Programe Multiple Data

Single Instrction Multiple Data

Single Instrction multiple Thread

Single Instruction Multiple Thread

### GPU架构
PowerVR使用了一种叫做Tile Based Deferred Rendering的架构，简称TBDR。在图形管线里尽早地减少冗余操作，减少带宽和电量的使用。

CPU处理多分支少数据的任务，线程处理的任务一般与其他任务无关。一般CPU的核心数为四到八个。

GPU在多个线程中执行相同的命令，处理不同的数据(SIMD)。SIMD的好处是相当数量的线程以一个很高的效率并行执行命令。很擅长处理大量相关的数据，如图像处理这种需要一次处理大量数据的算法。

一般3到4个元素，对SIMD是比较合适的，如果处理较少的数据，会有浪费的现象。如处理3个元素，只有75%的效率运行这些指令。

#### Immediate Mode Renderer(IMR)
#### Tile Based Renderer(TBR)
#### Tile Based Deferred Renderer(TBDR)

不要在一帧里面频繁的切换framebuffer
每一次bindunbind都需要对整个framebuffer的一次立即绘制

8. Do Not Use Discard

参考：
1. http://www.expreview.com/24705-3.html

2. https://blog.csdn.net/leonwei/article/details/79298381

3. http://alanliu90.hatenablog.com/entry/2017/08/27/%E7%A7%BB%E5%8A%A8%E5%B9%B3%E5%8F%B0GPU

4. PowerVR Hardware.Architecture Overview for Developers

5. https://blog.csdn.net/leonwei/article/details/79298381
6. https://blog.csdn.net/u013467442/article/details/40686523
7. https://blog.csdn.net/pizi0475/article/details/58627435
8. https://www.zhihu.com/question/20720436/answer/151341206
9. https://www.zhihu.com/question/20720436
10. https://stackoverflow.com/questions/4853856/why-are-draw-calls-expensive
11. https://docs.microsoft.com/zh-cn/windows/desktop/direct3d9/accurately-profiling-direct3d-api-calls
12. http://c0de517e.blogspot.com/2008/04/gpu-part-1.html
13. https://www.zhihu.com/question/54037063
14. http://www.cnblogs.com/gameknife/p/3515714.html
