---
title: Flutter 脚手架
---

解决的痛点：

- 多个项目的百花齐放，同个功能技术的多次实现
- 标准不统一，后期维护成本高，研发人员陷入固定项目
- 需要制定统一项目开发标准，提供基础能力，提高开发效率

我理解的脚手架：

- 能够快速帮业务同学生成新项目的目录模板
- 能够提升业务同学的开发效率和开发的舒适性

内置集成功能：

- 移动端通用UI组件库
- 移动端基础库
- ~~路由：Navigator 1.0，Navigator 2.0~~
- ~~国际化：flutter_Intl~~
- 主题切换
- 事件总线：EventBus??
- ~~存储管理：shared_preferences，本地数据库~~
- ~~数据状态管理： provider、riverPod、getX~~
- ~~网络：dio~~
- ~~屏幕适配：flutter_screenutil~~
- 常用的第三方SDK库：~~google登录~~、~~apple登录~~
- ~~应用内升级：kooboo_flutter_app_upgrade: ^0.0.4~~

项目结构：

```yaml
android/ 		# 安卓工程
ios/     		# ios工程
lib/
  |- components/ 	# 共用widget组件封装
  |- config/ 		# 全局的配置参数
  |- constants/ 	# 常量文件夹
  |- event_bus/ 	# 事件总线
  |- provider/ 		# 全局状态管理
  |- pages/ 		# 页面ui层，每个独立完整的页面，每个页面可独立放自己的provider状态管理
      |- AppHomePage/ 	# APP主体页面
      |- SplashPage/ 	# APP闪屏页
  |- service/ 		# 请求接口抽离层
  |- routes/ 		# 定义路由相关文件夹
  |- utils/ 		# 公共方法抽离
    |- dio/ 		# dio底层请求封装
  |- main.dart 		# 入口文件
pubspec.yaml 		# 配置文件

```



遇到的问题：

1. 为什么重复造轮子：

   基础框架如果直接用第三方插件，不方便以后修改和扩展
