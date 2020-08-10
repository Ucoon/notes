---
title: Android UI渲染机制
---

**UI 渲染的背景知识**

1. 屏幕与适配

   ```java
   px(pixel)：像素，1px代表物理屏幕上的一个像素点，不同分辨率设备上的1px显示的大小不一致
   dp/dip(density-independent pixel)：密度独立像素——一个基于屏幕物理密度的抽象单元，是Android引入的一个概念；这些单位相对于160 dpi(每英寸的点)屏幕，1dp大约等于1px。dp与px的比值随屏幕密度(dpi)而变化，但不一定是正比。使用dp单位是Android推荐的一个屏幕适配方案，它为您在不同设备上的UI像素的真实大小提供了一致性
   ppi(pixel per inch)：像素密度，单位英寸所包含的像素数量，该值越高，屏幕越细腻
   dpi(dots per inch)：表示某张图片单位英寸的点数，单位英寸内点数越多，则该图像越细腻，越清晰，是衡量图片质量的重要标准
   density：屏幕密度，在Android中是用来完成dp与px的转换
   关系：
   density = dpi/160; px = dp*(dpi/160) density = px/dp
   ```

2. CPU与GPU

   CPU(Central Processing Unit，中央处理器)是计算机设备核心器件，用于执行程序代码

   GPU(Graphics Processing Unit，图形处理器)主要用于处理图形运算，通常所说“显卡”的核心部件就是GPU。

   下面是CPU与GPU的结构对比图，其中：

   Control：控制器，用于协调控制整个CPU的运行，包括取出指令，控制其他模块的运行等；

   ALU(Arithmetic Logic Unit)：算术逻辑单元，用于进行数学，逻辑运算；

   Cache和DRAM分别为缓存和RAM，用于存储信息。

   <img src="https://awps-assets.meituan.net/mit-x/blog-images-bundle-2017/0fddd0b5.png" style="zoom:70%;" />

从结构图可看出，CPU的控制器