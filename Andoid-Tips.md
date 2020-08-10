1. Activity 解析

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

   4. 横竖屏切换

      ```
      onPause --> onSaveInstanceState --> onStop --> onDestroy --> onCreate --> onStart --> onRestoreInstanceState --> onResume
      ```

      指定属性```configChanges```避免横竖屏切换

   5. 启动模式

      ```java
      standard(标准模式)、singleTop(栈顶复用)、singleTask(栈内复用)、singleInstance(单例模式)
      ```

2. Service 解析

   	![service生命周期](D:\MyDocument\Picture\笔记\service生命周期.png)

   1. bindService 与 startService 区别

      ```java
      1. startService：不进行通信，停止服务使用stopService
         bindService：进行通信，停止服务使用unbindService
      2. 生命周期:
      	startService：onCreate --> onStartCommand --> onDestroy
      	bindService: onCreate --> onBind --> onUnBind --> onDestroy
      ```

   2. Service可以执行耗时操作吗

      ```java
      service同样运行在UI线程，而耗时操作(如网络请求、文件操作)会阻塞UI线程，给用户不好的体验
      如果需要在服务中进行耗时操作，可以选择IntentService，IntentService是Service的子类，用来处理异步请求；IntentService在onCreate()方法中通过HandlerThread单独开启一个线程来处理Intent请求对象所对应的任务，这样可以避免事务处理阻塞主线程。
      ```

   3. IntentService 解析

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

3. BroadcastReceiver 解析

   ```java
   
   ```

4. ContentProvider 解析

   ```java
   
   ```

5. Android 消息机制

   ```java
   主要指Handler的运行机制，Handler的运行又与Message、MessageQueue、Looper紧密相关。简单的来说，当Handler发送消息时，将会调用MessageQueue.enqueueMessage，向消息队列（实质上是单链表）中添加消息，当通过Looper.loop开启循环后，会不断地从线程池中读取消息，即调用MessageQueue.next，然后调用目标Handler（即发送该消息的Handler）的dispatchMessage方法传递消息，然后返回到Handler所在线程，目标Handler收到消息，调用handleMessage方法，接收消息，处理消息。
   ```

   ![handler_消息机制](D:\MyDocument\Picture\笔记\handler_消息机制.jpg)

6. HandlerThread 解析

