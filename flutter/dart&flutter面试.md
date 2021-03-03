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

   ```dart
   为什么要将build方法放在State中，而不是放在StatefulWidget中？
   1. 状态访问不便。
   2. 继承StatefulWidget不便
   ```

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

   

### Flutter三层架构：`Framework`、`Engine`、`Embedder`

![flutter_system_overview](http://ucoon.gitee.io/myblogimg/flutter_system_overview.png)

1. Framework

   Framework使用dart实现，包括Material Design风格的Widget，CuperTino风格的Widget，文本/图片/按钮等基础的Widgets，`Rendering`渲染、`Animation`动画、`Painting`图形绘制、`Gestures`手势等。

2. Engine

   Engine引擎层使用C++实现，主要包括：Skia，Dart和Text。Skia是开源的二维图形库，提供了适用于多种软硬件平台的通用API

3. Embedder

   Embedder是一个嵌入层，即把Flutter嵌入到各个平台上去，这里做的主要工作包括渲染Surface设置，线程设置，以及插件等。从这里可以看出，Flutter的平台相关层很低，平台只是提供一个画布，剩余的所有渲染相关的逻辑都在Flutter内部，这就使得它具有了很好的跨端一致性。

### Flutter 三棵树：Widget树、Element树、RenderObject树

#### 依赖关系

当应用启动时Flutter会遍历并创建所有的Widget会形成Widget Tree，同时与Widget Tree相对应，通过调用Widget上的`createElement()`创建每个Element对象，形成Element Tree，最后调用Element的`createRenderObject()`创建每个渲染对象，形成一个Render Tree（“渲染树”），总结一下，我们可以认为Fluuter的UI系统包含三棵树：Widget树，Element树、渲染树。他们的依赖关系是：Element树根据Widget树生成，而渲染树又依赖于Element树。

![依赖关系](https://upload-images.jianshu.io/upload_images/24924109-0d3c0dc4a2b7798b.png?imageMogr2/auto-orient/strip|imageView2/2/w/644/format/webp)

#### Widget

1. 在Flutter中，`Widget`的功能是“描述一个UI元素的配置数据”；

   ```dart
   /// Widgets are the central class hierarchy in the Flutter framework. A widget
   /// is an immutable description of part of a user interface. Widgets can be
   /// inflated into elements, which manage the underlying render tree.
   在Flutter中，Widget的功能是“描述一个UI元素的配置数据”，所以它是不可变的，变的是Widget里面的状态，也就是State。
   ```

2. Flutter中真正代表屏幕上显示元素的类是`Element`，也就是说`Widget`只是描述`Element`的配置数据；

3. Widget只是UI元素的一个配置数据，并且一个`Widget`可以对应多个`Element`，这个很好理解，根据同一份配置（`Widget`），可以创建多个实例（`Element`）。

```dart
@immutable
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });
  final Key key;

  @protected
  Element createElement();

  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

- Widget类继承自`DiagnosticableTree`，`DiagnosticableTree`即“诊断树”，主要作用是提供调试信息
- `key`：这个`key`的主要作用是决定是否在下一次Build时复用旧的widget，决定的条件在`canUpdate()`中
- `canUpdate()`：只要`newWidget`与`oldWidget`的`runtime`和`key`同时相等时就会用`newWidget`去更新`Element`对象的配置，否则就会创建新的`Element`

#### Element

Element是Widget在UI树具体位置的一个实例化对象，大多数Element只有唯一的`renderObject`，但是还有一些Element会有多个子节点，如继承自`RenderObjectElement`的一些类，比如`MultiChildRenderObjectElement`。

Element的生命周期如下：

1. Framework调用`Widget.createElement`创建一个Element实例，记为`element`
2. Framework调用`element.mount(parent, newSlot)`，mount()中首先调用`element`所对应Widget的`createRenderObject`方法创建与`element`相关联的RenderObject对象，然后调用`element.attachRenderObject`方法将`element.renderObject`添加到渲染树中插槽指定的位置（这一步不是必须的，一般发生在Element树结构发生变化时才需要重新attach）。插入到渲染树后的`element`就处于“active”状态，处于“active”状态后就可以显示在屏幕上了（可以隐藏）。
3. 当有父Widget的配置数据改变时，同时其`State.build`返回的Widget结构与之前不同，此时就需要重新构建对应的Element树，为了进行Element复用，在Element重新构建前会先尝试是否可以复用旧树上相同位置的element，element节点在更新前都会调用其对应Widget的`canUpdate`方法，如果返回`true`，则复用旧Element，旧的Element会使用新Widget配置数据更新，反之则会创建一个新的Element。`Widget.canUpdate`主要是判断`newWidget`与`oldWidget`的`runtimeType`和`key`是否同时相等，如果同时相等就返回`true`，否则就会返回`false`。根据这个原理，当我们需要强制更新一个Widget时，可以通过指定不同的Key来避免复用
4. 当有父级Element决定要移除`element`时（如Widget树结构发生了变化，导致`element`对应的Widget被移除），这时父级Element就会调用`deactivateChild`方法来移除它，移除后`element.renderObject`也会被渲染树中移除，然后Framework会调用`element.deactivate`方法，这时`element`状态会变成“inactive”状态。
5. “inactive”态的element将不会再显示到屏幕。为了避免在一次动画执行过程中反复创建、移除某个特定element，“inactive”态的element在当前动画最后一帧结束前都会保留，如果在动画执行结束后它还未能重新变成“active”状态，Framework就会调用其`unmount`方法将其彻底移除，这时element的状态为`defunct`，它将永远不会再被插入到树中。
6. 如果`element`要重新插入到Element树的其他位置，如`element`或`element`的父级拥有一个GlobalKey(用于全局复用元素），那么Framework会先将element从现有位置移除，然后再调用其`activate`方法，并将其`renderObject`重新attach到渲染树。

![element_生命周期](http://ucoon.gitee.io/myblogimg/element_%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.jpg)

#### RenderObject

`RenderObject`的主要职责是布局（Layout）和绘制，所有的`RenderObject`会组成一棵渲染树`Render Tree`。`RenderObject`是渲染树中的一个对象，它拥有一个`parent`和一个`parentData`插槽（`slot`），所谓插槽就是指预留的一个接口或位置，这个接口和位置是由其他对象来接入或占据的，这个接口或位置在软件中通常用预留变量来表示，而`parentData`正是一个预留变量，它正是由`parent`来赋值。`parent`通常会通过子`RenderObject`的`parentData`存储一些和子元素相关的数据。

```dart
constraints; //从父级传递给它的约束
parentData;//其父对象附加的有用的信息
performLayout();//计算此渲染对象的布局
paint();//绘制该组件及其子组件
```

##### 布局过程

1. 从顶部向下传递约束**Constraints**

   ```dart
   void layout(Constraints constraints, { bool parentUsesSize = false }) {
      ...
      RenderObject relayoutBoundary; 
       if (!parentUsesSize || sizedByParent || constraints.isTight 
           || parent is! RenderObject) {
         relayoutBoundary = this;
       } else {
         final RenderObject parent = this.parent;
         relayoutBoundary = parent._relayoutBoundary;
       }
       ...
       if (sizedByParent) {
           performResize();
       }
       performLayout();
       ...
   }
   ```

   `layout()`需传入两个参数，第一个为`constraints`，即父节点对子节点大小的限制，该值根据父节点的布局逻辑确定；另一个参数是`parentUsesSize`，该值用于确定`relayoutBoundary`，该参数表示子节点布局变化是否影响父节点，如果为true，当子节点布局发生变化时，父节点都会标记为需要重新布局，如果为false，则子节点布局发生变化后不会影响父节点。

2. 从底部向上传递布局信息

   这一过程用来传递具体的布局信息。子节点接受到来自父节点的约束后，会依据它产生自己具体的布局信息，如父节点规定我的最小宽度是 500 的单位像素，子节点按照这个规则可能定义自己的宽度为 500 个像素，或者大于 500 像素的任何一个值。这样，确定好自己的布局信息之后，将这些信息告诉父节点。父节点也会继续此操作向上传递一直到最顶部。

##### 绘制过程

`RenderObject`可以通过`paint()`方法来完成具体的绘制逻辑，流程和布局流程相似，子类可以实现`paint()`方法来完成自身的绘制逻辑，`paint()`签名如下：

```dart
void paint(PaintingContext context, Offset offset) { }
```

通过context.canvas可以获取到`Canvas`对象，接下来就可以调用`Canvas`API来实现具体的绘制逻辑。如果节点有子节点，它除了完成自身绘制逻辑之外，还要调用子节点的绘制方法

### Flutter路由跳转、开源框架(Fluro)及页面切换监视

### Flutter页面数据刷新Provider

### 插件设计

### dio网络请求封装









