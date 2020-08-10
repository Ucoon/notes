---
title: 如何构建Android MVVM应用框架
---

![MVC、MVP和MVVM区别](https://mmbiz.qpic.cn/mmbiz_jpg/v1LbPPWiaSt5eqj5m50sm0psVSyM211c9rYMicibweHpfflKOGr7BbSRcsdV2KfrVkaATmAKExVaAwVPNgraaxiaiaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**MVVM各层的作用**

**模型层（Model）**：

模型层（Model），负责处理数据逻辑，一般包含数据库、本地数据、网络获取的Bean、（这里我单独抽取的视图数据ViewData概念也属于Model层）等组成

**视图层（View）**：

View层做的就是和UI相关的工作，我们只在XML和Activity或Fragment写View层的代码，View层不做和业务相关的事，也就是我们的Activity 不写和业务逻辑相关代码，也不写需要根据业务逻辑来更新UI的代码，因为更新UI通过**数据绑定**实现，更新UI在ViewModel里面做（更新绑定的数据源即可）。**简单的说：View层不做任何业务逻辑、不涉及操作数据、不处理数据、UI和数据严格的分开。**

**ViewModel层**：

ViewModel层做的事情刚好和View层相反，ViewModel 只做和业务逻辑和业务数据相关的事，不做任何和UI、控件相关的事，ViewModel 层不会持有任何控件的引用，更不会在ViewModel中通过UI控件的引用去做更新UI的事情，VM通过数据绑定连接View和Model实现视图层和模型层的解藕，事件触发后通过ViewModel处理业务逻辑，并且通过数据驱动的方式修改视图数据，而达到间接修改视图的功能。

**ViewModel库：**

>ViewModel是以生命周期的方式存储与管理UI相关数据，
>
>作用：
>1、在MVVM模式中，使Model与View分离
>2、负责为ui准备数据
>3、存储数据

**AndroidViewModel VS ViewModel**：

```java
由于 ViewModel 生命周期可能长与 activity 生命周期，所以为了避免内存泄漏Google禁止在ViewModel中持有Context或activity或view的引用。如果非得使用Context，可以继承AndroidViewModel类中获取ApplicationContext
```

![](https://user-gold-cdn.xitu.io/2019/10/8/16daa4928d643d91?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**注意：**

1. **ViewModel一定不能持有视图层的引用，同样不能持有Context的引用！不然还是MVP！**
2. 官方的ViewModel库并不是实现MVVM架构的必备，**MVVM的重点是解藕，通过一定方式解除View和Model的耦合**，比如使用数据绑定库DataBinding。
3. 也有不使用DataBinding实现的MVVM吗？其实也有，比如说第三版的《第一行代码》中的方式，**利用LiveData实现View和Model的解藕，且ViewModel不依赖View和Context**，这里把Activity和Fragment当作View的主体，而我更倾向于把XML当作View的主体，所见即所得，看得到的当成View，会更直观一点。Activity和Fragment只是当作一个粘合剂，比如进行事件绑定和一些复杂动画的处理等。所以DataBinding更多的是服务于XML这种View的。
4. ViewModel库是在DadaBinding库之后才有的，ViewModel类旨在以注重生命周期的方式存储和管理界面相关的数据。ViewModel 类让数据可在发生屏幕旋转等配置更改后继续留存，这样可以更好的提升用户体验和提高应用性能。

**数据绑定：**

1. DataBinding：

   >Databinding 是一种框架，MVVM是一种模式，两者的概念是不一样的。我的理解DataBinding是一个实现数据和UI绑定的框架，只是一个实现MVVM模式的工具。ViewModel和View可以通过DataBinding来实现单向绑定和双向绑定，这套UI和数据之间的动态监听和动态更新的框架Google已经帮我们做好了。在MVVM模式中ViewModel和View是用绑定关系来实现的，所以有了DataBinding 使我们构建Android MVVM 应用程序成为可能。
   >
   >作者：Kelin
   >链接：https://www.jianshu.com/p/2fc41a310f79
   >来源：简书
   >著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

   

3. LiveData：

   >**简介**：LiveData 是一个有生命周期感知 & 可观察的数据持有者类
   > **作用**： 持久化的观察数据的更改与变化
   > **特点**：
   > 1、感知对应Activity的生命周期，只有生命周期处于onStart与onResume时，LiveData处于活动状态，才会把更新的数据通知至对应的Activity
   > 2、当生命周期处于onStop或者onPause时，不回调数据更新，直至生命周期为onResume时，立即回调
   > 3、当生命周期处于onDestory时，观察者会自动删除，防止内存溢出
   > 4、共享资源。可以使用单例模式扩展LiveData对象以包装系统服务，以便可以在应用程序中共享它们，同时有遵守了以上生命周期
   >
   >LiveData有2个方法通知数据改变：
   >
   >- 同步：.setValue（value）接收端数据回调与发送端同一个线程（UI线程中使用）
   >- 异步：.postValue（value）接收端在主线程回调数据（非UI线程中使用）
   >
   >作者：岩浆李的游鱼leo2
   >链接：https://juejin.im/post/5d9d8f756fb9a04dd8591b8e
   >来源：掘金
   >著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
   
   