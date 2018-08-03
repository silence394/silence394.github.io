---
title: Metal的三重缓冲机制
categories: 渲染
---
## 一、多重缓冲
CPU和GPU是并行工作的，CPU提交命令，GPU处理命令。当二者的任务处理完成，每帧的工作也就完成了。以下面的渲染流程为例，介绍多重缓冲的概念。

CPU更新VertexBuffer，GPU读取VertexBuffer绘制。以此为例来说明，CPU和GPU的工作流程。

1. 每帧开始
2. CPU更新VertexBuffer
3. CPU提交渲染命令
4. GPU执行渲染命令
5. GPU读取VertexBuffer
6. GPU绘制像素到屏幕
7. 完成绘制

### 1、CPU与GPU并行
当CPU和GPU只拥有一份VertexBuffer的时候，CPU和GPU为避免存取冲突，必须要等待VertexBuffer没有引用时才能对其操作。很明显这种方案的效率是及其低下的。

![image](https://docs-assets.developer.apple.com/published/a8fcc3ae6f/3f778646-c337-42d1-977b-f5e8f1ac411c.png)

而为了最大化的利用CPU和GPU的并行性，CPU和GPU需要处于持续工作的状态。理想情况下，GPU在处理第一帧的绘制工作，CPU已经在处理渲染命令的组织了。在此种情况下仍然只有一种VertexBuffer的情况下，CPU Frame2更新VertexBuffer，比GPU Frame1里读取VertexBuffer要早的情况下，会出现错误的绘制结果。

![image](https://docs-assets.developer.apple.com/published/a8fcc3ae6f/e4f8f52c-ec6d-4c8d-bac3-e9e7a8a28be5.png)

为了解决以上的处理器空闲和对Buffer存取的冲突，可以创建多份VertexBuffer供CPU和GPU使用，也就是多重缓冲。CPU和GPU依次处理第1帧的VertexBuffer 1，第2帧的VertexBuffer 2，第3帧的VertexBuffer 3，以此类推。在这种结构下，CPU和GPU可以无等待的存取VertexBuffer，最大化的利用CPU和GPU的并发性。

![image](https://docs-assets.developer.apple.com/published/a8fcc3ae6f/5cd216dc-7d80-4d95-ac0b-b9fa783513f3.png)

### 2、多重缓冲
假设CPU提交命令消耗的时间是GPU处理命令所花费的时间的3/4，Buffer的数量无限制，不会发生资源存取冲突的情况，如果按照最简单的CPU和GPU不用等待、并行执行，会导致CPU与GPU之间的执行延时越来越大，即延迟越来越高。

![image](https://i.loli.net/2018/08/02/5b6314eb98ddc.jpg)

所以为了避免CPU与GPU的延迟越来越大，需要让CPU等待GPU的执行完成。假设有N重缓冲，第一次GPU处理命令需要与CPU处理第N帧同时开始，这样GPU处理完第1帧转向第2帧的时候，CPU可以处理第N+1帧。也就是说CPU与GPU会有N-1帧的延迟。如果N过大，缓冲的内存占用和延迟都是无法接受的。

![image](https://i.loli.net/2018/08/02/5b6314ebba1ff.jpg)

设想双重缓冲的情况。在GPU执行完命令执行SwapBuffer的时候，由于FrontBuffer和BackBuffer需要进行数据的拷贝，GPU在这个时候是无法进行绘制的。所以GPU在双重缓冲下并没有满负载的工作。

![image](https://i.loli.net/2018/08/02/5b6314f003fa9.jpg)

为了最大化的压榨GPU，可以使用三重缓冲。由于有两个BackBuffer，在SwapBuffer的时候，GPU可以继续像另一个BackBuffer写入像素。它可以完美的消除GPU的等待时间。

![image](https://i.loli.net/2018/08/02/5b6314f00ef64.jpg)

## 二、Metal三重缓冲的实现
缓冲过多会造成延迟过大、内存占用过高，缓冲过少CPU和GPU会有空闲时间。在Metal上，使用信号量和三重缓冲能比较优美的解决这些问题。
### 1、信号量
信号量可以理解为一个特殊的变量，程序对它的操作都是原子性的，可以通过PV操作修改信号量。在IOS上可以通过以下代码控制信号量。
``` obj-c
dispatch_semaphore_t sem =  dispatch_semaphore_create(3); // 创建数目为3的信号量
dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);	// 如果信号量为0，则等待；否则信号量-1，继续执行后面的代码
dispatch_semaphore_signal(sem);	// 释放信号量，信号量+1
```
通过这三个函数能够实现CPU与GPU之间的同步。
### 2、Metal的实现
帧之间循环操作缓冲，当GPU完成命令后通知CPU执行后续的任务，从而实现CPU与GPU之间的高并发。
``` obj-c
static const NSUInteger kMaxInflightBuffers = 3;
/* Additional constants */
 
@implementation Renderer
{
    dispatch_semaphore_t _frameBoundarySemaphore;
    NSUInteger _currentFrameIndex;
    NSArray <id <MTLBuffer>> _dynamicDataBuffers;
    /* Additional variables */
}
 
- (void)configureMetal
{
    // Create a semaphore that gets signaled at each frame boundary.
    // The GPU signals the semaphore once it completes a frame's work, allowing the CPU to work on a new frame
    _frameBoundarySemaphore = dispatch_semaphore_create(kMaxInflightBuffers);
    _currentFrameIndex = 0;
    /* Additional configuration */
}
 
- (void)makeResources
{
    // Create a FIFO queue of three dynamic data buffers
    // This ensures that the CPU and GPU are never accessing the same buffer simultaneously
    MTLResourceOptions bufferOptions = /* ... */;
    NSMutableArray *mutableDynamicDataBuffers = [NSMutableArray arrayWithCapacity:kMaxInflightBuffers];
    for(int i = 0; i < kMaxInflightBuffers; i++)
    {
        // Create a new buffer with enough capacity to store one instance of the dynamic buffer data
        id <MTLBuffer> dynamicDataBuffer = [_device newBufferWithLength:sizeof(DynamicBufferData) options:bufferOptions];
        [mutableDynamicDataBuffers addObject:dynamicDataBuffer];
    }
    _dynamicDataBuffers = [mutableDynamicDataBuffers copy];
}
 
- (void)update
{
    // Advance the current frame index, which determines the correct dynamic data buffer for the frame
    _currentFrameIndex = (_currentFrameIndex + 1) % kMaxInflightBuffers;
 
    // Update the contents of the dynamic data buffer
    DynamicBufferData *dynamicBufferData = [_dynamicDataBuffers[_currentFrameIndex] contents];
    /* Perform updates */
}
 
- (void)render
{
    // Wait until the inflight command buffer has completed its work
    dispatch_semaphore_wait(_frameBoundarySemaphore, DISPATCH_TIME_FOREVER);
 
    // Update the per-frame dynamic buffer data
    [self update];
 
    // Create a command buffer and render command encoder
    id <MTLCommandBuffer> commandBuffer = [_commandQueue commandBuffer];
    id <MTLRenderCommandEncoder> renderCommandEncoder = [commandBuffer renderCommandEncoderWithDescriptor:_renderPassDescriptor];
 
    // Set the dynamic data buffer for the frame
    [renderCommandEncoder setVertexBuffer:_dynamicDataBuffers[_currentFrameIndex] offset:0 atIndex:0];
    /* Additional encoding */
    [renderCommandEncoder endEncoding];
 
    // Schedule a drawable presentation to occur after the GPU completes its work
    [commandBuffer presentDrawable:view.currentDrawable];
 
    __weak dispatch_semaphore_t semaphore = _frameBoundarySemaphore;
    [commandBuffer addCompletedHandler:^(id<MTLCommandBuffer> commandBuffer) {
        // GPU work is complete
        // Signal the semaphore to start the CPU work
        dispatch_semaphore_signal(semaphore);
    }];
 
    // CPU work is complete
    // Commit the command buffer and start the GPU work
    [commandBuffer commit];
}
 
```

引用：
1. https://developer.apple.com/documentation/metal/fundamental_lessons/cpu_and_gpu_synchronization
2. http://gad.qq.com/article/detail/41867
3. https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/TripleBuffering.html#//apple_ref/doc/uid/TP40016642-CH5-SW1

4. https://gamedev.stackexchange.com/questions/82318/what-problem-does-double-or-triple-buffering-solve-in-modern-games