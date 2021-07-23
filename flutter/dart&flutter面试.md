### Flutter是什么

>Flutter是一款移动应用程序SDK，一份代码可以同时生成iOS和Android两个高性能、高保真的应用程序。
>
>Flutter目标是使开发人员能够交付在不同平台上都感觉自然流畅的高性能应用程序。我们兼容滚动行为、排版、图标等方面的差异。

#### 跨平台自绘引擎（与RN，Weex的区别）

RN、Weex核心是通过JavaScript开发，执行时需要JavaScript解释器，UI是通过原生控件渲染。Flutter与用于构建移动应用程序的其他大多数框架不同，因为Flutter既不使用WebView，也不使用操作系统的原生控件。相反，Flutter使用自己的高性能渲染引擎来绘制widget。Flutter使用C、C++、Dart和Skia（2D渲染引擎）构建。

>目前，**Skia已然是Android官方的图像渲染引擎**了，因此**Flutter Android SDK无需内嵌Skia引擎**就可以获得天然的Skia支持；而对于iOS平台来说，由于Skia是跨平台的，因此**它作为Flutter 的iOS渲染引擎被嵌入到了Flutter iOS SDK中，代替了iOS闭源的Core Graphics/Core Animation/Core Text**，这也正是Flutter iOS SDK打包的APP包体积比Android要大一些的原因。

#### 高性能(为什么是Dart？)

1. Flutter App采用Dart语言开发，Dart在JIT（Just-in-time 即时编译）模式下，速度与JavaScript基本持平；而且Dart还支持AOT（Ahead-of-time 提前编译）模式，当以AOT模式运行时，JavaScript便远远追不上。
2. Flutter使用自己的渲染引擎来绘制UI，布局数据等由Dart语言直接控制，所以在布局过程中不需要像RN那样通过JavaScriptCore在JavaScript和原生之间进行通信，这在一些滑动和拖动的场景下具有明显优势。

### Flutter的生命周期

1. Widget的生命周期

   1. `Widget`是有状态还是无状态的，取决于他们依赖于状态的变化：
      1. 有状态：交互或者数据改变导致`Widget`改变，例如改变文案
      2. 无状态：不会被改变的`Widget`，例如纯展示界面

   ```dart
   /// Widgets are the central class hierarchy in the Flutter framework. A widget is an immutable description of part of a user interface. Widgets can be inflated into elements, which manage the underlying render tree.
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

当应用启动时Flutter会遍历并创建所有的Widget会形成Widget Tree，同时与Widget Tree相对应，通过调用Widget上的`createElement()`创建每个Element对象，形成Element Tree，最后调用Element的`createRenderObject()`创建每个渲染对象，形成一个Render Tree（“渲染树”），总结一下，我们可以认为Flutter的UI系统包含三棵树：Widget树，Element树、渲染树。他们的依赖关系是：Element树根据Widget树生成，而渲染树又依赖于Element树。

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

### Flutter BuildContext VS Android Context

#### Flutter BuildContext

`BuildContext`是抽象接口类

```dart
abstract class BuildContext {
    ...
}
```

官方的解释如下：

```dart
BuildContextobjects are actually Element objects. The BuildContextinterface is used to discourage direct manipulation of Element objects
```

即`BuildContext`就是widget对应的`Element`，所以我们可以通过`context`在`StatelessWidget`和`StatefulWidget`的`build`方法中直接访问`Element`对象。`BuildContext`接口用于阻止对`Element`对象的直接操作。

使用这个`Context`可以获取主题，入栈新路由等。

#### Android Context

>Context是一个关于应用环境的抽象类，它的实现由安卓系统提供。用于访问一些应用内资源，也可以调用系统服务开启Activity、Service、发送和接收广播等

### Flutter路由跳转、开源框架(Fluro)及页面切换监视

#### Route和Navigator

- 路由（`Route`）在移动开发中通常指页面（`Page`），比如在`Android`中通常指一个`Activity`，所谓路由管理，就是管理页面之间如何跳转，通常也可被称为导航管理。

- 导航管理会维护一个路由栈，路由入栈(push)操作对应打开一个新页面，路由出栈(pop)操作对应关闭页面操作；`Navigator`是一个路由管理的组件，它提供了打开和退出路由页的方法，`Navigator`通过一个栈来管理活动路由集合，通常当前屏幕显示的页面就是栈顶的路由

  ```dart
  Future push(context, route):将给定的路由入栈（即打开新的页面），返回值是一个Future对象，用以接收新路由出栈（即关闭）时的返回数据
  bool pop(context, [result]):将栈顶路由出栈，result为页面关闭时返回给上一个页面的数据
  ```

- 实例代码（入栈）：

  ```dart
  await Navigator.push(context, MaterialPageRoute(
      settings: routeSettings,
      fullscreenDialog: fullscreenDialog,
      maintainState: maintainState,
      builder:(context){}
  ));
  ```

  `MaterialPageRoute`继承自`PageRoute`类，`PageRoute`类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口以及属性：

  - `settings`包含路由的配置信息，如路由名称，是否是初始路由（首页）
  - `fullscreenDialog`表示新的路由页面是否是一个全屏的模态对话框，默认为false
  - `maintainState`：默认情况下，当入栈一个新路由时，原来的路由仍然会被保存在内存中，如果想在路由没用的时候释放其所占用的所有资源，可以设置`maintainState`为false
  - `builder`是一个`WidgetBuilder`类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个widget。

#### 命名路由

1. 注册路由表：新建routes.dart文件
2. 配置main.dart
3. 配置路由传参
4. 配置路由守卫：`NavigatorObserver `
5. 路由生成钩子

#### RouteAware

>A [Navigator] observer that notifies [RouteAware]s of changes to the state of their [Route]

```dart
abstract class RouteAware {
  /// Called when the top route has been popped off, and the current route
  /// shows up.
  void didPopNext() { }

