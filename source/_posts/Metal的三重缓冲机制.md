---
title: Metal的三重缓冲机制
categories: 渲染
---
## 一、多重缓冲
CPU和GPU是并行工作的，CPU提交命令，GPU处理命令。当二者的任务处理完成，每帧的工作也就完成了。
### 1、CPU与GPU并行
CPU更新VertexBuffer，GPU读取VertexBuffer绘制。以此为例来说明，CPU和GPU的工作流程。

1. 每帧开始
2. CPU更新VertexBuffer
3. CPU提交渲染命令
4. GPU执行渲染命令
5. GPU读取VertexBuffer
6. GPU绘制像素到屏幕
7. 完成绘制

当CPU和GPU只拥有一份VertexBuffer的时候，CPU和GPU为避免存取冲突，必须要等待VertexBuffer没有引用时才能对其操作。很明显这种方案的效率是及其低下的。

![image](https://docs-assets.developer.apple.com/published/a8fcc3ae6f/3f778646-c337-42d1-977b-f5e8f1ac411c.png)

而为了最大化的利用CPU和GPU的并行性，CPU和GPU需要处于持续工作的状态。理想情况下，GPU在处理第一帧的绘制工作，CPU已经在处理渲染命令的组织了。在此种情况下仍然只有一种VertexBuffer的情况下，CPU Frame2更新VertexBuffer，比GPU Frame1里读取VertexBuffer要早的情况下，会出现错误的绘制结果。

![image](https://docs-assets.developer.apple.com/published/a8fcc3ae6f/e4f8f52c-ec6d-4c8d-bac3-e9e7a8a28be5.png)

为了解决以上的处理器空闲和对Buffer存取的冲突，可以创建多份VertexBuffer供CPU和GPU使用。CPU和GPU依次处理第1帧的VertexBuffer 1，第2帧的VertexBuffer 2，第3帧的VertexBuffer 3，以此类推。在这种结构下，CPU和GPU可以无等待的存取VertexBuffer，最大化的利用CPU和GPU的并发性。

![image](https://docs-assets.developer.apple.com/published/a8fcc3ae6f/5cd216dc-7d80-4d95-ac0b-b9fa783513f3.png)

### 2、多重缓冲
假设CPU提交命令比GPU处理命令所花费的时间要短一些。假设Buffer的数量无限制，不会发生资源存取冲突的情况，如果按照最简单的CPU和GPU不用等待、并行执行，会导致CPU与GPU之间的执行延时越来越大，即延迟越来越高。

![image](https://i.loli.net/2018/08/02/5b6314eb98ddc.jpg)

所以为了避免CPU与GPU的延迟越来越大，需要让CPU等待GPU的执行完成。假设有N重缓冲，第一次GPU处理命令需要与CPU处理第N帧同时开始，这样GPU处理完第1帧转向第2帧的时候，CPU可以处理第N+1帧。也就是说CPU与GPU会有N-1帧的延迟。如果N过大，缓冲的内存占用和延迟都是无法接受的。

![image](https://i.loli.net/2018/08/02/5b6314ebba1ff.jpg)

设想只有双重缓冲的情况。在GPU执行完命令进行swapbuffer的时候，由于FrontBuffer和BackBuffer需要进行数据的拷贝，GPU在这个时候是无法进行绘制的。所以GPU在双重缓冲下并没有满负载的工作。

![image](https://i.loli.net/2018/08/02/5b6314f003fa9.jpg)

为了最大化的压榨GPU，可以使用三重缓冲，它可以完美的消除GPU的等待时间。

![image](https://i.loli.net/2018/08/02/5b6314f00ef64.jpg)

## 二、Metal三重缓冲的实现
缓冲过多会造成延迟过大、内存占用过高，缓冲过少CPU和GPU会有空闲时间。使用三重缓冲能比较优美的解决这些问题。
### 1、信号量机制
### 2、实现

引用：
1. https://developer.apple.com/documentation/metal/fundamental_lessons/cpu_and_gpu_synchronization
2. http://gad.qq.com/article/detail/41867
3. https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/TripleBuffering.html#//apple_ref/doc/uid/TP40016642-CH5-SW1

4. https://gamedev.stackexchange.com/questions/82318/what-problem-does-double-or-triple-buffering-solve-in-modern-games