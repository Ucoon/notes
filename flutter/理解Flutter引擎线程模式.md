---
title: 理解Flutter引擎线程模式 
---

# Flutter体系结构

![flutter_system_overview](http://ucoon.gitee.io/myblogimg/flutter_system_overview.png)

## Framework

Framework是我们直接接触到的，它使用dart实现，包括Material Design风格的Widget，CuperTino风格的Widget，文本/图片/按钮等基础的Widgets，`Rendering`渲染、`Animation`动画、`Painting`图形绘制、`Gestures`手势等。

## Engine

Engine引擎层使用C++实现，主要包括：Skia，Dart和shell。

- Skia是开源的二维图形库，提供了适用于多种软硬件平台的通用API

- shell：不同的平台有不同的shell

## Embedder

Embedder是一个嵌入层，即把Flutter嵌入到各个平台上去，这里做的主要工作包括渲染Surface设置，线程设置，以及插件等，为Engine创建和管理线程，作用是把Engine的task runners运行在嵌入层管理的线程上。

从这里可以看出，Flutter的平台相关层很低，平台只是提供一个画布，剩余的所有渲染相关的逻辑都在Flutter内部，这就使得它具有了很好的跨端一致性。

# Flutter 线程模型

Flutter Engine是不创建和管理线程的，是由Embedder为Engine创建和管理线程（包括线程里的消息循环），Flutter在Engine中定义了四种`Task Runners`，`Task Runners`是需要运行在平台提供的线程上，这四种`Task Runners`分别是

- Platform Task Runner
- UI Task Runner
- GPU Task Runner
- IO Task Runner

## Platform Task Runner

1. Flutter Engine的主Task Runner，运行Platform Task Runner的线程叫Platform Thread，Platform Thread必须要运行在**平台的Main Thread**上。
2. 一个Flutter Engine实例对应一个Platform Thread：一个Flutter应用启动的时候会创建一个Engine实例，Engine创建的时候会创建一个线程供Platform Runner使用
3. 跟Flutter Engine的所有交互（接口调用）以及处理来自平台的消息都必须发生在Platform Thread，试图在其他线程中调用Flutter Engine会导致异常
4. 阻塞Platform Thread不会直接导致Flutter 应用的卡顿（跟iOS Android主线程不同），但是建议复杂计算逻辑操作不要放在Platform Thread，而是放在其他线程（不包括上述的四个线程）。长时间卡住Platform Thread应用可能会被系统Watchdot强行杀死

## UI  Task Runner

1. UI Task Runner被Flutter Engine用于执行Dart root isolate代码，Root isolate比较特殊，它绑定了Flutter需要的函数方法，运行应用的main code，UI Task Runner运行所在的线程对应到平台的线程，其实是**子线程**
2. 调度提交渲染帧：
   1. Root isolate通知Flutter Engine有帧需要渲染
   2. Flutter Engine通知平台，需要在下一个vsync的时候得到通知
   3. 平台等待下一个vsync
   4. 对创建的对象和Widgets进行Layout并生成一个Layer Tree，这个Tree马上被提交到Flutter Engine

3. Root Isolate处理来自Navite Plugins的消息响应，Timers，Microtasks和异步IO操作
4. 阻塞这个线程会直接导致Flutter应用卡顿掉帧，繁重计算建议其放到独立的Isolate执行

## GPU  Task Runner

1. GPU Task Runner被用于执行设备GPU的相关调用：UI Task Runner创建的Layer Tree信息是平台不相关的，具体如何实现绘制取决于具体平台和方式，可以是OpenGL，Vulkan、软件绘制或者其他Skia配置的绘图实现。
2. UI Task Runner和GPU Task Runner跑在不同的线程：基于Layer Tree的处理时长和GPU帧显示到屏幕的耗时，GPU Task Runner可能会延迟下一帧在UI Task Runner的调度。存在这种可能，UI Task Runner在已经准备好下一帧的情况下，GPU Task Runner却还在向GPU提交上一帧。这种延迟调度机制确保不让UI Task Runner分配过多的任务给GPU Task Runner。
3. GPU Task Runner的过载会导致Flutter应用的卡顿：建议为每一个Engine实例都新建一个专用的GPU Runner线程

## IO  Task Runner

1. IO Task Runner主要功能是从图片存储中读取压缩的图片格式，将图片数据进行处理，为GPU Task Runner的渲染做好准备。在Textture的准备过程中，IO Runner首先要读取压缩的图片二进制数据，将其解压转换成GPU能够处理的格式，然后将数据上传到GPU。
1. IO Task Runner不会直接导致Flutter应用卡顿，但是可能会导致图片和其他一些资源加载的延迟，间接影响性能，所以还是建议为IO Task Runner创建一个专用的线程

# 各个平台默认的Runner线程实现

1. Platform Task Runner

| Android |  iOS   |
| :-----: | :----: |
| 主线程  | 主线程 |

2. UI Task Runner

| Android |  iOS   |
| :-----: | :----: |
| 子线程  | 子线程 |

3. GPU Task Runner

| Android |  iOS   |
| :-----: | :----: |
| 子线程  | 子线程 |

4. IO Task Runner

| Android |  iOS   |
| :-----: | :----: |
| 子线程  | 子线程 |
