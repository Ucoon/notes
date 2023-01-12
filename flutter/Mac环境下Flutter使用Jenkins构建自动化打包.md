---
title: Mac环境下Flutter使用Jenkins构建自动化打包
---

前提：在产品迭代过程中，开发人员需要频繁的提供安装包给测试人员，这不仅占用了大量的开发时间，还影响了工作效率和积极性，所以我们急需解放双手，这时候Jenkins自动化打包的优越性就体现了出来。

# 准备工作

## 搭建Flutter开发环境

在Mac上需要搭建Flutter开发环境，这部分不再赘述，可参考[Flutter iOS真机调试、打包及上架](http://ucoon.tech/2022/01/27/Flutter-iOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%95%E3%80%81%E6%89%93%E5%8C%85%E5%8F%8A%E4%B8%8A%E6%9E%B6/)

## 安装Jenkins

1. 使用`brew`进行安装：`brew install jenkins`，
2. 启动jenkins：`brew services start jenkins`
3. 在浏览器中输入地址 http://0.0.0.0:8080 ，即可看到 Jenkins 页面
4. 首次启动需要解锁Jenkins，安装推荐的插件，自定义设置用户名及密码

安装完成Jenkins面板界面如下：

![jenkins_dashboard](http://ucoon.tech/MyBlogImg/jenkins_dashboard.png)

# 环境配置

## workspaceDir修改

自定义工作目录`workspaceDir`路径，打开安装目录下的`config.xml`：`open $HOME/.jenkins/config.xml`，修改`workspaceDir`的值

![workspaceDir.png](http://ucoon.tech/MyBlogImg/workspaceDir.png)

## 配置JDK

在Manage Jenkins->Global Tool Configuration下的JDK设置`JAVA_HOME`

![JDK配置](http://ucoon.tech/MyBlogImg/JDK.png)

## 全局环境变量配置

在Manage Jenkins->Configure System下的全局属性设置`ANDROID_HOME`、`PUB_HOSTED_URL`、`FLUTTER_STORAGE_BASE_URL`、`FLUTTER_HOME`、`PATH`

![Environment_variables](http://ucoon.tech/MyBlogImg/Environment_variables.png)

属性对应值分别如下：

```xml
<globalNodeProperties>
    <hudson.slaves.EnvironmentVariablesNodeProperty>
      <envVars serialization="custom">
        <unserializable-parents/>
        <tree-map>
          <default>
            <comparator class="java.lang.String$CaseInsensitiveComparator"/>
          </default>
          <int>5</int>
          <string>ANDROID_HOME</string>
          <string>/Users/yardi_wuhan/Library/Android/sdk</string>
          <string>FLUTTER_HOME</string>
          <string>/Users/yardi_wuhan/fvm/default</string>
          <string>FLUTTER_STORAGE_BASE_URL</string>
          <string>https://storage.flutter-io.cn</string>
          <string>PATH</string>
          <string>$PATH:$FLUTTER_HOME/bin:$HOME/.pub-cache/bin</string>
          <string>PUB_HOSTED_URL</string>
          <string>https://pub.flutter-io.cn</string>
        </tree-map>
      </envVars>
    </hudson.slaves.EnvironmentVariablesNodeProperty>
  </globalNodeProperties>
```

# 自动化打包

## Android

1. 新建一个Project，并选择`Freestyle project`

   ![create_jenkins_project](http://ucoon.tech/MyBlogImg/create_jenkins_project.png)

2. 填写描述

3. 源码管理——配置git，填写仓库地址`Repository URL`，并添加凭据`Credentials`

   如果你使用的是 https，那么需要配置认证，我这里使用的是 ssh，所以不需要配置认证，认证的方式需要添加凭据，参考如下所示，

   在`Private Key`下填写私钥值(`.ssh`目录下的`id_ed25519`)，对应git账号应上传公钥值(`.ssh`目录下的`id_ed25519.pub`)

   ![add_ssh_credentials](http://ucoon.tech/MyBlogImg/add_ssh_credentials.png)

4. 配置参数化构建过程

   可增加构建环境ENV参数等，示例如下

   ![build_parameter](http://ucoon.tech/MyBlogImg/build_parameter.png)

5. 构建打包脚本

   在构建脚本中可获取上述传入的构建参数值(`${BUILD_ENV}`)

   ![build_step_shell](http://ucoon.tech/MyBlogImg/build_step_shell.png)

6. 归档成品(**artifacts**)

   ![archive_artifacts](http://ucoon.tech/MyBlogImg/archive_artifacts.png)

7. 配置完成后，可在构建页面开始构建，如下所示

   ![android_project](http://ucoon.tech/MyBlogImg/android_project.png)

8. 构建完成后可在首页下载构建完成后的安装包

## iOS

**[通过在打包机器上配置证书和mobile provision等文件的方式来完成打包认证]**

1. 手动配置证书

2. 配置描述文件

3. 开始打包

   新建Project—>填写描述—>配置git—>配置参数化构建过程这几个步骤和Android项目类似，不同的是iOS的构建脚本，如下所示：

   ```shell
   cd app_common
   flutter clean
   flutter pub get
   cd ../patient
   flutter clean
   flutter pub get
   mv sample.env .env
   security unlock-keychain -p Yardi5550586
   flutter build ipa --release --dart-define=ENV=${BUILD_ENV}
   ExportOptionsPath=$JENKINS_HOME/jobs/$JOB_NAME/ExportOptions.plist
   ArchivePath=$WORKSPACE/patient/build/ios/archive/Runner.xcarchive
   PackagePath=$WORKSPACE/patient/build/ios/archive/build
   xcodebuild -exportArchive -exportOptionsPlist $ExportOptionsPath -archivePath $ArchivePath -exportPath $PackagePath -allowProvisioningUpdates
   ```

   关键命令解释：

   1. `security unlock-keychain -p xxxxxx`：在开始打包之前，需要先解锁下`keychain`，这里的xxxx就是对应Mac上的密码
   2. `flutter build ipa --release --dart-define=ENV=${BUILD_ENV}`：指定release模式，开始编译iOS代码，并在 `build/ios/archive` 文件夹下生成一个 Xcode 构建归档（`.xcarchive` 文档）
   3. 执行完`Archive`之后，就可以进入`export`阶段，`exportArchive`之前需要先准备一个`ExportOptions.plist`文件到指定目录，这个文件可以先在打包机上用Xcode执行一次完整的Export流程，在对应的archive文件夹下存在对应的`ExportOptions.plist`
   4. 接着通过指定命令`exportArchive`，指定`ExportOptions.plist`，最终输出到`PackagePath`，得到一个ipa文件

   上述命令均为`ad-hoc`模式，如果是指定`app-store`模式，则需准备`app-store`对应的`ExportOptions.plist`文件，最终才能得到一个`app-store`模式的ipa文件

# 插件推荐

## 切换为中文

插件：`Locale plugin`，`Localization: Chinese (Simplified)版本`

使用：在Manage Jenkins->Configure System下的Local设置默认语言(Default Language)：`zh_CN`

## 蒲公英上传和二维码显示

插件：`Upload to pgyer`和`description setter plugin`

使用：在构建完成后可以将apk/ipa文件上传至蒲公英，并以二维码形式展示在构建历史处

注意：Jenkins默认是plain text模式，所以不会对蒲公英上传成功后返回的html信息进行解析，所以装完`description setter plugin`后还需在全局安全设置(Config Global Security)中，将标记格式器(Markup Formatter)的设置更改为Safe HTML即可

```html
<a href="${appBuildURL}"><img src="${appQRCodeURL}" width="118" height="118"/></a>
```

![jenkins_upload_pgy](http://ucoon.tech/MyBlogImg/jenkins_upload_pgy.png)

参考文章：

[Flutter 搭建 iOS 命令行服务打包发布全保姆式流程](https://www.agora.io/cn/community/blog/21605)