---
title: Dart-tips
---

# Dart 的特性

## JIT与AOT

>JIT（Just In Time，即时编译）：在运行时即时编译，在开发周期中使用，可以动态下发和执行代码，并发测试效率高，但运行速度和执行性能则会因为运行时即时编译受到影响
>
>AOT（Ahead of Time，运行前编译）：提前编译，可以生成被直接执行的二进制代码，运行速度快，执行性能表现好，但每次执行前都需要提前编译，开放测试效率低

Flutter最受欢迎的功能之一热重载，正是基于JIT特性，而在发布期使用AOT，就不需要像`React Native`那样在跨平台`JavaScript`代码和原生`Android`、`iOS`代码之间建立低效的方法调用映射关系，所以说，**Dart具有运行速度快、执行性能好的特点**。

## 内存分配与垃圾回收

## 单线程模型



1. var;//关键词，可以接收任何类型的变量，Dart中var变量一旦赋值，类型便会确定，则不能再改变其类型(Dart是一个强类型语言，区别js)

2. dynamic;//关键词，动态任意类型，**编译阶段不检查**类型，并且可以在后期改变赋值类型

3. Object;//动态任意类型，**编译阶段**检查类型，Dart所有对象的根基类(包括Function和Null)

4. final和const;//常量，const变量是一个编译时常量，final变量在第一次使用时被初始化

5. 函数

6. 异步支持，待实际编码过程中学习

7. 在Dart语言中使用下划线前缀标识符，会强制其变成私有的，如：_suggestions

8. 语法 "i ~/ 2" 表示i除以2，但返回值是整形（向下取整）

9. => 箭头函数：可以有返回值，返回值就是这条语句的值

10. @immutable：
        来源：https://api.flutter.dev/flutter/meta/immutable-constant.html
        说明：被@immutable注解标明的类或者子类都必须是不可变的

11. class get set方法

    ```dart
    class User {
      String _userName;
      set uesrName(String userName) => _userName = userName;
      String get userName => _userName;
    }
    ```

12. with(混入mixins)

    https://juejin.im/post/6844903764441202702