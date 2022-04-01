---
title: dart单线程模型
---

# Event Loop机制

1. **Dart是单线程的**：即Dart代码是有序的，按照在main函数出现的次序一个接一个地执行，不会被其他代码中断；

2. **Dart也支持异步**：单线程和异步并不冲突

   >这里有个大前提，那就是我们的App绝大多数时间都在等待。比如，等待用户点击，等待网络请求返回、等待文件IO结果等等，而这些等待行为并不是阻塞的；所以，基于这些特点，单线程模型可以在等待的过程中做别的事情，等真正需要响应结果了，再去做对应的处理。
   
3. **Event Loop**

   等待这个行为是通过Event Loop驱动的，事件队列`Event Queue`会把其他平行世界（比如Socket）完成的，需要主线程响应的事件放入其中。Dart有一个巨大的事件循环，在不断的轮询事件队列，取出事件，在主线程同步执行其回调函数，如下图所示：

   <img src="https://static001.geekbang.org/resource/image/0c/ec/0cb6e6d34295cef460e48d139bc944ec.png" alt="简化版 Event Loop" style="zoom:50%;" />

4. **异步任务Future**

   在Dart中，实际上有两个队列，一个是事件队列`Event Queue`，另一个是微任务队列`MicroTask Queue`。Event Loop 完整版的流程图，应该如下所示：

   <img src="https://static001.geekbang.org/resource/image/70/bc/70dc4e1c222ddfaee8aa06df85c22bbc.png" alt="Microtask Queue 与 Event Queue" style="zoom:50%;" />
   
   在每次事件循环中，Dart总是先去第一个微任务队列中查询是否有可执行的任务，如果没有，才会处理后续的事件队列的流程。
   
   1. 定义
   
      > Future：Dart为Event Queue的任务建议的一层封装，表示一个在未来时间才会完成的任务
   
   2. 使用
   
      把一个函数体放入Future，就完成了从同步任务到异步任务的包装。Future还提供了链式调用的能力，可以在异步任务执行完毕后依次执行链路上的其他函数体
   
   3. 执行流程
   
      在声明一个Future时，Dart会将异步任务的函数执行体放入Event Queue，然后立即返回，后续的代码将继续同步执行。而当同步执行的代码执行完毕后，Event Queue会按照加入事件队列的顺序，依次取出事件，最后同步执行Future的函数体及后续的then（**then与Future函数体共用一个事件循环**）
   
      示例代码：
   
      ```dart
        print('main Future 1');
        Future(() => print('Running in Future 2'))
            .then((value) => print('and then 1'))
            .then((value) => print('and then 2'));
        Future(() => null).then((value) => print('microTask 1'));
        Future(() => print('Running in Future 3'));
        print('main Future 2');
      ```
   
      但是，如果Future执行体已经执行完毕，但是这个Future的引用存在着，并且往里面加了一个then方法体，此时Dart会将后续加入的then方法体放入MicroTask Queue，尽快执行。
   
5. **异步函数**
