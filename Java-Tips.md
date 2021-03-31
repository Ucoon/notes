---
title: Android面试——Java高频题
---

前言：记录Android面试中常见的Java题目

#  数据结构

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

# Java集合——ArrayList

1. **以数组实现**。节约空间，但数组有容量限制。超出限制时会增加50%容量，用System.arraycopy()复制到新的数组，因此最好能给出数组大小的预估值。默认第一次插入元素时创建大小为10的数组。（`private static final int DEFAULT_CAPACITY = 10;`）
2. 按数组下标访问元素——**get(i)/set(i, e)的性能很高，这是数组的基本优势**。
3. 直接在数组末尾加入元素——add(e)的性能也高，但如果**按下标插入、删除元素——add(i, e)，remove(i)，remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是基本劣势**。

ArrayList是一个相对来说比较简单的数据结构，最重要的一点就是它的自动扩容，可以认为就是我们常说的**“动态数组”**。

## add函数

```java
/**
* Appends the specified element to the end of this list.
*
* @param e element to be appended to this list
* @return <tt>true</tt> (as specified by {@link Collection#add})
*/
public boolean add(E e){
    ensureCapacityInternal(size+1); // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

`ensureCapacityInternal()`是自动扩容机制的核心。

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 扩展为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果扩为1.5倍还不满足需求，直接扩为需求值
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

# Java集合——LinkedList

1. **以双向链表实现**。链表无容量限制，但双向链表本身使用了更多空间，需要额外的链表指针操作
2. 按下标访问元素——get(i)/set(i, e)，**要遍历链表将指针移动到位**
3. **插入、删除元素时修改前后节点的指针即可**，但还是要遍历部分链表的指针才能移动到下标所指的位置，只有在链表两头的操作——add()，addFirst()，removeLast()或者用iterator()上的remove()能省掉指针的移动。

# ArrayList和LinkedList的区别

1. ArrayList是基于动态数组的数据结构，Linked基于链表的数据结构
2. 对于随机访问get和set，ArrayList优于LinkedList，因为LinkedList要移动指针
3. 对于新增和删除操作，LinkedList比较占优势，因为ArrayList要移动数据

**应用场景**

ArrayLis使用在查询比较多，但是插入和删除比较少的情况；而LinkedList用在查询比较少而插入删除比较多的情况

# Java集合——HashMap

1. 特点：基于Map接口的实现，存储键值对，允许null键值，是非同步的，不保证有序，也不保证时序不随时间变化，HashMap存储着Entry(hash、key、value、next)对象

2. 两个重要参数：容量(`Capacity`)和负载因子(`load factor`)
     //初始容量，即最开始线性表的大小为16
     DEFAULT_INITIAL_CAPACITY = 1<<4
     //初始的负载因子
     DEFAULT_LOAD_FACTOR = 0.75

3. 实现原理：

   ```java
   1.HashMap底层是由数组加链表结构实现的。数组用来存放元素位置，链表用来解决hash冲突。当往HashMap中添加对象时，先计算key的hashCode，然后根据hashCode计算出元素应该放到数组的哪个位置。找到相应的位置，判断该位置是否已经存在键值对，如果已存在，那么覆盖掉原来的value，如果不存在，就放到该位置。链表的存在就是为了解决不同key出现hash冲突的问题，一般元素会放在链表头，这样做减少操作。
   
     当线性表中的元素（这里指的是HashMap中元素的总个数）>初始容量*负载因子时，线性表会进行扩容操作，将数组长度变为原来的2倍，然后将元素重新计算hashCode放到相应位置（为什么要重新计算hashCode？：虽然key的hashCode不变，但是数组长度变了，在根据hashCode计算数组位置时，得出的索引值肯定是不同的，如果平移过来，会直接导致扩容前添加到HashMap中的数据无法被get到，因为在数组中索引变了）。
   ```

4. 非同步导致线程不安全：在多线程操作情况下什么时候线程不安全？

   ```java
   1.如果在多个线程，在某一时刻同时操作HashMap并执行put操作。可能会有大于两个key的hash值相同，这个时候需要解决碰撞冲突
   2.put方法不是同步的，同时调用了addEntry方法，addEntry方法依然不是同步的
   3.HashMap存在扩容现象，扩容方法也不是同步的
   解决：使用Collections.synchronizedMap(new HashMap(...))
   ```

5. ```TreeMap```和```LinkedHashMap```

   ```TreeMap```实现了```SortedMap```接口，对插入的记录根据key排序，默认按照升序排序

   ```LinkedHashMap```是Hash表和链表的实现，并且依靠双向链表保证了迭代顺序是插入的顺序

6. ```ConcurrentHashMap```和```HashTable```

   1.```HashTable```是线程安全的，它使用```synchronized```来做线程安全，全局只有一把锁

   2.```ConcurrentHashMap```引入了分段锁的技术，即把一个大的Map拆分成N个小的```HashTable```

# TCP 与 UDP区别

1. TCP（Transmission Control Protocol，传输控制协议）是基于连接的协议，也就是说，在正式收发数据前，必须和对方建立可靠的连接。（三次握手）
2. UDP（User Data Protocol，用户数据报协议）是基于非连接的协议，它不与对方建立连接，而是直接发送数据包，UDP适用于一次只传送少量数据、对可靠性要求不高的应用环境，比如“ping”命令。

|            | TCP          | UDP        |
| ---------- | ------------ | ---------- |
| 是否连接   | 面向连接     | 面向非连接 |
| 传输可靠性 | 可靠         | 不可靠     |
| 应用场合   | 传输大量数据 | 少量数据   |
| 速度       | 慢           | 快         |

# TCP三次握手，四次挥手

```java
1.三次握手
  第一次握手: 建立连接时，客户端发送syn包(syn=j)到服务器，并进入SYN_SENT状态，等待服务器确认；  
  第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态;
  第三次握手：客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手
  为什么要三次握手：防止已失效的连接请求报文段突然又传送到服务端，因而产生错误
