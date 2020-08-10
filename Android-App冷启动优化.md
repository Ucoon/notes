---
title: Android App冷启动优化
---

# 启动方式

## 1. 概念

- 冷启动

  当启动应用时，后台没有该应用的进程（常见如：进程被杀，首次启动等），这时系统会重新创建一个新的进程分配给该应用

- 暖启动

  当启动应用时，后台已有该应用的进程（常见如：按```back```键、```home```键，应用虽然会退出，但是该应用的进程依然会保留在后台，可进入任务列表查看），所以在已有进程的情况下，这种启动会从已有的进程来启动应用

- 热启动

  相比暖启动，热启动时应用做的工作更少，启动时间更短（常见如：用户使用返回键退出应用，然后马上又重新启动应用）

一般情况下，我们都会将暖启动和热启动合在一起统称为热启动。一般对App的启动优化指的是针对冷启动方式的优化，在官方文档《[App startup time](https://developer.android.com/topic/performance/vitals/launch-time)》中也建议做好冷启动优化，也能提高热启动方式的优化。

## 2. 区别

- 冷启动：系统没有该应用的进程，需要创建一个新的进程分配给应用，所以会先创建和初始化```Application```，再创建和初始化```MainActivity```（包括一系列的测量、布局、绘制），最后显示在界面上
- 热启动：从已有的进程启动该应用，不会创建和初始化```Application```，直接创建和初始化```MainActivity```（包括一系列的测量、布局、绘制），最后显示在界面上

## 3. 冷启动流程

应用在冷启动的时候，需要执行一下三个任务：

- 加载和启动应用
- 应用启动之后会立即展示出一个空白的Window
- 创建App的进程

在创建完App进程之后，App进程会立即执行以下任务：

1. 创建App对象

2. 启动Main Thread

3. 创建启动的Activity对象

4. 加载View

5. 布局

6. 进行第一次绘制

   而一旦App进程完成了第一次绘制，系统进程就会用```main activity```替换已经展示的```background window```，此时用户就可以开始使用App了

   ![](https://developer.android.com/topic/performance/images/cold-launch.png)

# 启动时间

在官方文档中描述到：

1. 冷启动在5秒或者更长
2. 暖启动在2秒或者更长
3. 热启动在1.5秒或者更长

超过以上描述时间，即需要进行启动优化。那么对于启动时间该如何去检测？

## Time to initial display

在Android 4.4(API 19)以上，```logcats```包含了一个名为```Displayed```的log信息，这个值表示从启动过程到完成在屏幕上绘制```main activity```所用的时间。

![Displayed_Time](D:\MyDocument\Picture\Displayed_Time.jpg)

## 使用命令行方式

```shell
adb shell am start -S -R 10 -W com.fanhuan/.ui.SplashActivity
```

其中```-S```表示每次启动前先强行停止，```-R```表示重复测试次数

每一次的输出如下所示：

![adb_shell_time](D:\MyDocument\Picture\adb_shell_time.jpg)

其中```TotalTime```代表当前Activity启动时间，将多次```TotalTime```加起来求平均即可得到这个Activity的启动时间

# 优化方案

## 基础（官方提供）

### 1.将启动页主题背景设置成闪屏页图片

按照官方文档说明：使用Activity的```windowBackground```主题属性来为启动页提供一个简单的drawable，这么做的目的是为了消除启动时的黑白页，给用户一种秒响应的感觉，但是并不会真正减少启动时间，仅属于视觉优化

### 2. Application和主Activity的onCreate中异步初始化某些代码

因为在主线程上进行资源初始化会降低启动速度，所以可以将不必要的资源初始化延迟，达到优化的效果。

### 3. 主页面布局优化

1. 通过减少冗余或者嵌套布局来降低视图层次结构
2. 用```ViewStub ```替代在启动过程中不需要显示的UI控件

## 进阶



参考文档：

[App startup time](https://developer.android.com/topic/performance/vitals/launch-time)