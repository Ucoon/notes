---
title: Flutter渲染原理
---

# Flutter 三棵树

Flutter 三棵树指的是：**Widget树**、**Element树**、**RenderObject树**

>Widgets configure, Elements manage, and RenderObjects paint

当应用启动时Flutter会遍历并创建所有的Widget会形成Widget Tree，同时与Widget Tree相对应，通过调用Widget上的`createElement()`创建每个Element对象，形成Element Tree，最后调用Element的`createRenderObject()`创建每个渲染对象，形成一个Render Tree（“渲染树”），总结一下，我们可以认为Flutter的UI系统包含三棵树：Widget树，Element树、渲染树。

**他们的依赖关系是：Element树根据Widget树生成，而渲染树又依赖于Element树**

![依赖关系](https://upload-images.jianshu.io/upload_images/24924109-0d3c0dc4a2b7798b.png?imageMogr2/auto-orient/strip|imageView2/2/w/644/format/webp)

## Widget

>**A widget is an immutable description of part of a user interface**

在源码中，对于`Widget`的解释是这样的：

```dart
/// Describes the configuration for an [Element].

/// Widgets are the central class hierarchy in the Flutter framework. A widget
/// is an immutable description of part of a user interface. Widgets can be
/// inflated into elements, which manage the underlying render tree.
```

1. 在Flutter中，`Widget`的功能是“描述一个UI元素的配置数据”
2. Flutter中真正代表屏幕上显示元素的类是`Element`，也就是说`Widget`只是描述`Element`的配置数据；
3. 需要注意的是`Widget`和`Element`是一一对应的，但并不和`RenderObject`对应，并且一个`Widget`可以对应多个`Element`

下面是`Widget`的部分代码：

```dart
@immutable // 不可变的
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });

  final Key? key;

  @protected
  @factory
  Element createElement();

  @override
  String toStringShort() {
    final String type = objectRuntimeType(this, 'Widget');
    return key == null ? type : '$type-$key';
  }

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
  ...
}
```

- `@immutable`代表`Widget`是不可变的，这会限制`Widget`中定义的属性（即配置信息）必须是不可变的（final）
- `key`：`key`的主要作用是决定是否在下一次`build`时复用旧的widget，决定的条件在`canUpdate()`中
- `canUpdate`：只要`newWidget`与`oldWidget`的`runtime`和`key`同时相等时就会用`newWidget`去更新`Element`对象的配置，否则就会创建新的`Element`

## Element

>**Element: an instantitation of a widget at a particular location in the tree**

# setState()分析

## 时序图

![setState](http://ucoon.tech/MyBlogImg/setState.jpg)

## 分析

1. State#setState(VoidCallback fn)

   ```dart
   @protected
   void setState(VoidCallback fn) {
       final Object? result = fn() as dynamic;
       _element!.markNeedsBuild();
     }
   ```

   1. 调用我们传入的`VoidCallback fn`
   2. 调用`_element!.markNeedsBuild()`

2. StatefulElement#markNeedsBuild()

   ```dart
   /// The object that manages the lifecycle of this element.
   /// 负责管理所有element的构建以及生命周期
   @override
   BuildOwner? get owner => _owner;
   
   void markNeedsBuild() {
       if (dirty)
         return;
       _dirty = true;//标记为脏
       owner!.scheduleBuildFor(this);
   }
   ```

   `BuildOwner`在`WidgetsBinding`的初始化中完成实例化，负责管理widget框架，**每个`Element`对象在`mount`到`element`树中之后都会从父节点获得它的引用**

3. BuildOwner#scheduleBuildFor(Element element)

   ```dart
   void scheduleBuildFor(Element element) {
       if (!_scheduledFlushDirtyElements && onBuildScheduled != null) {
         _scheduledFlushDirtyElements = true;
         onBuildScheduled!();
       }
       ///添加到_dirtyElements集合中
       _dirtyElements.add(element);
       element._inDirtyList = true;
   }
   ```

**总结**：

1. setState过程其实只是将当前对应的Element标记为脏，并且添加到`_dirtyElements`集合中
2. Element持有Widget，存放上下文信息，RenderObjectElement额外持有RenderObject，通过它遍历视图树，支撑UI结构

可以看出`setState`没做任何渲染相关的事，那么页面是如何重新绘制？关键点在于Flutter的渲染机制

# Flutter渲染机制

  **Flutter**作为一个UI框架，它解决的是一套代码在多端渲染的问题。在渲染管线上更加精简，加上自建渲染引擎，相比`ReactNative`、`Weex`以及`WebView`等方案，具有更好的性能体验。

## 渲染技术对比

目前在渲染技术上共有三种方案：

- `WebView`渲染：依赖`WebView`进行渲染，在功能和性能上有妥协，例如PhoneGap、Cordova等
- 原生渲染：上层拥抱W3C，通过中间层把前端框架翻译为原生控件，例如`ReactNative+React`、`Weex+Vue`的组合，这种方案多了一层转译层，性能上有损耗。随着原生系统的升级，在兼容性上也会有问题
- 自建渲染：自建渲染框架，底层使用Skia等图形库进行渲染，例如`Flutter`、`Unity`

Flutter由于其自建渲染引擎，贴近原生的实现方式，获得了优秀的渲染性能。

## 架构分析

![架构分析](https://ucc.alicdn.com/pic/developer-ecology/592009c400a440e99591d86db61c0eed.png)



参考

[How Flutter renders Widgets](https://www.youtube.com/watch?v=996ZgFRENMs)

[Element、BuildContext和RenderObject](https://book.flutterchina.club/chapter14/element_buildcontext.html#_14-2-1-element)

[原来我一直在错误的使用 setState()?](https://juejin.cn/post/6905996819445055495)

[从架构到源码：一文了解Flutter渲染机制](https://developer.aliyun.com/article/770384)
