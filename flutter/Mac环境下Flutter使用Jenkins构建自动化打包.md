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
3. 首次启动需要解锁Jenkins，安装推荐的插件，自定义设置用户名及密码

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



# 插件推荐

## 切换为中文

插件：`Locale plugin`，`Localization: Chinese (Simplified)版本`

使用：在Manage Jenkins->Configure System下的Local设置默认语言(Default Language)：`zh_CN`