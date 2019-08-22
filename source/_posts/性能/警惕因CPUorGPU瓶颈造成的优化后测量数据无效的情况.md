---
title: 
categories: 警惕因CPUorGPU瓶颈造成的优化后测量数据无效的情况
---
### 一、三缓冲条件下CPU和GPU的工作模式
参考[Metal的三缓冲机制](https://silence394.github.io/2018/08/01/Metal%E7%9A%84%E4%B8%89%E9%87%8D%E7%BC%93%E5%86%B2%E6%9C%BA%E5%88%B6/)，可以认识到：CPU与GPU是并行的工作状态，可以创建多重缓冲让CPU与GPU之间有更好的并发性。

![](https://i.loli.net/2018/08/04/5b658a83cdeac.jpg)

测试性能时观测的帧间隔，更接近于CPU执行时间和GPU执行时间的更大者。判定一个优化是否有效，需要以CPU瓶颈还是GPU瓶颈为前提，如果优化的是一方的工作，但是瓶颈却在令一方，我们通过数据（帧间隔）无法看到性能的提升。
### 二、CPU瓶颈与GPU瓶颈
当CPU为瓶颈时，CPU和GPU的工作状态如下：

![](http://ww1.sinaimg.cn/mw690/c5c3a364ly1g68jv25f13j20pu06w3ye.jpg)

测试**GPU**的优化是无法体现在数帧间隔上的。

当GPU为瓶颈时，CPU和GPU的工作状态如下：

![](http://ww1.sinaimg.cn/mw690/c5c3a364ly1g68jv240vtj20p906o746.jpg)

测试**CPU**的优化是无法体现在帧间隔上的。

所以当测试CPU/GPU优化的时候，一定要设法让相关的硬件成为瓶颈，这样测得的性能数据可以最大化的体现优化的作用。