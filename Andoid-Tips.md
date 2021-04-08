# Activity 解析

1. 生命周期：

   ```java
   onCreate --> onstart --> onResume --> 活动运行中 --> 另一个Activity --> onPause --> onStop --> onDestroy
   ```

2. A --> B

   ```java
   A onCreate --> onstart --> onResume --> 活动运行中 --> 启动 B --> A onPause --> B onCreate --> onstart --> onResume --> A onStop
   ```

3. 返回键 --> A

   ```java
   B onPause --> A onRestart --> onStart --> onResume --> B onStop --> B onDestroy
   ```

   ActivityB是个**窗口Activity**的情况下，2、3的结论就不一样了：ActivityA跳转到ActivityB时，ActivityA失去焦点部分可见，故不会调用onStop

   ```java
   A-->B: A onPause-->B onCreate --> B onStart --> B onResume
   返回键-->A：B onPause--> A onResume--> B onStop --> B onDestroy
   ```

4. Fragment生命周期：

   ```java
   onAttach-->onCreate-->onCreateView-->onActivityCreated-->onStart-->onResume-->onPause-->onStop-->onDestroyView-->onDestroy-->onDeteach
   ```

5. 横竖屏切换

   ```
   onPause --> onSaveInstanceState --> onStop --> onDestroy --> onCreate --> onStart --> onRestoreInstanceState --> onResume
   ```

   指定属性```configChanges```避免横竖屏切换

6. 启动模式

   ```java
   standard(标准模式)、singleTop(栈顶复用)、singleTask(栈内复用)、singleInstance(单例模式)
   ```

# Service 解析

