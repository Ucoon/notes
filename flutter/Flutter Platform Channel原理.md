---
title: Flutter Platform Channel原理
---

概述：本文不讲述如何编写代码，只学习其原理，如何编写可参考[Flutter-Plugin开发与发布](http://ucoon.tech/2022/01/27/Flutter-Plugin%E5%BC%80%E5%8F%91%E4%B8%8E%E5%8F%91%E5%B8%83/)

>Flutter平台特定的API支持不依赖于代码生成，而是依赖于灵活的消息传递的方式：
>
>应用的Flutter部分通过平台通道（Platform Channel）将传递的数据编码成消息的形式，跨线程发送到其应用程序的所在的宿主（iOS或Android）
>
>宿主监听的平台通道，并接收该消息，然后它会调用特定于该平台的API（使用原生编程语言），并将结果数据用过同样方式原路发送回客户端，即应用程序的Flutter部分
>
>整个过程的消息和响应是异步传递的，所以不会直接阻塞用户界面

# 官方架构图：

![Flutter Platform Channel](https://flutter.cn/docs/assets/images/docs/PlatformChannels.png)

# 流程图

## MethodChannel调用流程

![MethodChannel调用流程](http://ucoon.tech/MyBlogImg/MethodChannel调用流程.jpg)

小结：

1. Dart层使用codec对根据方法名(`channel method name`)和参数(`channel method param`)构建得到的对象进行编码，然后通过dart的类似JNI的本地接口，调用`SendPlatformMessage`，传递给c++层
2. c++层通过持有java对象`flutterJNI`的方法调用将消息传递到java层
3. java层解码接收到的消息，并根据`channel name`获取到对应的`handlerInfo`，调用相应的逻辑处理

## MethodChannel返回流程

![MethodChannel返回流程](http://ucoon.tech/MyBlogImg/MethodChannel返回流程.jpg)

小结：

1. java层得到结果后进行编码，通过JNI将响应结果返回给c++层
2. c++层将结果通过发送时保存的dart响应方法对象回调给dart层
3. dart层通过回调方法对结果数据进行处理，然后通过codec解码数据做后续操作

>总结：MethodChannel的执行流程涉及到主线程和UI线程的交互，代码从Dart层到C++层再到Java层，执行完相应逻辑后原路返回，从Java层到C++层再到Dart层

参考：

[深入理解Flutter的Platform Channel机制](http://gityuan.com/2019/08/10/flutter_channel/)

[全面解析Flutter Platform Channel原理](https://zhuanlan.zhihu.com/p/139600343)
