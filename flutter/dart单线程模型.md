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

# 异步任务

   在Dart中，实际上有两个队列，一个是事件队列`Event Queue`，另一个是微任务队列`MicroTask Queue`。Event Loop 完整版的流程图，应该如下所示：

   <img src="https://static001.geekbang.org/resource/image/70/bc/70dc4e1c222ddfaee8aa06df85c22bbc.png" alt="Microtask Queue 与 Event Queue" style="zoom:50%;" />

   在每次事件循环中，Dart总是先去第一个微任务队列中查询是否有可执行的任务，如果没有，才会处理后续的事件队列的流程。

   1. 定义

      > Future：Dart为Event Queue的任务建议的一层封装，表示一个在未来时间才会完成的任务

   2. 使用

      把一个函数体放入Future，就完成了从同步任务到异步任务的包装。Future还提供了链式调用的能力，可以在异步任务执行完毕后依次执行链路上的其他函数体

   3. 执行流程

      在声明一个Future时，Dart会将异步任务的函数执行体放入Event Queue，然后立即返回，后续的代码将继续同步执行。而当同步执行的代码执行完毕后，Event Queue会按照加入事件队列的顺序，依次取出事件，最后同步执行Future的函数体及后续的then（**then与Future函数体共用一个事件循环**）

      但是，如果Future执行体已经执行完毕，但是这个Future的引用存在着，并且往里面加了一个then方法体，此时Dart会将后续加入的then方法体放入`MicroTask Queue`，尽快执行。

   **案例**：

   ```dart
   
   Future(() => print('f1'));//声明一个匿名Future
   Future fx = Future(() =>  null);//声明Future fx，其执行体为null
   
   //声明一个匿名Future，并注册了两个then。在第一个then回调里启动了一个微任务
   Future(() => print('f2')).then((_) {
     print('f3');
     scheduleMicrotask(() => print('f4'));
   }).then((_) => print('f5'));
   
   //声明了一个匿名Future，并注册了两个then。第一个then是一个Future
   Future(() => print('f6'))
     .then((_) => Future(() => print('f7')))
     .then((_) => print('f8'));
   
   //声明了一个匿名Future
   Future(() => print('f9'));
   
   //往执行体为null的fx注册了了一个then
   fx.then((_) => print('f10'));
   
   //启动一个微任务
   scheduleMicrotask(() => print('f11'));
   print('f12');
   ```

   执行结果如下：

   ```dart
   f12-->f11-->f1-->f10-->f2-->f3-->f5-->f4-->f6-->f9-->f7-->f8
   ```

# 异步函数

> 对于一个异步函数来说，其返回时内部执行动作并未结束，因此需要返回一个Future对象，供调用者使用。调用者根据Future对象，来决定：是在Future对象上注册一个then，等Future的执行体结束了再进行异步处理；还是一直同步等待Future执行体结束。

**同步等待**：需要在调用处使用**await**关键字，并且在调用处的函数体使用**async**关键字：**Dart的await并不是阻塞等待，而是异步等待**，并且**await与async只对调用上下文的函数有效，并不向上传递**

**案例：**

```dart
//声明了一个延迟2秒返回Hello的Future，并注册了一个then返回拼接后的Hello 2019
Future<String> fetchContent() => 
  Future<String>.delayed(Duration(seconds:2), () => "Hello")
    .then((x) => "$x 2019");
//异步函数会同步等待Hello 2019的返回，并打印
func() async => print(await fetchContent());

main() {
  print("func before");
  func();
  print("func after");
}
```

执行结果如下：

```dart
func before-->func after-->Hello 2019
```

# Isolate

Dart为了利用多核CPU，将CPU层面的密集型计算进行了隔离设计，提供多线程机制，即`Isolate`。每个`Isolate`资源隔离，都有自己的Event Loop、Event Queue和Microtask Queue，`Isolate`之间的资源共享通过消息机制通信（和进程一样）

此外Flutter中提供了执行并发计算任务的快捷方式—`compute函数`，其内部对`Isolate`的创建和双向通信进行了封装。

# 总结

- Future适合耗时小于16ms的操作
- 可以通过compute()进行耗时操作
- Dart是单线程模型，但也支持多线程，线程间数据不互通，可通过消息机制通信