  /// Called when the current route has been pushed.
  void didPush() { }

  /// Called when the current route has been popped off.
  void didPop() { }

  /// Called when a new route has been pushed, and the current route is no
  /// longer visible.
  void didPushNext() { }
}
```



### Flutter页面数据刷新(Provider mvvm)

状态管理的原则：如果状态是组件私有的，则应该由组件自己管理；如果状态要跨组件共享，则该状态应该由各个组件共同的父元素来管理。

三个概念：

- `Provider`

- `ChangeNotifier`
- `ChangeNotifierProvider`
- `ChangeNotifierProxyProvider`
- 其他类型的`Provider`
- `Consumer`
- `selector`

#### Provider

最基础的`provider`，它会获取一个值并将它暴露出来，但是该值改变时，并不会更新widget

`Provider.of<CartModel>(context, listen: false);`

#### ChangeNotifier

`ChangeNotifier`是Flutter SDK中的一个简单的类，它用于向监听器发送通知。换言之，如果被定义为`ChangeNotifier`，你可以订阅它的状态变化（类似观察者模式）。`notifyListeners()`：当模型发生改变并且需要更新UI的时候可以调用该方法。

#### ChangeNotifierProvider

1. `ChangeNotifierProvider` 可以向其子孙节点暴露一个`ChangeNotifier`实例。
2. `ChangeNotifierProvider`的位置：在需要访问它的widget之上。
3. `ChangeNotifierProvider`不会重复实例化`ChangeNotifier`实例，如果该实例已经不会再被调用，`ChangeNotifierProvider`也会自动调用`ChangeNotifier`实例的`dispose()`方法。
4. 如果需要提供更多状态，可以使用`MultiProvider`

#### ChangeNotifierProxyProvider

如果需要同时监听一个数据model的改变来自动更新UI，并且某个数据model的改变依赖于另一个model类，那么应该使用`ChangeNotifierProxyProvider`

#### 其他类型的provider

- `ListenableProvider`：用于暴露可监听的对象，该`provider`将会监听对象的改变以便及时更新组件状态
- `ValueListenableProvider`：监听一个可被监听的值，并只暴露`ValueListenable.value`方法
- `StreamProvider`：监听一个流，并暴露出其最近发送的值
- `FutureProvider`：接受一个`Future`作为参数，在这个`Future`完成时更新依赖

#### Consumer

`ChangeNotifier`实例已经通过`ChangeNotifierProvider`在应用中与widget相关联，通过`Consumer`即可开始调用它。

```dart
return Consumer<CartModel>(
    builder:(context, cart, child){
        return Text("Total price: ${cart.totalPrice}");
    }
);
```

1. 必须指定要访问的模型类型

2. `Consumer`唯一必须的参数就是builder。当`ChangeNotifier`发生变化的时候就会调用builder这个函数。

   builder参数解析

   1. cart：`ChangeNotifier`的实例，可通过该实例定义UI的内容
   2. `child`：用于优化目的

#### selector

```dart
class Selector<A, S> extends Selector0<S>{
    Selector({
        Key key,
        @required ValueWidgetBuilder<S> builder,
        @required S Function(BuildContext, A) selector,
        ShouldRebuild<S> shouldRebuild,
        Widget child,
    }) : assert(selector != null),
    	 super(
             key: key,
             shouldRebuild: shouldRebuild,
             builder: builder,
             selector: (context) => selector(context, Provider.of(context)),
             child: child,
         );
}
```

- `Selector`相当于`Consumer`，但是可以在某些值不变的情况下，防止rebuild

- `selector()`：`Selector`使用`Provider.of`获取共享的数据。数据作为`selector`方法的入参A，执行`selector`方法，返回build需要的数据S，返回的数据要尽可能少，能满足build就好

- `shouldRebuild`：默认判断前后两次S相等性，来决定是否rebuild。并且也提供了自定义的`shouldRebuild`方法来判断，参数是前后两次S

- S：selector的数据，必须是immutable(不可变)的，因此，selector通常返回集合或覆盖了“==”的类。如果需要selector多个值，推荐tuple

  >immutable：一个类的对象在通过构造方法创建后如果状态不会再被改变，那么它就是一个不可变(immutable)类，它的所有成员变量的赋值仅在构造方法中完成，不会提供任何setter方法供外部类去修改



#### MVVM状态管理



### 插件设计 EventChannel && MethodChannel

#### 平台通道

平台通道是Flutter和原生之间通信的桥梁，是Flutter插件的底层基础设施

#### EventChannel && MethodChannel

>MethodChannel用通俗的语言来描述它的作用是，当你想在flutter端调用native功能的时候，可以用它
>
>EventChannel用通俗的语言来描述它的作用是，当native想通知flutter层一些消息的时候，可以用它。



### dio网络请求封装

## Future、microtask执行顺序







