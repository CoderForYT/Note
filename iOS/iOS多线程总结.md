# iOS多线程总结

## 目的

本文将从一下几个各方面分享多线程的的相关内容：

1. 多线程的基本概念
2. 多线程的状态和生命周期
3. iOS提供的四种解决方案：pthread, NSThread, GCD, NSOperation
4. NSThread 的使用
5. CGD的理解和使用
6. NSOperation的理解和使用
7. GCD和NSOperation对比
8. 线程安全

## 1. 多线程的基本概念

​	我们都知道，进程是运转中的程序，是为了在CPU上实现多道线程而发明的概念，但是进程在同一时间只能处理一件事，如果我们需要同时处理两件或者两件以上的事情，就需要使用到线程。

### 1.1 进程的概念 

​	进程是一个具有一定独立功能的程序关于某个数据集合的一次运行活动，它是操作系统动态执行的基本单元。在传统的操作系统中，进程即使基本的分配单元，也是基本的执行单元。

​	进程的概念有两点：第一，进程是一个实体，每个进程有自己的地址空间，包括文本区域（text region），数据区域（data region），堆栈。第二，进程是一个“执行中的的程序”。程序是没有生命的实体，只有处理器赋予程序生命时（执行），它才能成为一个活动的尸体，称之为进程。

​	在iOS中，可以认为一个App都有一个对应的进程，App并不能操作进程。

### 1.2 线程的概念

​	每个正在系统运行的程序都是一个进程，每个进程包含一到多个线程。进程也可能是整个程序或者部分程序的动态执行。线程是一组指令的集合，或者是程序的特殊段。它能在程序里独立执行。也可以把它理解为代码运行的上下文。所以基本上是轻量级的进程，他负责在单个程序里执行多个程序。通常有操作系统负责多个线程的调度和执行。

### 1.3 多线程的概念

​	多线程是指从软件或者硬件上实现多个线程并发执行的技术。多线程是为了完成同步完成多项任务，不是为了提高运行效率。多线程是通过提高资源使用率来提高系统的总体性能。

## 2. 多线程的状态和生命周期

