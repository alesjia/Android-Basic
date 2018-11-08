# Java基础知识
# Java Serializable
[Java 序列化的高级认识](https://www.ibm.com/developerworks/cn/java/j-lo-serial/index.html)

学习总结：
1. Java虚拟机序列化反序列化允许的条件：
* 实现serializable接口
* 两端的类路径和功能代码一致
* 两个类的序列化id一致 serialVersionUID
3. 静态变量并不会被序列化
4. 子类实现了serializable接口，但父类没有实现serializable接口，则父类的变量值为无参数构造函数设置的值。这是因为一个java对象的构造必须有父对象，才有子对象。
5. Transient关键字的作用是控制变量的序列化
6. 序列化的时候，会默认先调用类的writeObject方法（用户自定义），如果没有，再调用ObjectOutputStream的defaultWriteObject方法。
7. 如果调用了将序列化对象连续写入两次，实际上并不会保存两份，只会创建一个引用。

# Java类加载器
[深入探讨 Java 类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html)

1. 类加载器的树状结构
引导类加载器
扩展类加载器
系统类加载器
2. 加载类的过程
启动类加载的类加载器和完成类加载的类加载器不一定相同
3. 线程上下文类加载器
SPI的接口是Java核心库的一部分，由引导类加载器加载
在SPI接口的代码中使用线程上下文类加载器，就可以成功的加载到SPI实现类
5. 开发自己的类加载器
开发自己的类加载器一般只需覆写findClass(String name)，最好不要覆写loadClass(String name)，因为这样会影响类加载器的代理模式
6. 类加载器与Web容器
Web容器的代理模式跟Java类的代理模式相反
Web容器会保证Java核心库的类型安全
8. 类加载器与OSGI

# JAVA的equals方法
只有一个类是用来表示值时，并且父类没有实现equals方法，那么这个时候需要实现equals方法
* 自反性：x.equals(x)
* 对称性：if x.equals(y)  then y.equals(x)
* 传递性：if x.equals(y),y.equals(z) then x.equals(z)
* 一致性：对于多次调用x.equals(y), 返回的结果是一致的
* 对于每一个不是空的对象，x.equals(null)返回false

# Java的hashcode方法
* 对于同一个的对象，每次调用hashcode方法返回的值需要一致
* 对于相同（x.equals(y)）的对象，他们的hashcode返回返回的值需要一致
* 对于不同的对象，最好能够做到hashcode尽可能不一样（还是可以一样的）

# String是immutable的
* String存储在final char[]中，所以String的内容不能更改
* 常量字符串是存放在方法区（内存的一部分）中的，对于多个相同的 String常量，实际上它们都是同一个对象，它们都指向方法区中的这个常量对象
* String的hashcode是缓存起来的，Map<String, Object>对于这个对象来说，如果key是相同的，那么只需要计算一次hashcode，从而提高访问效率
* new String(“aa”)这个对象是存放在堆中的，如果想把“aa”存放在方法区中，那么就使用new String(“aa”).intern()方法。实际上这个时候会存在两个对象，一个放在堆中，一个放在方法区中

# HashMap
* HashMap底层保存在一个Node数组里，默认数组大小为16
* Node是一个链表，包括hash，key，value以及next node
* 把数据存入Map时，先进行hash计算，然后计算出index（就是数组的index），将数据放入对应的数组里面，如果数组中已经存在数据，则需要放在链表的后面
* 取出Map数据的时候也一样，先计算hash值，然后计算出index。通过index来访问node数组，然后根据hashcode和key的值来判断，如果hashcode和index值一致的话，那么就取出对应的数据。
* DEFAULT_LOAD_FACTOR=0.75，当超过12（16capicity*0.75）时，这个时候需要将数组扩容为原来容量的2倍。
* TREEIFY_THRESHOLD =8 当一个桶的数据超过8，将会把linked List变为平衡二叉树
# ConcurrentHashmap
[Map 综述（三）：彻头彻尾理解 ConcurrentHashMap - CSDN博客](https://blog.csdn.net/justloveyou_/article/details/72783008)
* 通过锁分段技术保证并发环境下的写操作
* 通过HashEntry的不变性，volatile变量的内存可见性和加锁重读机制保证高效，安全的读操作（在HashEntry类中，key，hash和next域都被声明为final的，value域被volatile所修饰，因此HashEntry对象几乎是不可变的，这是ConcurrentHashmap读操作并不需要加锁的一个重要原因。）
* 通过不加锁和加锁两种方案控制跨段操作的安全性
* 与Hashmap不同的是，桶中的链表Hashmap是按顺序插入的(->A->B->C)，而ConcurrentHashmap则是相反的顺序（->C->B->A），其原因是ConcurrentHashmap中的HashEntry的next字段被申明为final。
* 本质上，ConcurrentHashMap就是一个Segment数组，而一个Segment实例则是一个小的哈希表。
* ConcurrentHashMap不同于HashMap，它既不允许key值为null，也不允许value值为null。

# ArrayList和LinkedList
* ArrayList数据保存在Object[]数组中，而LinkedList则保存在双向链表中，Node first，Node last
* 如果需要比较频繁的插入和删除操作，那么选择用LinkedList
* 如果只是用来保存还有获取数据，那么使用ArrayList

# Interface和abstract class的区别
[Difference between Interface and Absract Class - YouTube](https://www.youtube.com/watch?v=nNwZN3mOVcw)
[TOP 6 difference between Abstract class and Interface in JAVA || Must know facts - YouTube](https://www.youtube.com/watch?v=Ud0zfImgbYw)
什么时候用Interface，什么时候用abstract class？
如果你目前还不知道如何实现，只知道协议，那么直接用接口；如果你知道部分实现，那么就用abstract class
* Interface只定义方法，没有实现，而abstract class则可以包括代码实现
* Interface的方法是public abstract， 而abstract class则不一定
* Interface定义的属性只能是public static final的，而abstract class则没有这个限制
* 一个类可以implements多个接口，但是只能extends一个abstract类
* Interface不能含有构造函数，而abstract class则含有构造函数
* Interface定义的属性需要初始化，而abstract class则不一定需要
* Interface不能定义instance和static块，而abstract class则可以
* 
# Volatile
[Java 并发：volatile 关键字解析 - CSDN博客](https://blog.csdn.net/justloveyou_/article/details/53672005)
1.为了解决缓存不一致问题，引入
* 通过在 总线加 LOCK# 锁 的方式 （在软件层面，效果等价于使用 synchronized 关键字）；
* 通过 缓存一致性协议 （在软件层面，效果等价于使用 volatile 关键字）
2.并发编程三个问题
* 原子性问题
* 可见性问题
* 有序性问题
指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。也就是说，要想使并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。

# JAVA锁
[Java并发性和多线程介绍目录 | 并发编程网 – ifeve.com](http://ifeve.com/java-concurrency-thread-directory/)
[Java Memory Model](http://tutorials.jenkov.com/java-concurrency/java-memory-model.html)
## Synchronized
## ReEntrantLock
## Semaphore
## 非阻塞算法的好处
* 选择
* 没有死锁
* 没有线程挂起
* 减少线程时延

# Java虚拟机
[专栏：深入理解Java语言 - CSDN博客](https://blog.csdn.net/column/details/zhangjg-java-blog.html)

## 注解
[秒懂，Java 注解 （Annotation）你可以这样学 - CSDN博客](https://blog.csdn.net/briblue/article/details/73824058)
注解如同类，接口一样，是一种类型
### 1.元注解
什么是元注解，就是可以注解到注解上面的注解，或者说元注解就是一种基本注解，它可以应用在其他注解之上
@Retention、@Documented、@Target、@Inherited、@Repeatable
### 2.注解的属性
注解的属性也叫成员变量，注解只有成员变量，没有方法。
需要注意的是，在注解中定义属性时，它的类型必须是8种基本类型外加类，接口，注解以及他们的数组
### 3.Java预置的注解
### 4.注解的提取
要想提取注解，需要使用反射，这个是针对运行时注解来说的
### 5.注解的使用场景
注解给编译器和apt使用（annotation processing tool）
## 动态代理
[Java 动态代理机制分析及扩展，第 1 部分](https://www.ibm.com/developerworks/cn/java/j-lo-proxy1/index.html)
代理是一种设计模式，其目的是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息以及转发消息，以及进行消息被委托类执行后的后续处理。
动态代理就是代理类是由Proxy这个类自动生成的。
Proxy类，接口是动态代理类的子类实现。

