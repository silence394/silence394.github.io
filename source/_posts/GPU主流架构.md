---
title: 移动GPU主流架构
categories: 渲染
---

在TBR GPU上，将像素重新写到memorybuffer上的操作可以省略，但是shadering 还是要执行。

PowerVR的GPU上不存在这个问题，因为会先画深度。也叫TBDR.

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