![service生命周期](http://ucoon.gitee.io/myblogimg/service生命周期.png)

# bindService 与 startService 区别

```java
1. startService：不进行通信，停止服务使用stopService
   bindService：进行通信，停止服务使用unbindService
2. 生命周期:
	startService：onCreate --> onStartCommand --> onDestroy
	bindService: onCreate --> onBind --> onUnBind --> onDestroy
```

# Service可以执行耗时操作吗

```java
service同样运行在UI线程，而耗时操作(如网络请求、文件操作)会阻塞UI线程，给用户不好的体验
如果需要在服务中进行耗时操作，可以选择IntentService，IntentService是Service的子类，用来处理异步请求；IntentService在onCreate()方法中通过HandlerThread单独开启一个线程来处理Intent请求对象所对应的任务，这样可以避免事务处理阻塞主线程。
```

# IntentService 解析

```java
1. 在onCreate中：
a. 通过HandlerThread单独开启一个名为IntentService的线程
HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
b. 创建一个名叫ServiceHandler的内部Handler，并与HandlerThread所对应的子线程进行绑定
mServiceLooper = thread.getLooper();
mServiceHandler = new ServiceHandler(mServiceLooper);
c 通过onStartCommand()传递给服务intent，依次插入到工作队列中，并逐个发送给onHandleIntent()
Message msg = mServiceHandler.obtainMessage();
msg.arg1 = startId;
msg.obj = intent;
mServiceHandler.sendMessage(msg);
d.通过onHandleIntent()来依次处理所有Intent请求对象所对应的任务
onHandleIntent((Intent)msg.obj);
stopSelf(msg.arg1);
```

# BroadcastReceiver 解析

```java
1.Android中的广播使用了设计模式中的观察者模式：基于消息的发布/订阅事件模型
2.默认情况下，广播接收器运行在UI线程，因此，onReceive方法不能执行耗时操作，否则将导致ANR。
3.注册方式：
	静态注册：在AndroidManifest.xml通过标签声明
	动态注册：通过调用Context.registerReceiver()
    	动态广播最好在Activity的onResume注册、onPause注销
    	对于动态广播，有注册就必然有注销，否则会导致内存泄露
```

![广播区别](http://ucoon.gitee.io/myblogimg/广播区别.png)

广播类型：

普通广播、系统广播、有序广播、App应用内广播（LocalBroadcastManager）

# ContentProvider 解析

```java
1.不仅常用于进程间通信，也适用于进程内通信
2.步骤使用：
	1.创建数据库类
	2.自定义ContentProvider类
	3.注册创建的ContentProvider类
	4.访问ContentProvider的数据
```

# Android 消息机制

```java
主要指Handler的运行机制，Handler的运行又与Message、MessageQueue、Looper紧密相关。简单的来说，当Handler发送消息时，将会调用MessageQueue.enqueueMessage，向消息队列（实质上是单链表）中添加消息，当通过Looper.loop开启循环后，会不断地从线程池中读取消息，即调用MessageQueue.next，然后调用目标Handler（即发送该消息的Handler）的dispatchMessage方法传递消息，然后返回到Handler所在线程，目标Handler收到消息，调用handleMessage方法，接收消息，处理消息。
```

![handler_消息机制](http://ucoon.gitee.io/myblogimg/handler_消息机制.jpg)

## Handler中有Looper死循环，为什么没有阻塞主线程，原理是什么

*应该从 主线程的消息循环机制 与Linux的循环异步等待作用讲起。最后将handler引起的内存泄漏，内存泄漏一定是一个加分项*

1. 为什么没有导致应用卡死

   **前提：**ActivityThread.java是主线程入口的类

   ```java
   public static final void main(String[] args) {
       ...
       //创建Looper和MessageQueue
       Looper.prepareMainLooper();
       ...
       //轮询器开始轮询
       Looper.loop();
       ...
   }
   ```

   Looper.loop()方法：

   ```java
   while (true) {
       //取出消息队列的消息，可能会阻塞
       Message msg = queue.next(); // might block
       ...
       //解析消息，分发消息
       msg.target.dispatchMessage(msg);
      	...
   }
   ```

   显而易见的，如果main方法没有looper进行循环，那么主线程一运行完毕就会退出。

   **总结：ActivityThread的main方法主要就是做消息循环，一旦退出消息循环，那么应用就会退出**

   **真正的原因：**因为Android的应用是由事件驱动的，`looper.loop()`不断地接收事件、处理事件，每一个点击触摸或者说Activity的生命周期都是运行在Looper.loop()的控制之下，如果它停止了，应用也就停止了。**也就是说我们的代码其实就是在这个循环去执行的，当然不会阻塞**。

   **真正会卡死主线程的操作**是在回调方法`onCreate/onStart/onResume`等操作时间过长，会导致掉帧，甚至发生ANR，`looper.loop`本身不会导致应用卡死。

2. 主线程的死循环一直运行是不是特别消耗CPU资源？原理是什么

   `Linux pipe/epoll`机制：简单说就是在主线程的`MessageQueue`没有消息时，便阻塞在loop的`queue.next()`中的`nativePollOnce()`方法里，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。

# Android中UI的刷新机制

# HandlerThread 解析

# 事件分发机制

   ```java
   1. 事件分发的对象：事件(MotionEvent)，当用户触摸屏幕时（View或ViewGroup派生的控件），将产生点击事件（Touch事件）
   2. 事件类型：
   MotionEvent.ACTION_DOWN、MotionEvent.ACTION_UP、MotionEvent.ACTION_MOVE、MotionEvent.ACTION_CANCEL
   3. 事件列：从手指触摸屏幕到手指离开屏幕，这个过程产生的一系列事件。任何事件列都是以DOWN事件开始，UP事件结束，中间有无数的MOVE事件。
   4. 事件分发的本质：所谓事件分发机制，其实就是对MotionEvent事件的分发过程，即当一个MotionEvent产生以后，系统需要将这个事件传递给一个具体的View，而这个传递的过程就是分发过程。
   5. 事件分发的流程：
   dispatchTouchEvent()：用于事件的分发
   onInterceptTouchEvent()：用于判断是否拦截当前事件【只存在与ViewGroup，普通的view没有此方法】
   onTouchEvent()：用来处理点击事件
   一般流程：一个点击事件产生后，传递的顺序是Activity（Window）–> ViewGroup –> View。
   具体流程：
   1.事件最先传递到Activity的dispatchTouchEvent()进行事件分发
   2.然后调用Window类实现类PhoneWindow的SuperDispatchTouchEvent()
   3.调用DecorView的SuperDispatchTouchEvent()
   4.调用ViewGroup的dispatchTouchEvent()，如果ViewGroup的dispatchTouchEvent()返回false，则表示事件停止传递，且将事件回传给Activity的onTouchEvent();
   5.调用ViewGroup的onInterceptTouchEvent()，如果ViewGroup的onInterceptTouchEvent()返回true，则表示它要拦截当前事件，事件就由ViewGroup自己处理，调用自身的onTouchEvent()；如果返回false，表示它不拦截当前事件，这时事件会传递到子元素，子元素的dispatchTouchEvent()会被调用，如此反复
   ```

![事件分发业务流程图](http://ucoon.gitee.io/myblogimg/事件分发业务流程图.jpg)

![Android事件分发方法](http://ucoon.gitee.io/myblogimg/Android事件分发方法.png)

思考：Android自定义View长按事件的实现

```java
public boolean dispatchTouchEvent(MotionEvent event) {
		int x = (int) event.getX();
		int y = (int) event.getY();
		
		switch(event.getAction()) {
		case MotionEvent.ACTION_DOWN:
			mLastMotionX = x;
			mLastMotionY = y;
			isMoved = false;
			postDelayed(mLongPressRunnable, ViewConfiguration.getLongPressTimeout());
			break;
		case MotionEvent.ACTION_MOVE:
			if(isMoved) break;
			if(Math.abs(mLastMotionX-x) > TOUCH_SLOP 
					|| Math.abs(mLastMotionY-y) > TOUCH_SLOP) {
				//移动超过阈值，则表示移动了
				isMoved = true;
				removeCallbacks(mLongPressRunnable);
			}
			break;
		case MotionEvent.ACTION_UP:
			//释放了
			removeCallbacks(mLongPressRunnable);
			break;
		}
		return true;
	}
```

# View的工作过程：

```java
主要指measure、layout和draw过程，View的绘制流程是从ViewRoot的performTraversals，它经过measure、layout和draw三个过程才能最终将一个view绘制出来，针对performTraversals的大致流程，如下图所示：
```

![performTraversals的工作流程](http://ucoon.gitee.io/myblogimg/performTraversals的工作流程.png)

```java
performTraversals会依次调用performMeasure、performLayout和performDraw这三个方法，这三个方法分别完成顶级View的measure、layout和draw这三大流程，其中performMeasure会调用measure方法（measure方法是一个final类型的方法），在measure方法中又会调用onMeasure方法，在onMeasure方法中则会对所有的子元素进行measure过程，这个时候measure流程就从父容器传递到子元素中了，完成了一次measure过程。接着子元素会重复父容器的measure过程，如此反复就完成了整个view树的遍历。同理，performLayout和performDraw的传递流程和performMeasure是类似的
```

1. Measure过程：MeasureSpec.Mode的三种测量模式：UNSPECIFIED、EXACTLY、AT_MOST
2. layout过程：
3. draw过程：

# Android 自定义view过程

```java
1. 构造函数-->onMeasure-->onSizeChanged-->onLayout-->onDraw-->提供接口(属性设置函数)
```

# Android中的动画

```java
 动画类型：View动画、帧动画、属性动画
 View动画通过对场景里的对象不断做图像变化（平移、缩放、旋转、渐变），从而产生动画效果，它是一种渐近式动画，并且View动画支持自定义；
 帧动画通过顺序播放一系列图像从而达到动画效果，可以理解为图片切换动画；
 属性动画通过动态地改变对象的属性从而达到动画效果，属性动画为API11的新特性。
 使用动画注意事项：
 1.内存溢出：这个问题主要出现在帧动画中，当图片数量较多且图片较大时就极易出现OOM
 2.内存泄漏：在属性动画中有一类无限循环的动画，这类动画需要在Activity退出时及时停止，否则将导致Activity无法释放从而导致内存泄漏；而View动画并不存在此问题
 3.兼容性问题：动画在3.0以下的系统上有兼容性问题
 4.View动画问题：View动画是对View的影像做动画，并不是真正的改变View的状态，因此有时候会出现动画完成后View无法隐藏的现象，即setVisibility(View.Gone)失效了，这个时候只要调用view.clearAnimation()清除View动画即可解决问题
 5.不要使用px：尽量使用dp
 6.动画元素的交互：将view移动后，在Android3.0以前，不管是View动画还是属性动画，新位置均无法触发单击事件，同时老位置仍可触发单击事件；在Android3.0以后，属性动画的单击事件触发位置为移动后的位置，但是View动画仍在老位置。
 7.硬件加速
```

# Android的进程间通信

```java
 1.使用Intent
 2.使用文件共享
 3.使用Binder: Messenger、AIDL
	服务端：服务端创建一个 Service 用来监听客户端的连接请求，然后创建一个 AIDL 文件，将暴露给客户端的接口在这个 AIDL 文件中声明，最后在 Service 中实现这个 AIDL 接口即可。
	客户端：绑定服务端的 Service ，绑定成功后，将服务端返回的 Binder 对象转成 AIDL 接口所属的类型，然后就可以调用 AIDL 中的方法了。
	注意：客户端调用远程服务的方法，被调用的方法运行在服务端的 Binder 线程池中，同时客户端的线程会被挂起，如果服务端方法执行比较耗时，就会导致客户端线程长时间阻塞，导致 ANR 。客户端的 onServiceConnected 和 onServiceDisconnected 方法都在 UI 线程中。
 4.使用ContentProvider
 5.使用socket
 	client: new Socket("127.0.0.1", 8081);->判断是否连接socket.isConnected();->通过输入流（socket.getInputStream）获取server发送过来的数据->通过输出流（socket.getOutputStream）发送数据
 	server: new ServerSocket(8081);->调用serverSocket.accept()->通过输出流（socket.getOutputStream）发送数据->通过输入流接收数据（socket.getInputStream）
```

![Binder工作机制](http://ucoon.gitee.io/myblogimg/Binder工作机制.jpg)

# Activity启动模式(launchMode)

```java
 standard、singleTop、singleTask、singleInstance
 ps：Activity的启动模式设置为singleTask或Intent设置了FLAG_ACTIVITY_NEW_TASK，在startActivityForResult之后，会立即被返回Activity.RESULT_CANCELED，再执行页面跳转
```

1. 引申：点击通知栏跳转到Activity B，点击返回如何跳转到MainActivity（应用已被KILL掉）

    ```java
    1. 使用 TaskStackBuilder：任务栈的创造者
    2. 直接使用PendingIntent.getActivities()
    ```

# Android App包瘦身优化实践

   1. 了解Apk的构成
      
       | 文件/目录           | 描述                                                         |
       | ------------------- | ------------------------------------------------------------ |
       | lib/                | 存放so文件，可能会有armeabi、armeabi-v7a、arm64-v8a、x86、x86_4、mips，大部分情况下只需要支持armabi与x86的架构即可，如果非必需，可以考虑拿掉x86的部分 |
       | res/                | 存放编译后的资源文件，例如：drawable、layout等               |
       | assets/             | 应用程序的资源，应用程序可以使用AssetManager来检索该资源     |
       | META-INF/           | 该文件夹一般存放于已经签名的APK中，它包含了APK中所有文件的签名摘要等信息 |
       | classes(n).dex      | classes文件是Java Class，被DEX编译后可供Dalvik/ART虚拟机所理解的文件格式 |
       | resources.arsc      | 编译后的二进制资源文件                                       |
       | AndroidManifest.xml | Android的清单文件，格式为AXML，用于描述应用程序的名称、版本、所需权限、注册的四大组件 |
       
       当然还会有一些其它的文件，例如上图中的```org/```、```src/```、```push_version```等文件或文件夹。这些资源是Java Resources
       
   2. 方式：
      
       1. 代码压缩：通过开```ProGuard```来实现代码压缩，可在build.gradle对应的构建类型中添加```minifyEnable true```；每次执行完```ProGuard```之后，```ProGuard```都会在```${project.buildDir}/outputs/mapping/${flavorDir}/```生成一下文件：
       
          | 文件        | 描述                                                         |
          | ----------- | ------------------------------------------------------------ |
          | dump.txt    | APK中所有类文件的内部结构                                    |
          | mapping.txt | 提供原始与混淆过的类、方法和字段名称之间的转换，可以通过`proguard.obfuscate.MappingReader`来解析 |
          | seeds.txt   | 列出未进行混淆的类和成员                                     |
          | usage.txt   | 列出从APK移除的代码                                          |
       
       2. R Field的优化
       
       3. 针对代码的其他瘦身方式：
       
          1. 减少ENUM的使用
          2. 通过pmd cpd来检查重复的代码从而进行代码优化
          3. 移除掉无用或功能重复的依赖库
       
       4. 图片优化：
       
          ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2017/25cf827c.png
       
       5. 开启资源压缩：```shrinkResources: true```
       
       6. 资源混淆

# 常见的xml解析方式：SAX解析、Pull解析和DOM解析

   1. Pull解析：基于事件的解析器，与SAX不同的是，在PULL解析过程中，我们需要自己获取产生的事件然后做相应的操作，而不像SAX那样由处理器触发一种事件的方法，执行我们的代码

   ```java
    优点：Pull解析器小巧轻便，解析速度快，简单易用
   ```

   2. SAX解析：基于事件的解析器，事件驱动的流式解析方式是，从文件的开始顺序解析到文档的结束，不可暂停或倒退
      
       工作原理：对文档进行顺序扫描，当扫描到文档（```document```）开始与结束、元素（```element```）开始与结束、文档（```document```）结束等地方时通知事件处理函数（回调），由事件处理函数做相应动作，然后继续同样的扫描，直至文档结束。

   ```java
    优点：解析速度快，占用内存小
    缺点：不会记录标签的关系，由程序自己处理，增加程序负担
   ```

   3. DOM解析：即对象文档模型，它是将整个XML文档载入内存，每个节点当作一个对象，结合代码分析。

   ```java
    优点：在内存中以树形结构存放，因此检索和更新效率会更高
    缺点：解析和加载整个文档耗资源
   ```

# 进程保活：

   ```java
    1. 监听全局的静态广播：时间更新、开机广播、网络状态、解锁加锁亮屏暗屏（3.1版本）
    2. 定时器、JobScheduler
    3. 双进程、双Service守护
    4. 提高Service优先级
    5. 前台通知
    6. 监听锁屏广播打开1像素Activity（基于onStartCommand() return START_STICKY）
   ```

# Android中的性能优化

   1. 布局优化：减少布局文件的层级

       如何进行布局优化：
       1. 删除布局中无用的控件和层次，其次有选择的使用性能较低的ViewGroup
       2. 采用include、merge、ViewStub标签
       3. 避免过度绘制

   2. 绘制优化：在View的onDraw里避免执行大量的操作：在onDraw里面不要创建新的局部变量；onDraw函数里不做耗时操作或者大量循环

   3. Android过度绘制优化

      > 应用可能会在单个帧内多次绘制同一个像素，这种情况称为“过度绘制”。过度绘制通常是不必要的，最好避免。它会浪费 GPU 时间来渲染与用户在屏幕上所见内容无关的像素，进而导致性能问题。

      举例：如果我们有若干界面卡片堆叠在一起，每张卡片都会遮盖其下面一张卡片的部分内容。

      但是，系统仍然需要绘制堆叠中的卡片被遮盖的部分。这是因为堆叠的卡片是根据 [Painter 算法](https://en.wikipedia.org/wiki/Painter's_algorithm)（也就是按从后到前的顺序）来渲染的。按照这种渲染顺序，系统可以将适当的透明度混合应用于阴影之类的半透明对象

      **常用工具：**

      - 设置-开发者选项-调试GPU过度绘制-显示过度绘制区域

      标准：

      ![](https://user-gold-cdn.xitu.io/2017/11/4/c616e9fa950d7e68baed5c4f0795985f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

      **每个颜色的说明如下**：

      原色：没有过度绘制

      蓝色：过度绘制1次

      绿色：过度绘制2次

      粉色：过度绘制3次

      红色：过度绘制4次及以上

      **可能原因：**

      - 无用的背景图片
      - 层级太深，太多叠加的View
      - 无用的父节点、子节点
      - 没有使用9patch图片

      **解决过度绘制问题的策略：**

      - 移除布局中不需要的背景
      - 使视图层次结构扁平化
      - 降低透明度

   4. 内存泄漏优化：

      1. 概念：内存泄漏（Memory leak）是在计算机科学中，由于疏忽或错误造成程序未能释放已经不再使用的内存。

      2. 在开发过程中避免写出有内存泄漏的代码

         常见场景：
         1. 单例——生命周期 

         2. Handler

            ###### 解决方法：

            ###### 1.创建静态Handler的匿名内部类 static class MyHandler extends Handler

            ###### 2.把对Handler持有的对象的使用弱引用 WeakReference context;

            ###### 3.在Activity销毁时移除消息队列中的任务或消息 handler.removeCallbacksAndMessages(null);取消所有的消息的处理

         3. 非静态内部类创建静态实例：非静态内部类和非静态匿名内部类中确实持有外部类的引用，静态内部类中未持有外部类的引用。隐式引用是导致内存泄漏的根本原因。
         4.  线程造成的内存泄漏
         5. 资源未关闭
         6. 系统服务、监听器未注销/移除
         7. 无限循环类型的动画

         3.通过一些分析工具（如MAT、Leakcanary）来找出潜在的内存泄漏，然后解决

   4. 内存溢出OOM：

      1. 概念：当应用的heap超过了Dalvik虚拟机分配的内存就会内存溢出
      2. 常见场景：
         1. 对象内存过大
         2. 布局重复加载、界面横竖屏切换、应用资源较多，来不及加载
         3. 内存泄露过多导致的内存溢出

      3. 解决方案：使用largeHeap，会请求系统为Dalvik虚拟机分配更大的内存空间。使用起来也很方便，只需在manifest文件application节点加入android:largeHeap=“true”即可。

21. 响应速度优化：避免ANR

     直观体验：用户在操作APP过程中，感觉界面卡顿，出现ANR对话框

     产生原因：

     1. Android规定，Activity如果5秒钟之内无法响应屏幕触摸事件或者键盘输入事件就会出现ANR，
     2. BroadcastReceiver如果10秒钟之内还未执行完操作也会出现ANR。
     3. Service如果20秒钟之内还未执行完操作也会出现ANR。

     典型场景：UI线程存在耗时操作（网络请求、文件操作、数据库操作、sleep函数等）

     追踪：/data/anr下会有traces.txt

22. ListView/RecycleView和Bitmap优化：

     ①使用ViewHolder模式来提高效率

     ②异步加载：耗时的操作放在异步线程中

     ③ListView/RecycleView的滑动时停止加载和分页加载

4. 线程优化：尽量采用线程池

# RxJava

- Observable：被订阅者，被订阅者是事件的来源，接收订阅者`(Observer)`的订阅，然后通过发射器`(Emitter)`发射数据给订阅者。
- Observer：订阅者，注册过程传给被订阅者，订阅者监听开始订阅，监听订阅过程中会把`Disposable`传给订阅者，然后在被订阅者中的发射器`(Emitter)`发射数据给订阅者`(Observer)`。
- Emitter：发射器，在发射器中会接收下游的订阅者`(Observer)`，然后在发射器相应的方法把数据传给订阅者`(Observer)`。
- Consumer：消费器
- Disposable：释放器

操作符：

1. empty：创建一个什么都不做直接通知完成的Observable
2. error：创建一个什么都不做直接通知错误的Observable
3. timer：创建一个在给定的延时之后发射数据项为0的Observable
4. interval：创建一个按照给定的时间间隔发射为0开始的整数序列的Observable

# EventBus 原理

1. ThreadMode，四种模式：PostThread、MainThread、BackgroundThread以及Async
   - PostThread：事件的处理和事件的发送在相同的进程 
   - MainThrad：事件的处理在UI线程中执行
   - BackgroundThread：如果事件是在UI线程中发布出来，那事件处理就会在子线程中运行，如果事件在子线程中发布出来，那么事件处理直接在子线程中执行
   - Async：事件处理在单独的线程中执行，主要用于在后台线程中执行耗时操作
2. 源码解析
   - register: 根据订阅者类名查找当前订阅者的所有事件响应函数-->循环每个事件响应函数-->得到该事件类型的所有订阅者信息，根据优先级将当前订阅者插入订阅者队列-->得到当前订阅者订阅的所有事件队列，将此事件保存到队列中-->判断是否是粘性事件--Yes-->取出粘性事件，post此事件给当前订阅者
   - post：从currentPostingThreadState得到当前线程的Post信息PostingThreadState，其中包括事件队列-->将当前事件添加到当前线程的事件队列中-->判断事件是否分发中-->循环事件队列中的每个事件-->查找该事件的所有订阅者，循环每个订阅者-->根据ThreadMode，在不同的线程调用订阅者的事件响应函数

   ![EventBus-register](http://ucoon.gitee.io/myblogimg/EventBus-register.png)

   ![EventBus-post](http://ucoon.gitee.io/myblogimg/EventBus-post.png)

3. 优缺点

   - 优点：简化组件之间的通信方式，实现解耦让业务代码更加简洁，可以动态设置事件处理线程和优先级

   - 缺点：每个事件都必须自定义一个事件类，造成事件类太多，增加维护成本

   # 屏幕适配方案：

   ```java
 dp：设备独立像素，在不同分辨率和尺寸的手机上代表了不同的真实像素
    density：屏幕密度
    dpi：像素密度
    关系：density=dpi/160  px=dp*(dpi/160)
    ==> density=px/dp
   ```

    1. dp直接适配（最原始的Android适配方案）
   
    2. 宽高限定符适配
       
    3. AndroidAutoLayout——鸿洋
       
    4. smallestwidth限定符适配
       
    5. 今日头条适配方案：它通过修改density值，强行把所有不同尺寸分辨率的手机的宽度dp值改成一个统一的值，这样就解决了所有的适配问题
       
       dpi=density*160

# Java sleep()和wait()区别

```java

```

# Android 各版本特性

```java
Android 10.0:
1. 设备硬件信息读取限制：在Android10中, 系统不允许普通App请求android.permission.READ_PHONE_STATE权限, 故新版App需要取消该动态权限的申请。
2. 后台App启动限制：使用全屏Intent(full-screen intent), 既创建通知栏通知时, 加入full-screen intent 设置
3. Android沙盒化存储
4. 用户的定位权限的变更
5.使用无线扫描权限
Android 9.0：
1. 限制Http网络请求（明文传输）
解决方式：
	1.在资源目录中新建一个 xml 文件作为网络安全配置文件，例如 xml/network_security_config.xml，内容如下
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
在AndroidManifest.xml进行配置：android:networkSecurityConfig="@xml/network_security_config"
	2. <application android:usesCleartextTraffic=["true" | "false"]>
2. 弃用Apache Http Client
解决方式：
	1. <uses-library android:name="org.apache.http.legacy" android:required="false"/>
	2. 在应用内部直接将Apache Http Client相关的类打包并引用
3. 限制非SDK接口的调用，非SDK接口分类：light-greylist(浅灰名单)、dark-greylist(深灰名单)以及blacklist(黑名单)
light-greylist(浅灰名单)：对于此名单中的非 SDK 接口，官方暂未找到可替代的 SDK 接口，因此开发者仍可继续访问（如果 targetSdkVersion 大于等于28时会出现警告）。
dark-greylist(深灰名单)：targetSdkVersion 小于28时仍可继续使用此名单中的接口，但会出现警告提示；大于等于28时，这些接口将会限制访问。
blacklist(黑名单)：无论 targetSdkVersion 为多少，只要应用运行在 Android 9.0 平台上，访问此名单中的接口都会受限
4. 前台服务权限：在Android 9.0设备上，应用使用前台服务之前必须先申请 FOREGROUND_SERVICE 权限，否则就会抛出 SecurityException 异常。
5. 强制执行FLAG_ACTIVITY_NEW_TASK要求：开发者需要通过非 Activity context 启动 Activity，就必须设置 Intent 标志 FLAG_ACTIVITY_NEW_TASK，否则会启动失败并抛出异常
Android 8.0:
1. 通知栏：Android 8.0 引入了通知渠道，其允许您为要显示的每种通知类型创建用户可自定义的渠道。用户界面将通知渠道称之为通知类别。针对 8.0 的应用，创建通知前需要创建渠道，创建通知时需要传入 channelId，否则通知将不会显示
2. 后台执行限制：如果针对 Android 8.0 的应用尝试在不允许其创建后台服务的情况下使用 startService() 函数，则该函数将引发一个 IllegalStateException。
3. 允许安装未知来源应用：针对 8.0 的应用需要在 AndroidManifest.xml 中声明 REQUEST_INSTALL_PACKAGES 权限，否则将无法进行应用内升级
4. 桌面图标适配

	
```

# 蓝牙开发

>蓝牙是一种短距离的无线通信技术，可以实现固定设备、移动设备之间的数据交换。一般将蓝牙分为两大类，蓝牙3.0规范之前的版本称为传统蓝牙，蓝牙4.0规范之后的版本称为低功耗蓝牙，也就是常说的BLE（Bluetooth Low Energy）。

![蓝牙开发流程](http://ucoon.gitee.io/myblogimg/蓝牙开发流程.jpg)

- **1.BluetoothAdapter**

本地蓝牙适配器，用于一些蓝牙的基本操作，比如判断蓝牙是否开启、搜索蓝牙设备等。

- **2.BluetoothDevice**

蓝牙设备对象，包含一些蓝牙设备的属性，比如设备名称、mac地址等。

- **3.BluetoothProfile**

一个通用的蓝牙规范，设备之间按照这个规范来收发数据。

- **4.BluetoothGatt**

蓝牙通用属性协议，定义了BLE通讯的基本规则，是BluetoothProfile的实现类，Gatt是Generic Attribute Profile的缩写，用于连接设备、搜索服务等操作。

- **5.BluetoothGattCallback**

蓝牙设备连接成功后，用于回调一些操作的结果，**必须连接成功后才会回调**。

- **6.BluetoothGattService**

蓝牙设备提供的服务，是蓝牙设备特征的集合。

- **7.BluetoothGattCharacteristic**

蓝牙设备特征，是构建GATT服务的基本数据单元。

- **8.BluetoothGattDescriptor**

蓝牙设备特征描述符，是对特征的额外描述。



# 混合开发

1. Android调用JS方法

   ```java
   1. webView.loadUrl("javascript:javatojscallback('我来自Java')");
   2. webView.evaluateJavascript("javascript:javatojswith('我来自Java')",
               new ValueCallback<String>() {
           @Override
           public void onReceiveValue(String s) {
               textShow.setText(s);
           }
       });
   需要注意的是，evaluateJavascript()只能在android 4.4之后才能调用。
   ```

2. JS调用Android

   ```java
   1. addJavascriptInterface
   ```