2.四次挥手
  1.客户端发送Fin报文，用来关闭客户端到服务端的数据传送，客户端进入FIN_WAIT_1状态
  2.服务端收到之后发送Ack报文，确认序号为收到序号+1，服务端进入CLOSE_WAIT状态
  3.服务端向客户端发送FIN报文段，用来关闭服务端到客户端的数据传送，服务端进入LAST_ACK状态
  4.客户端收到服务端发送的FIN报文段，向服务端发送ACK报文段，然后客户端进入TIME_WAIT状态，服务端收到客户端的ACK报文之后就关闭连接，客户端等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，客户端也可以关闭连接了。
```

补充：

在TCP层，有个FLAGS字段，这个字段有以下几个标识：SYN、FIN、ACK、PSH、RST、URG，其中，对于我们日常的分析有用的就是前面的五个字段。

它们的含义是：

```
SYN表示建立连接，
FIN表示关闭连接，
ACK表示响应，
PSH表示有 DATA数据传输，
RST表示连接重置
```

一条TCP连接在其生存期内会经历一系列状态。TCP连接包含11个状态。分别是：LISTEN、SYN_SENT、SYN_RECEIVED、ESTABLISHED、FIN_WAIT_1、FIN_WAIT_2、CLOSE_WAIT、CLOSING、LAST_ACK、TIME_WAIT以及最终状态CLOSED。

它们的含义是：

```java
LISTEN: 表示服务端正在等待来自TCP客户端的连接请求。服务端为此要调用bind()以及listen()函数。
SYN_SENT : 客户端发起连接，发送SYN给服务端后，进入此状态。
SYN_RECEIVED : 服务端接收到客户端的SYN请求，服务端由LISTEN状态进入SYN_RECEIVED状态，同时发送一个ACK和一个SYN给客户端。另一种情况 是客户端发送SYN的同时接收到服务器的SYN请求，客户端就会由SYN_SENT状态进入SYN_RECEIVED状态。
ESTABLISHED : 表示经过三次握手，连接已经建立，可以在两个方向上传输数据。这是一条连接在数据传输阶段的常见状态。
FIN_WAIT_1 : 主动关闭的一方发送FIN，由ESTABLISHED状态进入FIN_WAIT_1状态。
FIN_WAIT_2 : 主动关闭的一方，接收到对方的FIN ACK,由FIN_WAIT_1 状态，进入FIN_WAIT_2 状态。
CLOSE_WAIT : 被动关闭的一方，在接收到FIN后，由ESTABLISHED状态进入此状态。
LAST_ACK : 被动关闭的一方，发起关闭请求，由CLOSE_WAIT状态，进入此状态。在接收到ACK后，会进入CLOSED状态。
```

# Http与Socket

1. http协议即超文本传送协议（Hypertext Transfer Protocol），http连接就是所谓的短连接，即客户端向服务端发送一次请求，服务端响应后连接会立即断掉
2. socket连接就是所谓的长连接，理论上客户端和服务端一旦建立起连接将不会主动断掉，但是由于各种环境因素可能会是连接断开。===>为了维持连接需要发送心跳消息
   1. Socket是一个针对TCP和UDP编程的接口，你可以借助它建立TCP连接等等。而TCP和UDP协议属于传输层 。
   2. Socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Socket，我们才能使用TCP/IP协议。Socket的出现只是使得程序员更方便地使用TCP/IP协议栈而已，是对TCP/IP协议的抽象，从而形成了我们知道的一些最基本的函数接口。

# Socket心跳包设计

心跳检测步骤：

1. 客户端每隔一个时间间隔发生一个探测包给服务器
2. 客户端发包时启动一个超时定时器
3. 服务器端接收到检测包，应该回应一个包
4. 如果客户机收到服务器的应答包，则说明服务器正常，删除超时定时器
5. 如果客户端的超时定时器超时，依然没有收到应答包，则说明服务器挂了

# HTTP/HTTPS

1. Http协议包括哪些请求：

   Http1.0定义了三种请求方式：GET、POST、HEAD

   Http1.1新增六种请求方式：OPTIONS、PUT、DELECT、TRACE、CONNECT、PATCH

2. HTTP中POST和GET的区别

   1. 报文格式不同：GET是将请求参数加到URL中，POST是将请求数据放在请求体中
   2. GET传送的数据量较小，不能超过2KB，POST传送的数据量较大，默认为不受限制；实际上，HTTP 协议没有 Body 和 URL 的长度限制，对 URL 限制的大多是浏览器和服务器的原因。

3. HTTP和HTTPS的区别

   |              | HTTP             | HTTPS          |
   | ------------ | ---------------- | -------------- |
   | URL          | http://          | https://       |
   | 安全性       | 不安全，明文传输 | 安全，加密传输 |
   | 标准端口     | 80               | 443            |
   | OSI网络模型  | 应用层           | 传输层         |
   | 是否需要证书 | 不需要           | 需要           |

4. 对称加密、非对称加密和MD5散列算法

   >对称加密：加、解密使用的是同一串密钥，所以被称为对称加密，对称加密只有一个密钥作为私钥。常见的
   >
   >对称加密算法有：DES、AES等
   >
   >非对称加密：指的是加、解密使用不同的密钥，一把作为公开的公钥，另一把作为私钥。公钥加密的信息，只有私钥才能解密，反之，私钥加密的信息，只有公钥才能解密。最常用的非对称加密算法：RSA

5. HTTPS = HTTP + SSL/TLS

   HTTPS请求流程：

   ```
   1. 客户端发起HTTPS请求
   2. 服务端的配置：公钥和私钥，公钥给别人加密使用，私钥给自己解密使用
   3. 服务端传送证书给客户端：证书即为公钥
   4. 客户端解析证书：这部分工作由客户端的TLS完成，首先会验证公钥是否有效，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值，然后用证书对该随机值进行加密
   5. 客户端传送加密信息：这部分传送的是用证书加密后的随机值，目的是让服务端得到这个随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密。
   6. 服务端解密信息：服务端用私钥解密后，得到客户端传过来的随机值，然后把内容通过该随机值进行对称加密
   7. 服务端传输加密后的信息：这部分信息是服务端用随机值加密后的信息
   8. 客户端解密信息，客户端用之前生成的随机值解密服务端传过来的信息
   ```

# Java内存管理

Java内存管理就是对象的分配和释放问题。在Java中，内存的分配是由程序完成，而内存的释放是由Java垃圾回收器（GC）完成的；为了能够正确释放对象，GC必须监控每一个对象的运行状态，包括对象的申请、引用、被引用、赋值等，监控对象状态是为了更加准确地、及时地释放对象，而**释放对象的根本原则就是该对象不再被引用。**

## Java内存分配策略

内存分配策略有三种：静态分配、栈式分配和堆式分配，三种方式所使用的内存空间分别是静态存储区（方法区）、栈区和堆区

1. 静态存储区：主要存放静态变量。这块内存在程序编译时就已经分配好了，并且在程序整个运行期间都存在
2. 栈区：当方法执行时，方法体内的局部变量（包括基本数据类型，对象的引用）都在栈上创建，并在方法执行结束时，这些局部变量所持有的内存将会自动被释放。因为栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限
3. 堆区：又称动态内存分配，通常就是指程序运行时直接new出来的内存，也就是对象的实例，这部分内存在不使用时将会被Java垃圾回收器来负责回收。

## Java垃圾回收器

垃圾回收（Garbage Collection）是Java虚拟机垃圾回收器提供的一种用于在**空闲时间**不定时回收**无任何对象引用**的对象占据的内存空间的一种机制。（注意：垃圾回收，回收的是无任何引用的对象占据的内存空间而不是对象本身）

**引用：**如果Reference类型的数据中存储的数值代表的是另外一块内存的起始地址，就称这块内存代表着一个引用（**引用都有哪些，对垃圾回收又有什么影响**）

Java中的引用：强引用、弱引用、软引用和虚引用

1. 强引用：类似Object object = new Object()，只要强引用还存在，垃圾回收器永远不会回收掉被引用的对象
2. 软引用：用来描述有用非必须的引用对象，对于软引用关联的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之内
3. 弱引用：用来描述非必须的对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象
4. 虚引用：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。它的作用是能在这个对象被收集器回收时收到一个系统通知。

**垃圾：**无任何对象引用的对象（**通过哪些算法找到这些对象**）

1. 采用标记计数法
   在这种方法中，堆中每个对象实例都有一个引用计数。当一个对象被创建时，就将该对象实例分配给一个变量，该变量计数设置为1。当任何其它变量被赋值为这个对象的引用时，计数加1（a = b,则b引用的对象实例的计数器+1），但当一个对象实例的某个引用超过了生命周期或者被设置为一个新值时，对象实例的引用计数器减1。任何引用计数器为0的对象实例可以被当作垃圾收集。当一个对象实例被垃圾收集时，它引用的任何对象实例的引用计数器减1。
2. 采用根可到达法
   可达性分析算法是从离散数学中的图论引入的，程序把所有的引用关系看作一张图，从一个节点GC ROOT开始，寻找对应的引用节点，找到这个节点以后，继续寻找这个节点的引用节点，当所有的引用节点寻找完毕之后，剩余的节点则被认为是没有被引用到的节点，即无用的节点，无用的节点将会被判定为是可回收的对象。

无论引用计数算法还是可达性分析算法都是基于强引用而言的。

**回收：**清理“垃圾”占用的内存空间而非对象本身（**怎么通过算法实现回收呢**）

1. 标记—清除算法
2. 复制算法
3. 标记—整理算法
4. 分代回收算法：JVM采用分代垃圾回收，在JVM的内存空间中把堆空间分为年老代和年轻代。将大量创建了没多久就会消亡的对象存储在年轻代，而年老代存放生命周期长久的实例对象。

# Java设计原则(S.O.L.I.D)

详情：http://ucoon.tech/2018/08/07/%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99%E5%92%8C%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E5%B0%8F%E8%AE%B0/

# Java设计模式

1. 单例模式