![Flutter知识体系](https://static001.geekbang.org/resource/image/99/64/9959006fe52706a123cc7fc596346064.jpg) ly0316924

### Flutter是怎么完成组件渲染

图像显示的基本原理：在计算机系统中，图像的显示想要CPU、GPU和显示器一起配合完成：CPU负责图像数据计算，GPU负责图像数据渲染，而显示器负责最终图像显示。CPU把计算好的、需要显示的内容交给GPU，由GPU完成渲染后放入帧缓冲区，随后视频控制器根据垂直同步信号（Vsync）以每秒60次的速度，从帧缓冲区读取帧数据交由显示器完成图像显示。

![Flutter绘制原理](https://static001.geekbang.org/resource/image/95/2a/95cb258c9103e05398f9c97a1113072a.png)

### Skia

Skia是一款用C++开发的、性能彪悍的2D图像绘制引擎。

### Flutter三层架构：`Framework`、`Engine`、`Embedder`

![flutter_system_overview](http://ucoon.tech/MyBlogImg/flutter_system_overview.png)

1. Framework

   Framework是一个用Dart实现的UI SDK，包括Material Design风格的Widget，CuperTino风格的Widget，文本/图片/按钮等基础的Widgets，`Rendering`渲染、`Animation`动画、`Painting`图形绘制、`Gestures`手势等。

2. Engine

   Engine引擎层使用C++实现，主要包括：Skia，Dart和Text，实现了Flutter的渲染引擎、文字排版、事件处理和Dart运行时等功能。Skia和Text为上层接口提供了调用底层渲染和排版的能力，Dart则为Flutter提供了运行时调用Dart和渲染引擎的能力。而Engine层的作用，则是将它们组合起来，从它们生成的数据中实现视图渲染。

3. Embedder

   Embedder是一个操作系统适配层，即把Flutter嵌入到各个平台上去，这里做的主要工作包括渲染Surface设置，线程设置，以及插件等。从这里可以看出，Flutter的平台相关层很低，平台只是提供一个画布，剩余的所有渲染相关的逻辑都在Flutter内部，这就使得它具有了很好的跨端一致性。

### Flutter界面渲染过程

页面中的各界面元素（`Widget`）以树的形式组织，即控件树。Flutter通过控件树中的每个控件创建不同类型的渲染对象（`Element`），组成渲染对象树。而渲染对象树在Flutter的展示过程分为四个阶段：布局、绘制、合成和渲染。

#### 布局

>深度优先遍历（Depth First Search）：从一个未访问的节点V开始，沿着一条路一直走到底，然后从这条路尽头的节点回退到上一个节点，再从另一条路开始走到底...，不断递归重复此过程，直到所有的节点都遍历完成。
>
>广度优先遍历（Breadth First Search）：从一个未访问的节点V开始，先遍历这个节点的相邻节点，再依次遍历每个相邻节点的的相邻节点

Flutter采用深度优先机制遍历渲染对象树，决定渲染对象树中各渲染对象在屏幕上的位置和尺寸。在布局过程中，渲染对象树中的每个渲染对象都会接收父对象的布局约束参数，决定自己的大小，然后父对象按照控件逻辑决定各个子对象的位置，完成布局过程。

![Flutter布局过程](https://static001.geekbang.org/resource/image/f9/00/f9e6bbf06231fbad54ed11ef291e8d00.png)

**布局边界（Relayout Boundary）**：可以在某些节点自动或者手动地设置布局边界，当边界内地任何对象发生重新布局时，不会影响边界外地对象，反之亦然。

#### 绘制

布局完成后，渲染对象树中的每个节点都有了明确的尺寸和位置。Flutter会把所有的渲染对象绘制到不同的图层上，与布局过程一样，绘制过程也是深度优先遍历，而且总是先绘制自身，再绘制子节点。

![Flutter绘制过程](https://static001.geekbang.org/resource/image/8c/b8/8c1d612990d9ada0508c5a41c9e4cab8.png)

**重绘边界（Repaint Boundary）**：在重绘边界内，Flutter会强制切换新的图层，这样就可以避免边界内外的互相影响，避免无关内容置于同一图层引起不必要的重绘。

#### 合成和渲染

Flutter的渲染树层级通常很多，直接交付给渲染引擎进行多图层渲染，可能会出现大量渲染内容的重复绘制，所以还需要先进行一次图层合成，即将所有的图层根据大小、层级、透明度等规则计算出最终的显示效果，将相同的图层归类合并，简化渲染树，提高渲染效率。合并完成后，Flutter会将几何图层数据交由Skia引擎加工成二维图像数据，最终交由GPU进行渲染，完成界面的展示。

### Flutter工程目录结构

![Flutter工程目录结构](https://static001.geekbang.org/resource/image/e7/fc/e7ecbd5c21895e396c14154b2f226dfc.png)

### Widget，构建Flutter界面的基石

`Widget`的功能是“描述一个UI元素的配置数据”，所以即使销毁重建也不影响真实的渲染树，Flutter会计算diff判断真正需要刷新的部分

Flutter将视图树的概念进行了扩展，把视图数据的组织和渲染抽象为三部分，即Widget、Element和RenderObject，其关系如下图所示：

![Widget、Element与RenderObject](https://static001.geekbang.org/resource/image/b4/c9/b4ae98fe5b4c9a7a784c916fd140bbc9.png)

#### Widget

- Flutter将Widget设计成不可变的，所以当视图渲染的配置信息发生变化时，Flutter会选择重建Widget树的方式进行数据更新，以数据驱动UI构建的方式简单高效。
- Widget本身并不涉及实际渲染位图，所以它只是一份轻量级的数据结构，重建的成本很低。
- 由于Widget的不可变性，可以以较低成本进行渲染节点复用，因此在一个真实的渲染树中可能存在不同的Widget对应同一个渲染节点的情况，这无疑又降低了重建UI的成本。

#### Element

Element是Widget的一个实例化对象，它承载了试图构建的上下文数据，是连接结构化的配置信息（Widget）到最终渲染（RenderObject）的桥梁。

Flutter渲染过程：

- 通过Widget树生成对应的Element树
- 创建相应的RenderObejct并关联到Element.renderObject属性上
- 最后，构建RenderObject树，完成最终的渲染。

### Widget中的State

- `StatefulWidget`应对有交互、需要动态变化视觉效果的场景

- `Statelesswidget`用于处理静态的、无状态的视图展示

  当你所要构建的用户界面不随任何状态信息变化而变化时，需要选择`StatelessWidget`，反之则选用`StatefulWidget`

>Widget是不可变的，更新则意味着销毁+重建。StatelessWidget是静态的，一旦创建则无需更新，而对于StatefulWidget来说，在State类中调用setState方法更新数据，会触发视图的销毁和重建，也将间接地触发其每个子Widget的销毁和重建

#### UI 编程范式

- 命令式(原生系统开发)：精确地告诉操作系统用何种方式去做事情
- 声明式：视图和数据分离

#### State生命周期



### Dart语法

#### 常量定义

- const适用于定义编译常量（字面量固定值）的场景
- final适用于定义运行时常量的场景

#### 函数

1. Dart所有类型都是对象类型，函数也是对象

2. Dart不支持重载，而是提供了可选命名参数和可选参数(可忽略参数)

   - 可选命名参数：给参数增加{}，以`paramName`:  `value`的方式指定调用参数

     ```dart
     void enable1Flags({bool bold, bool hidden}) => print("$bold, $hidden");
     ```

   - 可选(可忽略)参数：给参数增加[]，则意味着这些参数是可以忽略的

     ```dart
     void enable2Flags(bool bold, [bool hidden]) => print("$bold, $hidden");
     ```

3. 类的复用：继承父类、接口实现和“混入”（`Mixin`，关键字with）

4. 运算符

   - ?.
   - ??=：如果a为null，则给a赋值value
   - ??

   

