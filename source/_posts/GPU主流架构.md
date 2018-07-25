---
title: 手机GPU主流架构
categories: 渲染
---
路漫漫其修远兮。

在TBR GPU上，将像素重新写到memorybuffer上的操作可以省略，但是shadering 还是要执行。

PowerVR的GPU上不存在这个问题，因为会先画深度。也叫TBDR.

8. Do Not Use Discard


http://www.expreview.com/24705-3.html

https://blog.csdn.net/leonwei/article/details/79298381

http://alanliu90.hatenablog.com/entry/2017/08/27/%E7%A7%BB%E5%8A%A8%E5%B9%B3%E5%8F%B0GPU

PowerVR Hardware.Architecture Overview for Developers

