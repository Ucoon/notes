1. 项目的**STAR**模型
   - **Situation：简单的项目背景**。比如项目的规模，开发的软件的功能、目标用户等
   - **Task：自己完成的任务**。这个要写详细，用词上要注意区分“参与”和“负责”
   - **Action：为完成任务自己做了哪些工作，是怎么做的。**
   - **Result：自己的贡献。**



2. 面试题2：实现Singleton模式

   **饿汉模式**

   Java版：

   ```java
   public class Singleton{
       private static Singleton instance = new Singleton();
       private Singleton(){}
       public static Singleton getInstance(){
           return instance;
       }
   }
   ```

   优点：获取对象速度快

   缺点： 在类加载时就完成了初始化，所以类加载较慢

   **懒汉模式（线程安全）**

   ```java
   public class Singleton{
       private static Singleton instance;
       private Singleton();
       public static synchronized Singleton getInstance(){
           if(instance == null){
               instance = new Singleton();
           }
           return instance;
       }
   }
   ```

   优点：能在多线程中很好的工作

   缺点：每次调用`getInstance`时都需要进行同步，造成不必要的同步开销

   **懒汉式双重检查模式（DCL）**

   ```java
   public class Singleton{
       private static volatile Singleton instance;
       private Singleton();
       public static Singleton getInstance(){
           if(instance == null){
               synchronized(Singleton.class){
                   if(instance == null){
                       instance = new Singleton();
                   }
               }
           }
           return instance;
       }
   }
   ```

   **静态内部类单例模式**

   ```java
   public class Singleton{
       private Singleton();
       public static Singleton getInstance(){
           return SingletonHolder.instance;
       }
       
       private static class SingletonHolder{
           private static final Singleton instance = new Singleton();
       }
   }
   ```

   第一次加载Singleton类时不会初始化instance，只有第一次调用`getInstance`函数时虚拟机加载`SingletonHolder`并初始化instance，这样不仅能保证线程安全也能保证Singleton类的唯一性。

   Dart版：

   ```dart
   class Singleton{
       Singleton._internal();
       factory Singleton() => _instance;
       ///被标记为late的变量_instance的初始化操作将会延迟到字段首次被访问时执行，而不是类加载时就初始化
       static late final Singleton _instance = Singleton._internal();
   }
   ```

   