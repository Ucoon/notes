---
title: Android屏幕刷新机制
---

# 显示系统基础知识

在一个典型的显示系统中，一般包括CPU、GPU、Display三个部分，CPU负责计算帧数据，把计算好的数据交给GPU，GPU会对数据进行渲染，渲染好后放到buffer（图像缓冲区）存起来，然后Display（屏幕或显示器）负责把buffer的数据呈现到屏幕上。

# 基础概念

- 屏幕刷新频率

  一秒内屏幕刷新的次数（一秒内显示了多少帧的图像），单位是Hz；刷新频率取决于硬件的固定参数

- 逐行扫描

  显示器并不是一次性将画面显示到屏幕上，而是从左到右，从上到下逐行扫描，顺序显示整屏的一个个像素点。

- 帧率（Frame Rate）

  表示GPU在一秒内绘制操作的帧数，单位是fps。Android系统采用60fps，即每秒钟GPU最多绘制60帧画面。

- 画面撕裂（tearing）

  一个屏幕内的数据来自2个不同的帧，画面会出现撕裂感。

# 双缓存

## 原因

屏幕刷新频率是固定的，比如每16.6ms从buffer取数据显示完一帧，理想情况下帧率和刷新频率保持一致，即每绘制完成一帧，显示器显示一帧，但是CPU/GPU写数据是不可控的，所以会出现buffer里有些数据根本没显示出来就被重写了，即buffer里的数据可能是来自不同的帧。当屏幕刷新时，此时它并不知道buffer的状态，因此从buffer抓取的帧并不是完整的一帧画面，即出现画面撕裂。

## 解决：双缓存

让绘制和显示拥有各自的buffer：GPU始终将完成的一帧图像数据写入到**Back Buffer**，而显示器使用**Frame Buffer**，当屏幕刷新时，Frame Buffer并不会发生变化，当Back buffer准备就绪后，它们才进行交换。

## 交换时机：VSync

时机：等到屏幕处理完一帧数据后，才进行交换。

当扫描完一个屏幕后，设备需要重新回到第一行以进入下一次的循环，此时有一段时间空隙，称为Vertical Blanking Interval(VBI)。这个时间点就是我们进行缓冲区交换的最佳时间。因为此时屏幕没有在刷新，也就避免了交换过程中出现screen tearing的状况。

**VSyc**（垂直同步）是Vertical Synchronization的简写，它利用VBI时期出现的vertical sync pulse（垂直同步脉冲）来保证双缓冲在最佳时间点才进行交换。另外，交换是指各自的内存地址。

# Android屏幕刷新机制

## Android 4.1之前：双缓存 + VSync

