1. 环境安装

   1. 使用镜像
   2. 获取Flutter SDK
   3. 环境变量配置
   4. flutter doctor

2. 编辑器配置（AS、VS）

   安装Flutter和Dart插件

3. hello_flutter 

   1. 分析应用结构

   2. 该应用程序继承了 StatelessWidget，这将会使应用本身也成为一个widget。 在Flutter中，大多数东西都是widget，包括对齐(alignment)、填充(padding)和布局(layout)

   3. Flutter在构建页面时，会调用组件的`build`方法，widget的主要工作是提供一个build()方法来描述如何构建UI界面（通常是通过组合、拼装其它基础widget）

   4. 有状态组件`StatefulWidget`和无状态组件`StatelessWidget`

      - `StatefulWidget`可以拥有状态，这些状态在widget生命周期中是可以变的，而`StatelessWidget`是不可变的
      - `StatefulWidget`只是又两个类组成：
        1. 一个`StatefulWidget`类
        2. 一个`State`类；`StatefulWidget`类本身是不变的，但是`State`类中持有的状态在`widget`**生命周期**中可能会发生变化

   5. `setState`：通知Flutter框架，有状态发生了改变，Flutter框架收到通知后，会执行`build`方法来根据新的状态重新构建界面

      1. 在调用`initState()`之后。
      2. 在调用`didUpdateWidget()`之后。
      3. 在调用`setState()`之后。
      4. 在调用`didChangeDependencies()`之后。
      5. 在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其它位置之后。

   6. `Scaffold`是Material库中提供的页面脚手架，它提供了默认的导航栏、标题和包含主屏幕widget树（后同“组件树”或“部件树”）的`body`属性，组件树可以很复杂。

   7. 生命周期：

      ![](https://pcdn.flutterchina.club/imgs/3-2.jpg)

4. 编写第一个Flutter应用：scaffold_demo

   1. 组件：

      ```dart
   基础组件：Text、IconButton、Icon、Icons(https://design.google.com/icons/)、Text、TextStyle、FontWeight
      布局类组件：Scaffold、AppBar、BottomNavigationBar
   可滚动组件：List、ListView、EdgeInsets、ListTile
      布局类组件：Center
      功能型组件：MaterialPageRoute、GestureDetector
      ```
   
   2. 路由管理
   
      核心概念：**`Route`和`Navigator`**
   
      - 路由（`Route`）在移动开发中通常指页面（`Page`），比如在`Android`中通常指一个`Activity`，所谓路由管理，就是管理页面之间如何跳转，通常也可被称导航管理
   
      - `Navigator`是一个路由管理的组件，它提供了打开和退出路由页方法。`Navigator`通过一个栈来管理活动路由集合。通常当前屏幕显示的页面就是栈顶的路由。
   
        ```dart
        1. Future push(BuildContext context, Route route)
            将给定的路由入栈（即打开新的页面），返回值是一个Future对象，用以接收新路由出栈（即关闭）时的返回数据
        2. bool pop(BuildContext context, [ result ])
            将栈顶路由出栈，result为页面关闭时返回给上一个页面的数据
        3. Future pushNamed(BuildContext context, String routeName,{Object arguments})
            通过路由名称来打开新路由
        实例方法：
            Navigator.push(BuildContext context, Route route)等同于Navigator.of(context).push(Route route)
        ```
   
      `MaterialPageRoute`类
   
      `MaterialPageRoute`继承自`PageRoute`类，`PageRoute`类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性。`MaterialPageRoute` 是Material组件库提供的组件，它可以针对不同平台，实现与平台页面切换动画风格一致的路由切换动画
   
      ```dart
        MaterialPageRoute({
          WidgetBuilder builder,
          RouteSettings settings,
          bool maintainState = true,
          bool fullscreenDialog = false,
        })
      ```
   
      - `builder` 是一个WidgetBuilder类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个widget。我们通常要实现此回调，返回新路由的实例。
      - `settings` 包含路由的配置信息，如路由名称、是否初始路由（首页）。
      - `maintainState`：默认情况下，当入栈一个新路由时，原来的路由仍然会被保存在内存中，如果想在路由没用的时候释放其所占用的所有资源，可以设置`maintainState`为false。
      - `fullscreenDialog`表示新的路由页面是否是一个全屏的模态对话框，在iOS中，如果`fullscreenDialog`为`true`，新页面将会从屏幕底部滑入（而不是水平方向）。
   
      基本路由
   
      命名路由
   
      1. 注册路由表：新建routes.dart文件
      2. 配置main.dart
      3. 配置路由传参
      4. 配置路由守卫
   
      路由生成钩子
   
   3. 包管理
   
      `pubspec.yaml`：管理第三方依赖包
   
      1. 在`dependencies`字段下添加依赖，如：` english_words: ^3.1.5`
      2. 安装：`flutter packages get`
      3. 导入，如：`import 'package:english_words/english_words.dart';`
   
   4. 资源管理
   
5. 状态管理：

   - Widget管理自己的状态
   - Widget管理子Widget的状态
   - 混合管理（父Widget和子Widget都管理状态）

6. Dart 语言简介

   ```dart
   1.var;//关键词，可以接收任何类型的变量，Dart中var变量一旦赋值，类型便会确定，则不能再改变其类型(Dart是一个强类型语言，区别js)
   2.dynamic;//关键词，可以接收任何类型的变量，并且可以在后期改变赋值类型
   3.Object;//Dart所有对象的根基类(包括Function和Null)
   4.final和const;//常量，const变量是一个编译时常量，final变量在第一次使用时被初始化
   5.函数
   6.异步支持，待实际编码过程中学习
   7.在Dart语言中使用下划线前缀标识符，会强制其变成私有的，如：_suggestions
   8.语法 "i ~/ 2" 表示i除以2，但返回值是整形（向下取整）
   ```

**遇到的问题**：

1. flutter running gradle task 'assembledebug' 卡住：

   1. 下载gradle和kotlin依赖，网络问题，可采取离线方式

   2. 设置阿里云镜像：

      ```groovy
      maven{ url 'https://maven.aliyun.com/repository/google' }
      maven{ url 'https://maven.aliyun.com/repository/jcenter' }
      maven{url 'http://maven.aliyun.com/nexus/content/groups/public'}
      ```

      1. 在app/build.gradle加上阿里云镜像
      2. 在fluttersdk下的packages/flutter_tools/gradle/flutter.gradle加上阿里云镜像

2. Flutter android GradleException显示红色报错解决办法：

   ```throw new GradleException(...) ```替换为```throw new Exception(...) ```
   
3. VSCode 创建项目为默认包名（```com.example.projectName```）和默认app名称，如何修改：

   https://stackoverflow.com/questions/51534616/how-to-change-package-name-in-flutter

   ```java
   1. android/app/src/main/AndroidManifest.xml中修改package="xxx.xxx.xxx"
   2. android/app/src/main/build.gradle中修改applicationId "xxx.xxx.xxx"
   3. android/app/src/main/.../MainActivity.java对应的包路径
   4. 修改文件绝对路径
   ```

4. 由于VSCode 创建项目为默认包名，如何解决：

   通过命令行指定包名创建项目：

   ```powershell
   flutter create --org com.packagename projectname
   ```

   

参考：

https://flutterchina.club/setup-windows/

https://book.flutterchina.club/chapter1/dart.html