7. 事件分发机制

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
   事件最先传递到Activity的dispatchTouchEvent()进行事件分发
   然后调用Window类实现类PhoneWindow的SuperDispatchTouchEvent()
   调用DecorView的SuperDispatchTouchEvent()
   调用ViewGroup的dispatchTouchEvent()
   如果ViewGroup的onInterceptTouchEvent()返回true，则表示它要拦截当前事件，事件就由ViewGroup自己处理，调用自身的onTouchEvent()；如果返回false，表示它不拦截当前事件，这时事件会传递到子元素，子元素的dispatchTouchEvent()会被调用，如此反复
   ```

8. View的工作过程：

   ```java
   主要指measure、layout和draw过程，View的绘制流程是从ViewRoot的performTraversals，它经过measure、layout和draw三个过程才能最终将一个view绘制出来，针对performTraversals的大致流程，如下图所示：
   ```

   ![performTraversals的工作流程](D:\MyDocument\Picture\笔记\performTraversals的工作流程.png)

   ```java
   performTraversals会依次调用performMeasure、performLayout和performDraw这三个方法，这三个方法分别完成顶级View的measure、layout和draw这三大流程，其中performMeasure会调用measure方法（measure方法是一个final类型的方法），在measure方法中又会调用onMeasure方法，在onMeasure方法中则会对所有的子元素进行measure过程，这个时候measure流程就从父容器传递到子元素中了，完成了一次measure过程。接着子元素会重复父容器的measure过程，如此反复就完成了整个view树的遍历。同理，performLayout和performDraw的传递流程和performMeasure是类似的
   ```

   1. Measure过程：MeasureSpec.Mode的三种测量模式：UNSPECIFIED、EXACTLY、AT_MOST
   2. layout过程：
   3. draw过程：

9. Android 自定义view过程

   ```java
   1. 构造函数-->onMeasure-->onSizeChanged-->onLayout-->onDraw-->提供接口(属性设置函数)
   ```

10. Android中的动画

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

11. Android的进程间通信

    ```java
    1.使用Intent
    2.使用文件共享
    3.使用Messenger
    ```

    
    
12. ```Activity```启动模式(```launchMode```)

    ```java
    standard、singleTop、singleTask、singleInstance
    ps：Activity的启动模式设置为singleTask或Intent设置了FLAG_ACTIVITY_NEW_TASK，在startActivityForResult之后，会立即被返回Activity.RESULT_CANCELED，再执行页面跳转
    ```

13. 引申：点击通知栏跳转到Activity B，点击返回如何跳转到MainActivity（应用已被KILL掉）

    ```java
    1. 使用 TaskStackBuilder：任务栈的创造者
    2. 直接使用PendingIntent.getActivities()
    ```

14. Android App包瘦身优化实践

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

15. 数据结构：

    1. 线性表(```linked list```)

       线性表是一种线性结构，它是具有相同类型的n（n>0）个数据元素组成的有限序列

       组成部分：数组、单向链表、双向链表

       1. 数组：数组有上届和下界，数组的元素在上下界内是连续的；特点：数据连续，随机访问速度快
       2. 单向链表：链表的一种，由节点组成，每个节点都包含下一个节点的指针（后继节点）；特点：节点的链接方向是单向的，相对于数组来说，单链表的随机访问速度较慢，但是单链表删除/添加数据的效率很高
       3. 双向链表：链表的一种，由节点组成，每个节点都包含两个指针，分别指向直接后继和直接前驱

    2. 栈(```Stack```)

       线性存储结构；特点：后进先出；向栈中添加/删除数据，只能从栈顶操作

       三种操作：push、peek、pop

    3. 队列(```Queue```)

       线性存储结构；特点：先进先出，队首进行删除操作，队尾进行插入操作

    4. 树(```Tree```)

       由n(n>=1)个有限节点组成一个有层级关系的集合

    5. 哈希表(```hash table```)

       也叫散列表，是根据键码值(key-value)而直接进行访问的数据结构

16. Http与Https

    1. 对称加密、非对称加密和MD5散列算法

       >对称加密：加、解密使用的是同一串密钥，所以被称为对称加密，对称加密只有一个密钥作为私钥。常见的
       >
       >对称加密算法有：DES、AES等
       >
       >非对称加密：指的是加、解密使用不同的密钥，一把作为公开的公钥，另一把作为私钥。公钥加密的信息，只有私钥才能解密，反之，私钥加密的信息，只有公钥才能解密。最常用的非对称加密算法：RSA

       HTTPS = HTTP + SSL/TLS

       HTTPS请求流程：

       ```java
       1. 客户端发起HTTPS请求
       2. 服务端的配置：公钥和私钥，公钥给别人加密使用，私钥给自己解密使用
       3. 服务端传送证书给客户端：证书即为公钥
       4. 客户端解析证书：这部分工作由客户端的TLS完成，首先会验证公钥是否有效，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值，然后用证书对该随机值进行加密
       5. 客户端传送加密信息：这部分传送的是用证书加密后的随机值，目的是让服务端得到这个随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密。
       6. 服务端解密信息：服务端用私钥解密后，得到客户端传过来的随机值，然后把内容通过该随机值进行对称加密
       7. 服务端传输加密后的信息：这部分信息是服务端用随机值加密后的信息
       8. 客户端解密信息，客户端用之前生成的随机值解密服务端传过来的信息
       ```

17. 常见的xml解析方式：SAX解析、Pull解析和DOM解析

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

18. 进程保活：

    ```java
    1. 监听全局的静态广播：时间更新、开机广播、网络状态、解锁加锁亮屏暗屏（3.1版本）
    2. 定时器、JobScheduler
    3. 双进程、双Service守护
    4. 提高Service优先级
    5. 前台通知
    6. 监听锁屏广播打开1像素Activity（基于onStartCommand() return START_STICKY）
    ```

19. Java内存管理：
    1. java垃圾回收机制：
       内存回收就是释放掉在内存中已经没有用的对象
       怎么判断没用的对象：

       1. 采用标记计数法
           在这种方法中，堆中每个对象实例都有一个引用计数。当一个对象被创建时，就将该对象实例分配给一个变量，该变量计数设置为1。当任何其它变量被赋值为这个对象的引用时，计数加1（a = b,则b引用的对象实例的计数器+1），但当一个对象实例的某个引用超过了生命周期或者被设置为一个新值时，对象实例的引用计数器减1。任何引用计数器为0的对象实例可以被当作垃圾收集。当一个对象实例被垃圾收集时，它引用的任何对象实例的引用计数器减1。
       2. 采用根可到达法
           可达性分析算法是从离散数学中的图论引入的，程序把所有的引用关系看作一张图，从一个节点GC ROOT开始，寻找对应的引用节点，找到这个节点以后，继续寻找这个节点的引用节点，当所有的引用节点寻找完毕之后，剩余的节点则被认为是没有被引用到的节点，即无用的节点，无用的节点将会被判定为是可回收的对象。

    2. Java中的引用：强引用、弱引用、软引用和虚引用

       ```java
       强引用：类似Object object = new Object()，只要强引用还存在，垃圾回收器永远不会回收掉被引用的对象
       软引用：用来描述有用非必须的引用对象，对于软引用关联的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之内
       弱引用：用来描述非必须的对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象
       虚引用：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。它的作用是能在这个对象被收集器回收时收到一个系统通知。
       ```
       无论引用计数算法还是可达性分析算法都是基于强引用而言的。

20. Android中的性能优化：布局优化、绘制优化、内存泄漏优化、响应速度优化、ListView/RecycleView和Bitmap优化、线程优化

    10. 布局优化：减少布局文件的层级

       如何进行布局优化：

       1. 删除布局中无用的控件和层次，其次有选择的使用性能较低的ViewGroup
       2. 采用include、merge、ViewStub标签
       3. 避免过度绘制

    11. 绘制优化：在View的onDraw里避免执行大量的操作：在onDraw里面不要创建新的局部变量；onDraw函数里不做耗时操作或者大量循环

    12. 内存泄漏优化：

        1. 在开发过程中避免写出有内存泄漏的代码

           常见场景：1. 单例——生命周期   2. Handler和AsyncTask 3. 非静态内部类创建静态实例 4. 线程造成的内存泄漏

           5. 资源未关闭

        2. 通过一些分析工具（如MAT、Leakcanary）来找出潜在的内存泄漏，然后解决

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

23. 线程优化：尽量采用线程池

24. EventBus 原理：

    1. ThreadMode，四种模式：PostThread、MainThread、BackgroundThread以及Async

       ```java
       PostThread：事件的处理和事件的发送在相同的进程
       MainThrad：事件的处理在UI线程中执行
       BackgroundThread：如果事件是在UI线程中发布出来，那事件处理就会在子线程中运行，如果事件在子线程中发布出来，那么事件处理直接在子线程中执行
       Async：事件处理在单独的线程中执行，主要用于在后台线程中执行耗时操作
       ```

    2. 源码解析

       ```java
       register: 根据订阅者类名查找当前订阅者的所有事件响应函数-->循环每个事件响应函数-->得到该事件类型的所有订阅者信息，根据优先级将当前订阅者插入订阅者队列-->得到当前订阅者订阅的所有事件队列，将此事件保存到队列中-->判断是否是粘性事件--Yes-->取出粘性事件，post此事件给当前订阅者
       post：从currentPostingThreadState得到当前线程的Post信息PostingThreadState，其中包括事件队列-->将当前事件添加到当前线程的事件队列中-->判断事件是否分发中-->循环事件队列中的每个事件-->查找该事件的所有订阅者，循环每个订阅者-->根据ThreadMode，在不同的线程调用订阅者的事件响应函数
       ```

25. 屏幕适配方案：

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

15. Java sleep()和wait()区别

    ```java
    
    ```

16. Android 各版本特性

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