![](https://img-blog.csdnimg.cn/20200819205135422.png#pic_center)

1. Display显示第0帧数据，此时CPU和GPU渲染第1帧画面，且在Display显示下一帧前完成
2. 因为渲染及时，Display在第0帧显示完成后，也就是第一个VSync后，缓存进行交换，然后正常显示第1帧
3. 接着CPU第2帧开始处理，直到第2个VSYnc快来前GPU才开始处理
4. 第2个VSync来时，由于第2帧数据还没准备就绪，缓存没有交换，显示的还是第1帧，这种情况Android开发组命名为“Jank”，即发生了丢帧
5. 当第2帧数据准备完成后，它并不会马上被显示，而是要等待下一个VSync进行缓存交换再显示

所以总的来说，就是屏幕平白无故地多显示了一次第1帧。

**双缓存的交换是在VSync到来时进行，交换后屏幕会取Frame buffer内的新数据，而实际此时的Back buffer就可以供GPU准备下一帧数据了，如果VSync到来时CPU/GPU就开始操作的话，是有完整的16.6ms的，这样就可以基本避免jank的出现了**

## drawing with VSync——Project Butter（黄油工程）

![](https://img-blog.csdnimg.cn/20200819212951194.png#pic_center)

系统在收到VSync pulse后，将马上开始下一帧的渲染。即**一旦收到VSync通知（16ms触发一次），CPU和GPU才立刻开始计算然后把数据写入buffer**

CPU/GPU根据VSync信号同步处理数据，可以让CPU/GPU有完整的16ms时间来处理数据，减少了jank，一句话总结：**VSync同步使得CPU/GPU充分利用了16.6ms时间，减少jank。**

问题：假如界面比较复杂，CPU/GPU的处理时间较长，超过了16.6ms，结果会如下图所示：

![](https://img-blog.csdnimg.cn/2020081921343523.png#pic_center)

1. 在第2个时间段内，却因GPU还在处理B帧，缓存没能交换，导致A帧被重复显示
2. 而B完成后，又因缺乏VSync pulse信号，它只能等待下一个signal的来临，于是在这一过程中，有一大段时间是被浪费的
3. 当下一个VSync出现时，CPU/GPU马上执行操作（A帧），且缓存交换，相应的显示屏对应的就是B，这时看起来是正常的，只不过由于执行时间仍然超过16ms，导致下一次应该执行的缓冲区交换又被推迟了——如此反复循环，便出现了越来越多的“Jank”

## 双缓存机制总结

**当VSync出现时，CPU/GPU开始处理数据并写入到Back Buffer，并与显示屏进行缓存交换，显示屏取得是Frame Buffer的数据；当下一个VSync出现时，CPU/GPU重复上述操作，如此循环反复。**

## 三缓存

**三缓存**就是在双缓存机制基础上增加了一个 Graphic Buffer缓冲区，这样可以最大限度的利用空闲时间，带来的坏处就是多使用的一个Graphic Buffer所占用的内存

![](https://img-blog.csdnimg.cn/20200819220105382.png#pic_center)

1. 第1个Jank是不可避免的，但是在第二个16ms时间段，CPU/GPU使用**第三个Buffer**完成C帧的计算，虽然还是会多显示一次A帧，但是后续显示就比较顺畅了，有效避免Jank的进一步加剧
2. 注意在第3段中，A帧的计算已经完成，但是在第4个vsync来的时候才显示，如果是双缓存，那在第三个vsync就可以显示了。

**三缓存有效利用了等待vsync的时间，减少了jank，但是带来了延迟。**所以，是不是 Buffer 越多越好呢？这个是否定的，Buffer 正常还是两个，当出现 Jank 后三个足以。

延申1：View和SufaceView的区别？

View：显示视图，内置画布，提供图形绘制函数、触屏事件、按键事件函数等；必须在UI主线程内更新画面，速度较慢；

SufaceView：基于view视图进行拓展的视图类，更适合2D游戏的开发；是view的子类，使用双缓机制，保证了UI界面的流畅性，同时在SufaceView新的线程中更新画面，所以刷新界面速度比view快，例：Camera预览界面使用SufaceView。

1. View底层没有双缓冲机制，而SufaceView有
2. View主要适用于主动更新，而SufaceView适用于被动的更新，如频繁的刷新
3. View会在主线程去更新UI，而SufaceView则在子线程中刷新
4. SufaceView的内容不在应用窗口上，所以不能使用变换（平移、缩放、旋转等）

延申2：SufaceView和TextureView的区别

SufaceView和TextureView均继承于android.view.View，与其他view不同的是，两者都能在独立的线程中绘制和渲染，在专用的GPU线程中大大提高渲染的性能

1. SufaceView专门提供了嵌入视图层级的绘制界面，开发者可以控制该界面像Size等的形式，能保证界面在屏幕上的正确位置，但也有局限：
   1. 由于是独立的一层view，更像是独立的一个window，不能加上动画、平移、缩放；
   2. 两个SufaceView不能相互覆盖
2. TextureView是一个一般的view，像TextView那样能被缩放、平移，也能加上动画；TextureView只能在开启了硬件加速的Window中使用，并且消费的内存要比SufaceView多，并伴随着1~3帧的延迟