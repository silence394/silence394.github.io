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

### 2、三重缓冲
一般来说CPU

## 二、Metal三重缓冲的实现
### 1、信号量机制
### 2、实现

引用：
1. https://developer.apple.com/documentation/metal/fundamental_lessons/cpu_and_gpu_synchronization
2. http://gad.qq.com/article/detail/41867
3. https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/TripleBuffering.html#//apple_ref/doc/uid/TP40016642-CH5-SW1