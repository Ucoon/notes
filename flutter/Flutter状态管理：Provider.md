---
title: Flutter状态管理：Provider
---

# InheritedWidget

## 简介

`InheritedWidget`提供了一种在widget树中从上到下共享数据的方式，能实现组件跨级传递数据。比如我们在应用的根widget中通过`InheritedWidget`共享了一个数据，那么我们便可以在任意子widget中来获取该共享的数据！

应用：如Flutter SDK中正是通过`InheritedWidget`来共享`Theme`和`Local`（当前语言环境）信息

## 数据传输

示例：

```dart
class ShareDataWidget extends InheritedWidget {
  final int data; //需要在子树中共享的数据，保存点击次数
  const ShareDataWidget({
    Key? key,
    required this.data,
    required Widget child,
  }) : super(key: key, child: child);

  //定义一个便捷方法，方便子树中的widget获取共享数据
  static ShareDataWidget? of(BuildContext context, {bool listen = true}) {
    ShareDataWidget? shareDataWidget;
    if (listen) {
      shareDataWidget =
          context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
    } else {
      shareDataWidget =
          context.getElementForInheritedWidgetOfExactType<ShareDataWidget>()
              as ShareDataWidget?;
    }
    return shareDataWidget;
  }

  @override
  bool updateShouldNotify(covariant ShareDataWidget oldWidget) {
    return oldWidget.data != data;
  }
}
```

原理：

1. `dependOnInheritedWidgetOfExactType`方法将在下文解析

2. `getElementForInheritedWidgetOfExactType`方法：

   ```dart
   @override
   InheritedElement getElementForInheritedWidgetOfExactType<T extends InheritedWidget>() {
     final InheritedElement ancestor = _inheritedWidgets == null ? null : _inheritedWidgets[T];
     return ancestor;
   }
   ```

   通过上述两个函数既可以获取到InheritedElement实例，继而拿到了共享的数据

## 刷新机制

>InheritedElement和Element之间有一些交互，实际上自带了一套刷新机制

![](http://ucoon.tech/MyBlogImg/InheritedWidget%E5%88%B7%E6%96%B0%E6%9C%BA%E5%88%B6.jpg)

- InheritedElement存子节点Element：`_dependents`，这个变量用来存储需要刷新的子Element

  ```dart
  class InheritedElement extends ProxyElement {
    InheritedElement(InheritedWidget widget) : super(widget);
  
    final Map<Element, Object?> _dependents = HashMap<Element, Object?>();
  
    @override
    void debugDeactivated() {
      assert(() {
        assert(_dependents.isEmpty);
        return true;
      }());
      super.debugDeactivated();
    }
  
    @protected
    Object? getDependencies(Element dependent) {
      return _dependents[dependent];
    }
      
    @protected
    void setDependencies(Element dependent, Object? value) {
      _dependents[dependent] = value;
    }
      
    @protected
    void updateDependencies(Element dependent, Object? aspect) {
      setDependencies(dependent, null);
    }
  }
  ```

- InheritedElement刷新子Element

  1. 在`notifyClients`方法中，循环`_dependents`存储的Element，传入`notifyDependent`
  2. 在`notifyDependent`中，传入Element调用自身`didChangeDependencies`方法
  3. Element的`didChangeDependencies`方法会调用`markNeedsBuild`，来刷新自身

- **InheritedWidget的子节点是如何将自身Element添加到`_dependents`**

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
  
  - 在`dependOnInheritedElement`方法中，会传入`InheritedElement`实例`ancestor`
  - `ancestor`会调用`updateDependencies`方法，将自身的Element实例传入，这样就添加到`_dependents`中了
  
  可以发现：**调用`dependOnInheritedWidgetOfExactType()` 和 `getElementForInheritedWidgetOfExactType()`的区别就是前者会注册依赖关系，而后者不会**
  
  

## didChangeDependencies

`State`的生命周期之一，它会在**“依赖”**发生变化时被Flutter框架调用，而这个“依赖”指的就是子widget是否使用了父widget中`InheritedWidget`的数据！如果使用了，则代表子widget有依赖；如果没有使用则代表没有依赖。这种机制可以使子widget在所依赖的`InheritedWidget`变化时来更新自身！在数据发生变化时只对使用该数据的widget更新是合理并且性能友好的。

**应该在didChangeDependencies中做什么？**

如果需要在依赖改变后执行一些昂贵的操作，比如网络请求，这时最好的方式就是在此方法中执行，这样可以避免每次`build()`都执行这些昂贵操作。

# Provider

> `Provider`是对`InheritedWidget`组件的上层封装，使其更易用，更易复用。

![Provider原理](https://book.flutterchina.club/assets/img/7-3.531c5fdf.png)

原理：基于发布者-订阅者模式，Model变化后会自动通知`ChangeNotifierProvider`（订阅者），`ChangeNotifierProvider`内部会重新构建`InheritedWidget`，而依赖该`InheritedWidget`的子`widget`就会更新。

使用Provider 优点：

1. 业务代码更关注数据了，只要更新Model，则UI会自动更新，而不用在状态改变后去手动调用`setState()`来显示更新页面
2. 数据改变的消息传递被屏蔽了，我们无需手动去处理状态改变事件的发布和订阅，这一切都被封装在Provider中了
3. 在大型复杂应用中，尤其是需要全局共享的状态非常多时，使用Provider将会大大简化代码逻辑，降低出错的概率，提高开发效率

参考：

[【源码篇】Flutter Provider的另一面（万字图文+插件）](https://juejin.cn/post/6968272002515894303)

[数据共享（InheritedWidget）](https://book.flutterchina.club/chapter7/inherited_widget.html)
