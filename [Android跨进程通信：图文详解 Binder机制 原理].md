[Android跨进程通信：图文详解 Binder机制 原理]
(https://blog.csdn.net/carson_ho/article/details/73560642)
# Binder通信
## 1.Binder到底是什么
* 机制，模型
定义：Binder是一种Android中实现跨进程通信的一种方式
作用：在Android中实现跨进程通信
* 模型结构
定义：Binder是一种虚拟物理设备驱动
作用：链接service进程，client进程和ServiceManager进程
* Android代码来说
定义：Binder是一个类，实现了IBinder接口
作用：将Binder机制模型以代码的形式具体实现在Android中
## 2.知识储备
Linux系统会将操作系统的内存分成两段，一部分是高地址，分给内核使用，称为内核空间；每个进程有各自的私有用户空间，供各个进程使用，称为用户空间。
2.1传统的进程间通信
* 传统的进程间通信需要两次内存拷贝才能实现跨进程的通信，浪费时间
* 接收数据的缓冲区需要接收方提供，而接收方不知道提供多大空间才能够满足。要么分配足够大的缓冲区，要么通过API获取消息头从而获取消息体的大小。要么浪费空间，要么浪费时间
2.2 Binder进程间通信
* Binder实现了mmap系统调用，主要负责创建数据接收缓冲区以及管理数据接收缓冲区
* 只需要一次数据拷贝，通过内存映射来完成
## 3.binder跨进程通信机制
Binder驱动的作用
* 通过内存映射传递进程间的数据
* 采用Binder线程池，并由Binder驱动自身进行管理
* Binder驱动持有每个service进程在内核空间中的Binder实体，并且给Client进程提供Binder实体的引用
## 4.binder在android中的实现
## 5.binder机制的优点
## 6.总结
不支持RPC
应用的生命周期回调是由AMS通过Binder来调用的
Intents以及ContentProvider底层也是通过Binder来实现的
Messenger底层也是通过Binder来实现的
