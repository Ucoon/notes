### Flutter的生命周期

1. Widget的生命周期

   1. `Widget`是有状态还是无状态的，取决于他们依赖于状态的变化：
      1. 有状态：交互或者数据改变导致`Widget`改变，例如改变文案
      2. 无状态：不会被改变的`Widget`，例如纯展示界面

   ```dart
   /// Widgets are the central class hierarchy in the Flutter framework. A widget
   /// is an immutable description of part of a user interface. Widgets can be
   /// inflated into elements, which manage the underlying render tree.
   在Flutter中，Widget的功能是“描述一个UI元素的配置数据”，所以它是不可变的，变的是Widget里面的状态，也就是State。
   ```

   2. `有状态组件StatefulWidget`和`无状态组件StatelessWidget`

      1. ```有状态组件StatefulWidget```：有需要管理的内部状态，使用setState来管理状态改变。调用setState通知Flutter框架某个状态发生了变化，Flutter会重新运行build方法，应用程序便可以显示最新的状态
         - `StatefulWidget`可以拥有状态，这些状态在`Widget`生命周期中是可以变的
         - `StatefulWidget`由两个类组成：
           1. 一个`StatefulWidget`类
           2. 一个`State`类：`StatefulWidget`类本身是不变的，但是`State`类中持有的状态在`widget`生命周期中可能会发生变化
      2. 无状态组件`StatelessWidget`：没有要管理的内部状态。它通过构建一系列其他小部件来更加具体地描述用户界面，从而描述用户界面的一部分。当我们的页面不依赖`Widget`对象本身中的配置信息以及BuildContext时，就可以用到无状态组件。

      ![flutter-statefulwidget生命周期.png](http://ucoon.gitee.io/myblogimg/flutter-statefulwidget%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.jpg)

      **createState()**：createState是`StatefulWidget`里创建State的方法，当要创建新的`StatefulWidget`的时候，会立即执行createState，而且只执行一次；但是它在`StatefulWidget`的生命周期中可能会被多次调用。例如，当一个`StatefulWidget`同时插入到widget树的多个位置时，Flutter framework就会调用该方法为每一个位置生成一个独立的State实例

      **initState()**：当`Widget`第一次插入到`Widget`树时会被调用，对于每一个State对象，Flutter framework只会调用一次该回调，所以，通常在该回调中做一些一次性的操作，如状态初始化，订阅子树的事件通知等。不能在该回调中调用`BuildContext.dependOnInheritedWidgetOfExactType`（该方法用于在Widget树上获取离当前widget最近的一个父级`InheritWidget`）,原因是在初始化完成后，`Widget树中的IheritWidget`也可能会发生变化，正确的做法应该在`build()`或`didChangeDependencies()`中调用它。

      **didChangeDependencies()**：当State对象的依赖发生变化时会被调用；例如：在之前`build()`中包含了一个`InheritedWidget`，然后在之后的`build()`中`InheritedWidget`发生了变化，那么此时`InheritedWidget`的子`widget`的`didChangeDependencies()`回调都会被调用。典型的场景是当系统语言Locale或者应用主题改变时，Flutter framework会通知widget调用此回调。
      
      **build()**：主要是用于构建Widget子树的，在如下场景会被调用：
      
      - 在调用`initState()`之后
      - 在调用`didUpdateWidget()`之后
      - 在调用`setState()`之后
      - 在调用`didChangeDependencies()`之后
      - 在State对象从树中一个位置移除后（会调用`deactivate`）又重新插入到树的其他位置之后
      
      **reassemble()**：此回调是专门为了开发调试而提供的，在热重载（`hot reload`）时会被调用，此回调在Release模式下永远不会被调用。
      
      **didUpdateWidget()**：在widget重新构建时，Flutter framework会调用`Widget.canUpdate`来检测Widget树中同一个位置的新旧节点，然后决定是否需要更新，如果`Widget.canUpdate`返回`true`则会调用此回调。（`Widget.canUpdate`会在新旧widget的key和runtimeType同时相等时返回true）
      
      **deactivate()**：当State对象从树中被移除时，会调用此回调。在一些场景下，Flutter framework会将State对象重新插入到树中，如包含此State对象的子树在树的一个位置移动到另一个位置时（可以通过GlobalKey来实现），如果移除后没有重新插入到树中则紧接着会调用`dispose`方法。
      
      **dispose()**：当State对象从树中被永久移除时调用；通常在此回调中释放资源。

2. App的生命周期

   通过`WidgetsBindingObserver`可以获取Flutter App的生命周期：

   ```dart
     ///生命周期变化时回调
     ///resumed:应用可见并可响应用户操作(处于前台)
     ///inactive:用户可见，但不可响应用户操作
     ///paused:已经暂停了，用户不可见、不可操作（处于后台）
     ///detached：应用被挂起
     @override
     void didChangeAppLifecycleState(AppLifecycleState state) {
       switch (state) {
         case AppLifecycleState.resumed:
           break;
         case AppLifecycleState.inactive:
           break;
         case AppLifecycleState.paused:
           break;
         case AppLifecycleState.detached:
           break;
       }
     }
   ```

   

### Flutter 三棵树：Widget、Element、RenderObject

1. 在Flutter中，`Widget`的功能是“描述一个UI元素的配置数据”；
2. Flutter中真正代表屏幕上显示元素的类是`Element`，也就是说`Widget`只是描述`Element`的配置数据；
3. Widget只是UI元素的一个配置数据，并且一个`Widget`可以对应多个`Element`，这个很好理解，根据同一份配置（`Widget`），可以创建多个实例（`Element`）。



