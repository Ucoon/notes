---
title:flutter-tips
---

# 入门

## 环境安装

1. 使用镜像
2. 获取Flutter SDK
3. 环境变量配置
4. flutter doctor
5. 编辑器配置（AS、VS）：安装Flutter和Dart插件

## hello flutter 

2. 该应用程序继承了 StatelessWidget，这将会使应用本身也成为一个widget。 在Flutter中，大多数东西都是widget，包括对齐(alignment)、填充(padding)和布局(layout)

3. Flutter在构建页面时，会调用组件的`build`方法，widget的主要工作是提供一个build()方法来描述如何构建UI界面（通常是通过组合、拼装其它基础widget）

4. `setState`：通知Flutter框架，有状态发生了改变，Flutter框架收到通知后，会执行`build`方法来根据新的状态重新构建界面

   1. 在调用`initState()`之后。
   2. 在调用`didUpdateWidget()`之后。
   3. 在调用`setState()`之后。
   4. 在调用`didChangeDependencies()`之后。
   5. 在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其它位置之后。



# 杂记

## 项目结构

   ```dart
   ┬
   └ projectname
     ┬
     ├ android      - Android部分的工程文件
     ├ build        - 项目的构建输出目录
     ├ ios          - iOS部分的工程文件
     ├ lib          - 项目中的Dart源文件
       ┬
       └ src        - 包含其他源文件
       └ main.dart  - 自动生成的项目入口文件，类似RN的index.js文件
     ├ test         - 测试相关文件
     └ pubspec.yaml - 项目依赖配置文件类似于RN的 package.json 
   ```

## 各类组件

