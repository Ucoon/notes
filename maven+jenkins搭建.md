Tips:

1. so文件(```libmeetingsdk.so```)上传至maven：

   a.以jar包形式 ：文件大小：3348kb，需指定包路径，编译的apk正常

   b.以aar包形式：文件大小：432kb，不需指定包路径，编译的apk正常

2. modelVersion

3. 命令行上传文件至maven

   ```powershell
   mvn deploy:deploy-file -DgroupId=tech.somo.meeting -DartifactId=libmeetingsdk -Dversion=%VERSION% -DgeneratePom=false -Dpackaging=so -DrepositoryId=%REPOSITORYID% -Durl=http://192.168.1.55:10001/repository/%REPOSITORYID% -Dfile=%BASE%\workspace\Android\somo_video_reich_sdk\reichsource\depends\meetingsdk\lib\android\armeabi-v7a\libmeetingsdk.so
   ```

   ```java
   cd android\meetingsdk && gradlew clean uploadArchives -BUILD_TYPE=%BUILD_TYPE% -BUILD_VERSION=%BUILD_VERSION%
   ```






## Jenkins Tips

1. 账号相关：

   1. 账号注册页面：```http://192.168.1.55:8080/login?from=%2F```

      todo：账号安全策略待定（目前只要是登录用户有权限任何操作）

2. Android目录下：

   1. somo_video_app_android：

      目前支持如下参数动态配置：

      1. BUILD_TYPE：构建版本
      2. BRANCH：编译分支（该参数将传递至somo_video_android_somokit）
      3. SOMOSDK_VERSION：somosdk aar版本（该参数将传递至somo_video_android_somokit）
      4.  MEETINGSDK_VERSION：libmeetingsdk so版本（该参数将传递至somo_video_android_somokit，(确保maven上有对应版本的文件）

      编译somo_video_app_android之前会先编译somo_video_android_somokit，并将编译生成的aar上传至maven仓库，app项目中引用该aar，生成的apk默认上传至蒲公英

      **注：目前项目中maven的参数配置只提交在release_3.6.0_jenkins分支上**

   2. somo_video_android_somokit：

      目前支持如下参数动态配置：

      1. BRANCH：编译分支
      2. SOMOSDK_VERSION：somosdk aar版本
      3. MEETINGSDK_VERSION：libmeetingsdk so版本(确保maven上有对应版本的文件)

      somo_video_android_somokit编译生成后的aar将上传至maven仓库

      **注：目前项目中maven的参数配置只提交在release_3.6.0_jenkins分支上**

   3. meetingsdk：

      目前支持如下参数动态配置：

      1. BRANCH：编译分支
      2.  BUILD_VERSION：libmeetingsdk so版本

      meetingsdk编译生成后的aar将上传至maven仓库

   4. somo_video_sdk_android：待剥离去除，目前作用为更新其他so文件