​	下图是多线程状态示意图。线程的生命周期包括：新建，就绪，运行，阻塞，死亡。![多线程的状态](http://cc.cocimg.com/api/uploads/20170707/1499394752139363.png)

* 新建（NEW）：实例化线程对象
* 就绪（Runnable）：向线程对象发送start命令，线程多项被加入到可调度的线程池，等待CPU调度。
* 运行（Running）：CPU负责调度可调度线程池中线程的执行。线程执行完成前，状态可能会在就绪和运行之间来回切换。就绪和运行之间的状态变化由CPU负责。不能进行干预。
* 阻塞：（Blocked）：当满足某个预订条件时，可以使用休眠或者锁，阻塞线程执行。sleepForTimeInterval（休眠指定时长），sleepUntileDate（休眠到指定日期），@synchronized（self）（互斥锁）。
* 死亡：正常死亡：线程执行完毕。非正常死亡，当满足某个条件后，在线程内部中止执行，或者在主线程终止。

## 3. iOS提供的四种解决方案：pthread, NSThread, GCD, NSOperation

​	iOS提供了四种解决方案，分别是：pthread, NSThread, GCD, NSOperation。

​	下面的图是对四种方案的总结。

![iOS提供的多线程解决方案](http://cc.cocimg.com/api/uploads/20170707/1499394732413995.png)

## 4. NSThread 的使用(API)

### 4.1 线程创建方式

```
- init
- initWithTarget:selector:object:
```

### 4.2 开始执行线程

```objective-c
+ detachNewThreadSelector:toTarget:withObject:
- start
- main
```

### 4.3 停止, 阻塞线程

```objective-c
+ sleepUntilDate:
+ sleepForTimeInterval:
+ exit
- cancel
```

### 4.4 状态

```objective-c
executing
finished
cancelled
```

### 4.5 主线程相关

```objective-c
isMainThread
mainThread
```

## 5. CGD的理解和使用

​	GCD, 全称Grand Central Dispatch，是苹果为多核心处理器开发的一套异步调度机制。苹果官方文档中是这样介绍它的：

```
Grand Central Dispatch (GCD) comprises language features, runtime libraries, and system enhancements that provide systemic, comprehensive improvements to the support for concurrent code execution on multicore hardware in iOS and OS X.
```

​	GCD能够合理地利用更多的CPU内核，最重要的是它会自动管理线程的生命周期（创建线程，调度任务，销毁线程），完全不需要我们进行管理了，加上虽然使用的是C语言，但是使用了大部分使用了Block，使用起来更加方便，减轻了开发者多线程开发的难度。

### 5.1 两个基本概念

​	在GCD中有两个非常重要的概念：任务和队列。这也是GCD的核心。

#### * 任务

​	任务值得是操作的意思。换句话就是你需要在线程执行的代码。在GCD中，任务是放在block中的。

​	执行任务有两种方式：同步（sync）和异步（async）。**同步和异步的主要区别在于是否阻塞当前线程。**

 * 同步执行（sync）：
    * 在当前线程中执行任务，不具备开启新线程的能力。
    * 同步添加到指定的队列中，在新添加的任务执行完成之前，会一直阻塞（等待），直到队列中的任务完成之后，才会继续执行
* 异步执行（async）：
  * 另开一条线程执行任务。
  * **具备开启新线程的能力**，会在新的线程中执行新的任务，当前线程并不会受到任何影响。
  * 异步执行虽然具备开启线程的能力，但是并不一定开启新线程，这取决于**队列**的类型。

#### * 队列

​	这里的队列指的执行任务的等待队列，即用来存放任务的队列。队列是一种特殊的线性表，采用先进先出（FIFO）的原则。即新任务总是被插入到队列的末尾，而读取任务的时候总是从队列的头部开始读取。GCD有两种类型的队列：串行队列和并发队列。**两者最主要的区别是：执行顺序不同，以及开启的线程数不同。**

 * 串行队列
    * 每次只有一个任务被执行。只有任务处理结束，才会从队列中移除。只会用到一个线程。
 * 并发队列
   * 不需要等待任务处理结束，所以可以同时执行多个任务。但是并行执行的处理数量取决于当前系统的状态。所谓并行执行，就是使用多个线程同时执行多个任务。

#### * 总结

​	综合以上任务和队列的组合，加上主线程是特殊的串行队列，可以总结出一下的表

|       区别        |    并发队列（concurrent）    |                      串行队列（serial）                      |                            主队列                            |
| :---------------: | :--------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 同步任务（sync）  | 没有开启新线程，串行执行任务 | 当前线程调用：**死锁**     <br /> 其他线程调用：没有开启新线程，串行执行任务 | 主线程调用：**死锁**   <br />其他线程调用：没有开启新线程，串行执行任务 |
| 异步任务（async） |  有开启新线程，并发执行队列  |               有开启新线程(1条)，串行执行任务                |                 没有开启新线程，串行执行任务                 |

​      PS：Q： 为什么在主队列或者在当前线程调用串行同步队列会引起死锁？ 

​		A：原因是同步任务的特点是：添加一个新的同步任务，会先阻塞当前线程，等待新添加的任务执行完成。 而串行队列的特点是：必须等到先添加的任务执行完成，才会执行新添加的任务。如果在主队列，或者当前线程调用串行同步队列，就会造成原有的任务等待新添加的任务完成，而新添加的任务等待原有任务的完成，造成相互等待，形成死锁。

### 5.2 GCD队列的使用

GCD 的使用步骤其实很简单，只有两步。

	1.  创建或者获取一个队列(串行对类或者并行队列)
	2.  将任务追加到任务的等待队列中，然后系统就会根据任务类型执行任务（同步执行或异步执行）

#### 5.2.1 队列的创建/获取方法

创建队列的API:

```c
// 队列创建
dispatch_queue_t dispatch_queue_create(const char *label, dispatch_queue_attr_t attr);
// 参数
    const char *label：队列的唯一标识，推荐使用 com.apple.www这种格式，可以为空。
    dispatch_queue_attr_t attr ：队列类型，两个宏。
        * DISPATCH_QUEUE_SERIAL (or NULL） 串行队列
        * DISPATCH_QUEUE_CONCURRENT 并发队列
```

```
Discussion
Blocks submitted to a serial queue are executed one at a time in FIFO order. Note, however, that blocks submitted to independent queues may be executed concurrently with respect to each other. Blocks submitted to a concurrent queue are dequeued in FIFO order but may run concurrently if resources are available to do so.

If your app isn’t using ARC, you should call dispatch_release on a dispatch queue when your application no longer needs the dispatch queue, it should release it with the dispatch_release function. Any pending blocks submitted to a queue hold a reference to that queue, so the queue is not deallocated until all pending blocks have completed.

只有提交到串行队列的block函数出队列时才会FIFO，而对于独立的队列可能会并行执行。提交到并行队列的block出队时也是FIFO，但是在执行的过程中可能会根据系统的资源来并行执行。

如果你你是用MRC，当你的应用不在需要队列时，可以使用dispatch_release来释放，但如果队列中又任何pending状态的block，此时是无法dealloc的直到pending block完成。
```

代码：

```c
      // 串行队列
      dispatch_queue_t queue_serial = dispatch_queue_create("com.app.serial", DISPATCH_QUEUE_SERIAL);
      // 串行队列
      dispatch_queue_t queue_null = dispatch_queue_create("com.app.null", NULL);
      // 并发队列
      dispatch_queue_t queue_concurrent = dispatch_queue_create("com.app.concurrent", DISPATCH_QUEUE_CONCURRENT);
```

获取主队列和全局并发队列的API

```c
// 获取主队列
dispatch_queue_t dispatch_get_main_queue(void);
// 获取全局并发队列
dispatch_queue_t dispatch_get_global_queue(long identifier, unsigned long flags);
// 参数：
	long identifier: 队列的优先级,包含4个宏: 
		iOS8和Mac OS 10.10之后可以使用： 
            * QOS_CLASS_USER_INTERACTIVE
            * QOS_CLASS_USER_INITIATED 
            * QOS_CLASS_UTILITY
            * QOS_CLASS_BACKGROUND
		iOS8和Mac OS 10.10以前可以使用: 
            * DISPATCH_QUEUE_PRIORITY_HIGH
            * DISPATCH_QUEUE_PRIORITY_DEFAULT 
            * DISPATCH_QUEUE_PRIORITY_LOW
            * DISPATCH_QUEUE_PRIORITY_BACKGROUND
	unsigned long flags：是苹果预留的参数，暂时永远设置为0
```

#### 5.2.2 任务的创建

```c
// 同步任务的创建
void dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);
// 异步任务的创建
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
// 参数： 
	dispatch_queue_t queue: 队列
	dispatch_block_t block: 任务代码
```

#### 5.2.3 线程间的通信

​	在iOS中经常需要进行线程中的通信, 我们一般在主线程里处理UI刷新, 例如: 点击, 滚动, 拖拽等事件.  我们通常会把耗时操作放在异步线程, 比如图片下载, 网络请求, 解压缩等. 当我们完成操作以后, 需要吧结果展示到UI界面上,  就必须回到主线程, 那么就会用到线程间的通讯.  使用GCD, 可以通过block的嵌套完成通信

```objective-c
/**
 * 线程间通信
 */
- (void)communication {
    // 获取全局并发队列
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); 
    // 获取主队列
    dispatch_queue_t mainQueue = dispatch_get_main_queue(); 
    dispatch_async(queue, ^{
        // 异步追加任务
        for (int i = 0; i < 2; ++i) {
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        }
        // 回到主线程
        dispatch_async(mainQueue, ^{
            // 追加在主线程中执行的任务
            [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
            NSLog(@"2---%@",[NSThread currentThread]);      // 打印当前线程
        });
    });
}
```

输出结果

```
输出结果：
2018-03-20  demo[10717:2441148] 1---<NSThread: 0x60000046f340>{number = 3, name = (null)}
2018-03-20  demo[10717:2441148] 1---<NSThread: 0x60000046f340>{number = 3, name = (null)}
2018-03-20  demo[10717:2440012] 2---<NSThread: 0x60000007da40>{number = 1, name = main}
```

### 5.3 GCD的队列组: (dispatch_group)

​	有时候我们会有这样的需求：分别异步执行2个耗时任务，然后当2个耗时任务都执行完毕后再回到主线程执行任务。这时候我们可以用到 GCD 的队列组。











## 6. NSOperation的理解和使用

## 7. GCD和NSOperation对比

## 8. 线程安全

## 9. 参考来源

1. 进程： https://baike.baidu.com/item/进程/382503
2. 多线程： https://baike.baidu.com/item/多线程/1190404
3. 关于iOS多线程，你看我就够了： https://www.jianshu.com/p/0b0d9b1f1f19