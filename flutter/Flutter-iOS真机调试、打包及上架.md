---
title: Flutter iOS真机调试、打包及上架
---

# 1. 在Mac上搭建Flutter环境

这部分只需参考[官网](https://flutterchina.club/setup-macos/)搭建即可

**注意事项**

- 多个Path环境变量设置：

```shell
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
FLUTTER_HOME=/Users/yardi_wuhan/Library/FlutterSDK
PATH=$JAVA_HOME/bin:$FLUTTER_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME PATH CLASSPATH
```

- 如果你使用的是zsh，终端启动时 `~/.bash_profile` 将不会被加载，解决办法就是修改 `~/.zshrc` ，在其中添加：source ~/.bash_profile

# 2. Ruby、CocoaPods安装

>**CocoaPods** 就是iOS 项目的开发 第三方库的管理工具。
>
>`CocoaPods` 是用 `ruby` 实现的,要想使用它首先需要有`ruby`环境。 虽然 `Mac` 系统默认可以运行`ruby`。但是`ruby`版本过低是无法正常支持`CocoaPods`的使用

1. [更换Ruby源](https://gems.ruby-china.com/)

   ```shell
   # 查看现有的源
   gem source -l 
   # 移除
   gem sources --remove  https://rubygems.org/
   # 添加 ruby-china 的源
   gem sources -a https://gems.ruby-china.org/ 
   ```

2. 安装CocoaPods：`sudo gem install cocoapods`

3. 切换[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/CocoaPods/)安装CocoaPods

   ```shell
   # 移除原仓库镜像
   pod repo remove master
   # 使用清华源安装到本地 cd  ~/.cocoapods/repos/master
   git clone https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git 
   # 设置一下
   pod setup
   # 查看仓库信息
   pod repo 
   ```

4. 安装完成之后进入到项目的iOS目录下，执行`pod install `

# 3. 证书相关申请

1. 登录[apple开发者平台](https://developer.apple.com/account/resources/identifiers/list)

2. Register a new identifier（常见app的功能如第三方应用登录（Associated Domains）、推送（Push Notifications）可提前勾选Capabilities）

   ```dart
   注意事项：
   1. Bundle ID的区别
      - Explicit App ID「明确的 App ID」，一般格式是：com.company.appName；这种 id 只能用在一个app上，每一个新应用都要创建并只有一个。
      - Wildcard App ID「通配符 App ID」， 一般格式是：com.domainname.* ；这种 id 可以用在多个应用上，虽然方便，但是使用这种id的应用不能使用通知功能，所以不常用。
   ```

3. 在mac上通过钥匙串应用创建两个证书请求文件（CSR文件），分别对应Development环境和Distribution环境

   <img src="http://ucoon.tech/MyBlogImg/csr文件申请_1.png" style="zoom:25%;" />

   

   <img src="http://ucoon.tech/MyBlogImg/csr文件申请_2.png" style="zoom:25%;" />

4. Create a New Certificate(证书)

   1. 分别申请开发证书（iOS Development）和分发证书（iOS Distribution），开发证书用于开发和调试应用程序，可用于真机调试；生产证书用于打包上传App Store或者蒲公英，用于验证开发者身份。如果项目集成了推送功能，还需配置推送证书，推送证书同样也分两种：开发环境和生产环境（Apple Push Notification service SSL ：Sandbox & Production)，同时生成的p12文件需上传到服务端后台（如阿里云移动推送后台）
   
   2. 生成后的证书需要下载并通过钥匙串导入到Mac
   
      <img src="http://ucoon.tech/MyBlogImg/证书导入.png" alt="证书导入" style="zoom: 25%;" />
   
      ```dart
      注意事项：
          打开钥匙串应用，在点击登录-->我的证书页面下，双击证书导入即可
      ```
   
5. Register a New Provisioning Profile(描述文件)

   可参考[iOS 证书配置](https://zhuanlan.zhihu.com/p/69162456)

# 4. Xcode中的配置

Xcode中的配置

1. 点击`TARGETS`中的`Runner`，选中`Build Settings`栏，在`Code Signing Identity`中配置已下载导入的iOS证书

2. 选中`Signing & Capabilities`栏，`Provisioning Profile`选择下载相应的描述文件

   ```dart
   注意事项：Automatically manage signing不选中
   ```

3. 到此即可连接iOS真机在Android Studio中运行

   ```dart
   注意事项：需要将真机的UDID添加到apple开发者平台的设备列表(Devices)
   ```

# 5. 打包及上架

1. 运行命令：`flutter build ipa --release`
2. 在 Xcode 中打开 `build/ios/archive/MyApp.xcarchive`
3. 点击 `Distribute App` 按钮，选择分发的方式：App Store Connect（用于App Store上架）、Ad Hoc（用于蒲公英的分发平台）

参考文章：

[在macOS上搭建Flutter开发环境](https://flutterchina.club/setup-macos/)

[CocoaPods 换源 git 安装 与 使用](https://juejin.cn/post/6844903827297009677)

[App Bundle ID 基本信息介绍](https://zhuanlan.zhihu.com/p/60854366)

[iOS 证书配置](https://zhuanlan.zhihu.com/p/69162456)

[构建和发布为 iOS 应用](https://flutter.cn/docs/deployment/ios)

[一步快速获取 iOS 设备的 UDID](https://www.pgyer.com/tools/udid)



