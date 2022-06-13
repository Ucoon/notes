---
title: Flutter状态管理：Provider
---

# InheritedWidget

## 简介

`InheritedWidget`提供了一种在widget树中从上到下共享数据的方式，能实现组件跨级传递数据。比如我们在应用的根widget中通过`InheritedWidget`共享了一个数据，那么我们便可以在任意子widget中来获取该共享的数据！

应用：如Flutter SDK中正是通过`InheritedWidget`来共享`Theme`和`Local`（当前语言环境）信息

```dart
class ShareDataWidget extends InheritedWidget {
  final int data; //需要在子树中共享的数据，保存点击次数
  const ShareDataWidget({
    Key? key,
    required this.data,
    required Widget child,
  }) : super(key: key, child: child);

  //定义一个便捷方法，方便子树中的widget获取共享数据
  static ShareDataWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
  }

  @override
  bool updateShouldNotify(covariant ShareDataWidget oldWidget) {
    return oldWidget.data != data;
  }
}
```

## 刷新机制

![](http://ucoon.tech/MyBlogImg/InheritedWidget%E5%88%B7%E6%96%B0%E6%9C%BA%E5%88%B6.jpg)

## didChangeDependencies

`State`的生命周期之一，它会在**“依赖”**发生变化时被Flutter框架调用，而这个“依赖”指的就是子widget是否使用了父widget中`InheritedWidget`的数据！如果使用了，则代表子widget有依赖；如果没有使用则代表没有依赖。这种机制可以使子widget在所依赖的`InheritedWidget`变化时来更新自身！在数据发生变化时只对使用该数据的widget更新是合理并且性能友好的。

**应该在didChangeDependencies中做什么？**

如果需要在依赖改变后执行一些昂贵的操作，比如网络请求，这时最好的方式就是在此方法中执行，这样可以避免每次`build()`都执行这些昂贵操作。

**Flutter框架是怎么知道子widget有没有依赖父级InheritedWidget**

`dependOnInheritedWidgetOfExactType()`源码：

```dart
@override
InheritedWidget dependOnInheritedWidgetOfExactType({ Object aspect }) {
  assert(_debugCheckStateIsActiveForAncestorLookup());
  final InheritedElement ancestor = _inheritedWidgets == null ? null : _inheritedWidgets[T];
  if (ancestor != null) {
    return dependOnInheritedElement(ancestor, aspect: aspect) as T;
  }
  _hadUnsatisfiedDependencies = true;
  return null;
}

@override
InheritedWidget dependOnInheritedElement(InheritedElement ancestor, { Object aspect }) {
  assert(ancestor != null);
  _dependencies ??= HashSet<InheritedElement>();
  _dependencies.add(ancestor);
  ancestor.updateDependencies(this, aspect);
  return ancestor.widget;
}
```

可以看到在调用`dependOnInheritedWidgetOfExactType`时，`InheritedWidget`和依赖它的子widget注册了依赖关系，之后当`InheritedWidget`发生变化时，就会更新依赖它的子widget，也就是会调用这些子widget的`didChangeDependencies()`和`build()`。

# Provider

![Provider原理](https://book.flutterchina.club/assets/img/7-3.531c5fdf.png)

原理：基于发布者-订阅者模式，Model变化后会自动通知`ChangeNotifierProvider`（订阅者），`ChangeNotifierProvider`内部会重新构建`InheritedWidget`，而依赖该`InheritedWidget`的子`widget`就会更新。

使用Provider 优点：

1. 业务代码更关注数据了，只要更新Model，则UI会自动更新，而不用在状态改变后去手动调用`setState()`来显示更新页面
2. 数据改变的消息传递被屏蔽了，我们无需手动去处理状态改变事件的发布和订阅，这一切都被封装在Provider中了
3. 在大型复杂应用中，尤其是需要全局共享的状态非常多时，使用Provider将会大大简化代码逻辑，降低出错的概率，提高开发效率
