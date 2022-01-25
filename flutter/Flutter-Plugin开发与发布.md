---
title: Flutter Plugin开发与发布
---

以开发腾讯云基础版人脸核身Flutter插件为例，总结Flutter Plugin开发和发布流程。

项目地址：https://github.com/Ucoon/wb_cloud_face

# 1. 创建 package

- 方式一：通过`Android Studio`直接创建`Flutter Project`（选择Plugin类型）

  可选择使用的语言和插件支持的平台

  <img src="C:\MyProject\MyBlogImg\Flutter_Plugin_1.png" style="zoom:60%;" />

- 方式二：通过命令行创建Flutter Plugin Project（使用`--template=plugin`）

  ```dart
  flutter create --org tech.ucoon --template=plugin wb_cloud_face
  ```

  可以使用-i为iOS指定开发语言，使用-a为Android指定开发语言，如：

  ```dart
  flutter create --org tech.ucoon --template=plugin -i swift -a kotlin wb_cloud_face
  ```

# 2. 实现包package

1. 在lib下的dart文件中定义接口：`openCloudFaceService`

   ```dart
     static Future<WbCloudFaceVerifyResult> openCloudFaceService({
       required WbCloudFaceParams params,
     }) async {
       final res = await _channel.invokeMethod(
           'openCloudFaceService', params.toJson());
       return WbCloudFaceVerifyResult.fromJson(json.decode(res));
     }
   ```

2. 添加Android平台实现的代码(android目录下)

   注意事项：

   - `import io.flutter.embedding.engine.plugins.FlutterPlugin;`导入plugin相关代码报红

     参考[stack overflow的回答](https://stackoverflow.com/questions/62172420/flutter-not-found-when-developing-plugin-for-android)

     ```dart
     You should open the project in android studio from the example/android location.
     ```

   - 插件中引入第三方本地aar文件时，需在插件`/android/build.gradle`中添加`flatDir`

     ```dart
     rootProject.allprojects {
         repositories {
             ...
             flatDir {
                 dirs project(':wb_cloud_face').file('libs')
             }
         }
     }
     ```

3. 添加iOS平台实现的代码(iOS目录下)

   注意事项：

   - 在podspec文件(`podspec是一个描述pod库版本文件`)添加说明：`name`，`version`、`summary`、`homepage`、`author`

   - 引用第三方静态库Framework：在iOS目录下新建Framework文件夹，把需要的三方库拷贝到Framework文件夹下，并在podspec文件中配置如下三个字段

     ```dart
     s.vendored_frameworks //第三方静态库的.framework文件路径
     s.vendored_libraries//第三方静态库的.a文件路径
     s.resource //第三方静态库的.bundle文件路径
     ```

   至此plugin开发完成

# 3. 发布Plugin

1. 在发布之前，检查`pubspec.yaml`(检查`description`、`version`、`homepage`字段)、`README.md`（使用教程）以及`CHANGELOG.md`（版本记录）文件，以确保其内容的完整性和正确性。
2. 然后, 运行 dry-run 命令以查看是否都准备OK了: `flutter packages pub publish --dry-run`
3. 最后, 运行发布命令: `flutter packages pub publish`

注意事项：

- 发布插件压缩之后的包必须小于100M

  ```dart
  Your package must be smaller than 100 MB after gzip compression. If it’s too large, consider splitting it into multiple packages, using a .pubignore file to remove unnecessary content, or cutting down on the number of included resources or examples.
  ```

  针对这种情况，可考虑搭建Flutter pub私有仓库，或者以github方式引用插件

参考资料：

[开发Packages和插件](https://flutterchina.club/developing-packages/#plugin)

[Flutter Plugin引用iOS三方静态库Framework](https://www.jianshu.com/p/9077fb85a074)

[腾讯云人脸核身SDK文档](https://cloud.tencent.com/document/product/1007/35866)

