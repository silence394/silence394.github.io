---
title: AlphaBlend与AlphaTest
categories: 渲染
---
### AlphaBlend和AlphaTest是什么
- AlphaBlend：关闭DepthWrite，需要保证先绘制不透明物体，并对AlphaBlend的物体按照深度排序再绘制。对半透明的片元相互穿插的问题，没有一个完美的解决方案。可以拆分网格或者画两次AlphaBlend的网格，第一次开启深度写入并写入深度，第二次做正常的AlphaBlend，能够实现半透的效果。
- AlphaTest:在PixelShader里判断是否要discard当前的fragment。如果被discard就会舍弃当的像素；否则按照正常的方式处理颜色和深度。不会有半透明的效果，只有透或者不透。

### 效率比较
在一个平视角，满屏草的场景下，对OppoR9S手机测试，AlphaBlend(20ms)比AlphaTest(30ms)快1/3左右。但很多人给出AlphaBlend比AlphaTest慢的数据，可能跟大家测试的环境、手机、引擎等有很大关系。不过大家都认可的意见是以数据说话，通过测试来得到结论。

### 引用
1. https://blog.csdn.net/candycat1992/article/details/41599167
2. https://answer.uwa4d.com/question/59c09367a3d42c411c37a0cd/
3. https://www.zhihu.com/question/27875435
4. PowerVR 的资料。
5. https://zhuanlan.zhihu.com/p/33127345
6. https://www.zhihu.com/question/29904258
7. https://blog.csdn.net/leonwei/article/details/79298381
8. http://gad.qq.com/article/detail/15884