- 基础组件：`Text`、`IconButton`、`Icon`、`Icons`(https://design.google.com/icons/)、`Text`、`TextStyle`、`FontWeight`、`Form`、`TextFormField`、`SizedBox`、`RaisedButton`、`FlatButton`
- 布局类组件：`Scaffold`、`AppBar`、`BottomNavigationBar`、`Center`、`Row`、`Column`、`Expanded`
- 可滚动组件：`List`、`ListView`、`EdgeInsets`、`ListTile`、`CustomScrollView`
- 容器类组件：`Padding`、`EdgeInsets`、`SizedBox`、`LimitedBox`
- 功能型组件：`MaterialPageRoute`、`GestureDetector`

## 生命周期

![](https://pcdn.flutterchina.club/imgs/3-2.jpg)

## StatelessWidget VS StatefulWidget

https://juejin.im/post/6844903941025562632#heading-8

有状态组件`StatefulWidget`：有需要管理的内部状态，使用setState来管理状态改变。调用setState通知Flutter框架某个状态发生了变化，Flutter会重新运行build方法，应用程序变可以显示最新的状态。

- `StatefulWidget`可以拥有状态，这些状态在widget生命周期中是可以变的，而`StatelessWidget`是不可变的
- `StatefulWidget`只是又两个类组成：
  1. 一个`StatefulWidget`类
  2. 一个`State`类；`StatefulWidget`类本身是不变的，但是`State`类中持有的状态在`widget`**生命周期**中可能会发生变化

无状态组件`StatelessWidget`：没有要管理的内部状态。它通过构建一系列其他小部件来更加具体地描述用户界面，从而描述用户界面的一部分。当我们的页面不依赖Widget对象本身中的配置信息以及BuildContext时，就可以用到无状态组件。例如当我们只需要显示一段文字时。实际上Icon、Divider、Dialog、Text等都是StatelessWidget的子类。

**总结**

- 优先使用StatelessWidget
- 含有大量子Widget（如根布局、次根布局）最好使用StatelessWidget
- StatefulWidget最好用在子节点，同时尽量减少它的子节点

## 构造函数

https://blog.csdn.net/win7583362/article/details/100664528

## 路由管理

### `Route`和`Navigator`

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

### `MaterialPageRoute`类

`MaterialPageRoute`继承自`PageRoute`类，`PageRoute`类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性。`MaterialPageRoute` 是Material组件库提供的组件，它可以针对不同平台，实现与平台页面切换动画风格一致的路由切换动画：

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

### 基本路由

### 命名路由

1. 注册路由表：新建routes.dart文件
2. 配置main.dart
3. 配置路由传参
4. 配置路由守卫
5. 路由生成钩子

## 包管理

`pubspec.yaml`：管理第三方依赖包

1. 在`dependencies`字段下添加依赖，如：` english_words: ^3.1.5`
2. 安装：`flutter packages get`
3. 导入，如：`import 'package:english_words/english_words.dart';`

## 资源管理

## Widget状态(```State```)管理：

### 概念

- 短时状态（用户界面(UI)状态或者局部状态）：可以完全包含在一个独立widget的状态
- 应用状态（共享状态）：如果你想在你的应用中多个部分之间共享一个非短时的状态，并在用户会话期间保留这个状态，我们称之为应用状态

### 方式

1. **setState**：适用较小规模widget的暂时性状态的基础管理办法：
   
   - Widget管理自己的状态
   - Widget管理子Widget的状态
   - 混合管理（父Widget和子Widget都管理状态）
   
2. **Provider**：推荐的管理方式

    Flutter 在 widget 中存在一种机制，能够为其子孙节点提供数据和服务。（换言之，不仅仅是它的子节点，所有在它下层的 widget 都可以），首先理解三个概念：

   - ```ChangeNotifier```
   
     是 Flutter SDK 中的一个简单的类。它用于向监听器发送通知。换言之，如果被定义为 `ChangeNotifier`，你可以订阅它的状态变化。（这和大家所熟悉的观察者模式相类似）。
     
   - `ChangeNotifierProvider`
   
     `ChangeNotifierProvider` widget 可以向其子孙节点暴露一个 `ChangeNotifier` 实例。它属于 `provider` package。将其放在需要访问它的 widget 之上
   
     ```dart
     void main() {
       runApp(
         ChangeNotifierProvider(
           create: (context) => CartModel(),
           child: MyApp(),
         ),
       );
     }
     ```
   
   - ```Consumer```
   
   - `Provider.of`

## Http请求-Dio package

## Widget隐藏与显示

```dart
import 'package:flutter/widgets/dart';
import 'package:meta/meta.dart';

enum VisibilityFlag {
  visible,
  invisible,
  offscreen,
  gone,
}

class Visibility extends StatelessWidget {
  final Visibility visibility;
  final Widget child;
  final Widget removeChild;

  Visibility({
    @retuired this.child,
    @required this.visibility,
  }) : this.removeChild = Container();

  @override
  Widget build(BuildContext context) {
    if(visibility == VisibilityFlag.visible) {
      return child;
    }else if(visibility == VisibilityFlag.invisible) {
      return new IgnorePointer(
        ignoring: true,
        child: new Opacity(
          opcity: 0.0,
          child: child
        )
      );
    }else if(visibility == VisibilityFlag.offscreen) {
      return new Offstage(
        offstage: true,
        child: child
      );
    }else{
      return removeChild;
    }
  }
}
```

总结：

1. 对于visible: 什么也不做
2. 对于Invisible: 用IgnorePointer 和Opacity widget包裹，并将opacity的值设置为0
3. 对于offscreen：用Offstage widget包裹使得widget在屏幕外显示
4. 直接返回没有大小初始值container widget，可以根据需要自行更改另外的widget

## flutter dio json解析

针对复杂json解析会造成卡顿问题，dio给出的方案是使用`compute`方法在后台去解析json

```dart
// 必须是顶层函数
_parseAndDecode(String response) {
  return jsonDecode(response);
}

parseJson(String text) {
  return compute(_parseAndDecode, text);
}

void main() {
  ...
  // 自定义 jsonDecodeCallback
  (dio.transformer as DefaultTransformer).jsonDecodeCallback = parseJson;
  runApp(MyApp());
}
```

- Json数据自动生成实体类方式推荐

1. 使用网页自动生成： https://app.quicktype.io/

2. 使用 json_serializable（官方推荐：https://github.com/google/json_serializable.dart/tree/master/example）



参考：https://juejin.im/post/6854573217995030541

## flutter国际化插件Flutter Intl

https://juejin.im/post/6844903823119482888#heading-0

## flutter插件化开发  EventChannel && MethodChannel

参考：https://cloud.tencent.com/developer/article/1568736

>MethodChannel用通俗的语言来描述它的作用就是，当你想在flutter端调用native功能的时候，可以用它
>
>EventChannell用通俗的语言来描述就是，当native想通知flutter层一些消息的时候，可以用它

## iOS真机调试和运行

1. 在新的mac上搭建Flutter环境（所需软件：Android Studio、Xcode）

2. 安装ruby cocoapods（注意切换源）

3. cd project/ios -->pod install 

4. 在app开发者平台上创建id-->钥匙链创建私钥-->申请证书-->证书导入到mac-->创建profile-->在xcode导入profile-->修改build settings

5. 阿里云推送

6. 微信登录：

   ​	https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/iOS.html

   ​	https://www.jianshu.com/p/06108091db86

   

# 遇到的问题

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

5. Flutter 设备连接一直显示loading...

   连接网络！连接网络！连接网络！
   
6. ```dart
   A failure occurred while executing com.android.build.gradle.internal.tasks.Workers$ActionFacade
   File 'com.android.builder.files.ZipCentralDirectory@69765e30' was deleted, but previous version not found in cache
   ```

   >flutter channel stable 
   >
   >flutter upgrade --force 
   >
   >flutter pub cache repair 
   >
   >cd <YOUR APP FOLDER> flutter clean

# 常用flutter命令

1. `flutter create --org com.packagename projectname`：指定包名创建项目
2. `flutter config --enable-web`：开启web支持
3. `flutter run -d chrome`：部署web应用
4. `flutter create .`：向现有应用添加 Web 支持
5. `flutter packages get`或`pub get`：获取pubspec.yaml文件中列出的所需依赖
6. `flutter packages upgrade`或`pub upgrade`获取pubspec.yaml文件中列出的所有依赖的最新版本
7. `flutter clean`：清除缓存

# 常用第三方库

```yaml
  cupertino_icons: ^0.1.3
  dio: ^3.0.1
  dio_cookie_manager: ^1.0.0
  cookie_jar: ^1.0.0
  flutter_webview_plugin: 0.3.0+2
  crypto: ^2.0.6
  permission_handler: ^5.0.1+1
  # 本地json对象存储
  shared_preferences: ^0.5.3+5
  path_provider: ^1.6.11
  # State
  provider: 4.0.3
  #UI
  oktoast: ^2.3.1
  badges: ^1.1.0
  #设备信息
  device_info: ^0.4.2+6
  #uuid
  uuid: ^2.2.0
  #package info
  package_info: ^0.4.1  
```

# 参考

https://flutterchina.club/setup-windows/

https://book.flutterchina.club/chapter1/dart.html

https://flutter.cn/docs/development/data-and-backend/state-mgmt/options