本着实用主义的精神，字里行间绝对不废话，句句都是干货！

首先，Apple不建议大家直接使用Threads来手工管理线程。因为太TM复杂啦！Apple推荐大家使用下面[三种技术](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ConcurrencyandApplicationDesign/ConcurrencyandApplicationDesign.html#//apple_ref/doc/uid/TP40008091-CH100-SW1)来讲进行多线程编程: 

1. Dispatch Queues(GCD)
2. Operation Queues(可看作是GCD的ObjC版)
3. Dispatch Sources(基本用不上)


#GCD编程的思考策略
##定义你的任务: Block Task
把你的任务定义在block中。注意block内存管理和循环引用的问题。
##选择队列: 串行/并发
1. 主队列: `dispatch_get_main_queue();` 默认是串行的。
2. 全局队列: `dispatch_get_global_queue(long identifier, unsigned long flags);` 默认是并发的。
3. 自定义队列： 一般是手工创建的串行队列，很少用

怎么选择呢？很简单。一些耗时的非UI更新操作，比如数据库查询、I/O、通过网络获取数据资源等，请使用全局队列。更新UI的操作则必须放在主队列执行。

>再科普一下
>
>串行队列的任务总是按顺序执行的。当两个任务修改同一块数据时，那你必须要用串行(serial)。比如: 任务A和任务B都需要修改tableA的表结构，那肯定得一个任务执行完毕后，再执行下一个任务啊，总不能任务A和任务B同时修改tableA的表结构吧。
>
>当两个任务不会互相干扰时，你可以使用并行(concurrent)，最大限度地利用CPU核数提高程序性能。比如: 用户在使用TableView查看微博时，同样可以执行小视频的下载任务。此时下载任务放在非主队列执行。这时"浏览微博"和"下载任务"是互不干扰的，所以可以用并行。
>
>简单来说，串行/并行描述的是多个任务之间的执行顺序(是否按顺序执行)。

##选择执行方式: 同步/异步
如果使用同步执行方式，则不会开启新的线程，此时队列中的所有任务都在一个线程中执行，如果遇到非常耗时的任务，就会阻塞当前线程。

如果使用异步执行方式则会开启新线程，此时队列中的任务会在新的线程中执行，这样就不会阻塞源线程。最经典的例子就是更新UI时，必须对主队列使用异步asyn方式，因为如果使用同步方式，则会造成UI界面暂时无法响应用户输入。

#我的理解 不一定对
GCD中不再强调线程的概念。而是让你使用队列。

远古时期的多线程管理操作方式是: 创建一个线程，然后把任务封装到线程中。并用delegate的方式设置线程生命周期中的处理代码。这时你得解决"死锁"的问题，也就是多个任务同时修改一份数据。

但在GCD中，你就不用关心线程创建啦，你只需要把任务封装到队列中。我猜测：如果是串行队列，则GCD默认只在内部创建一个线程，毕竟你的任务都是按顺序执行嘛。如果是并发队列，则默认创建N个线程，毕竟你可能需要同时执行多个任务嘛。

再来看下同步和异步。同步说实话我也不知道有啥用。但异步就真心有用。基本上GCD的队列都是异步执行的。比如全局队列，如果是异步执行，则新加入队列的任务就会被放到新的线程执行，效率就更高啦(当然前提你的硬件够牛逼)。如果是同步执行，则新加入的任务就会卡住其所在的线程，造成线程阻塞。

所以说，只要可以，尽量选择异步执行。

>串行和并行关注的是任务是**"按顺序执行"还是"并发执行"**。
>
>同步和异步关注的是**"任务是否立即执行(是否会造成阻塞)"**。

##Operation Queues
>官网定义: 
>
>Cocoa operations are an object-oriented way to encapsulate work that you want to perform **asynchronously**.

看见没！看见没！看见asynchronously那个单词没！operation queues肯定是异步执行的！同步真的没什么鬼用。


#GCD读书笔记(日本大神的书)
dispatch queue的执行顺序遵循first-in-first-out(FIFO)。 Tasks are executed in the added order.

##串行队列
第一个task执行完毕后，才执行下一个task。

##并行队列
所有任务按顺序开始执行，但不会等前一个任务执行完毕后才开始执行。注意哦，并不是队列里有多少个task就会开多少个线程。比如书中的例子，有7个task，但底层只开了4个线程。

##创建队列的两种方式

```objc

//第一种方式：用create方法
dispatch_queue_t mySerialDispatchQueue = dispatch_queue_create("com.example.gcd.MySerialDispatchQueue", NULL);

dispatch_queue_t myConcurrentDispatchQueue = dispatch_queue_create( "com.example.gcd.MyConcurrentDispatchQueue", DISPATCH_QUEUE_CONCURRENT);

//使用create方法一定要自己release
dispatch_release(mySerialDispatchQueue);
dispatch_release(myConcurrentDispatchQueue);

//第二种方式：获取全局的queue
dispatch_queue_t mainDispatchQueue = dispatch_get_main_queue();

dispatch_queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);dispatch_queue_t globalDispatchQueueDefault = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_queue_t globalDispatchQueueLow = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0);dispatch_queue_t globalDispatchQueueBackground = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);

```

##dispatch_after
N秒后把task add到queue中。

```objc

//added the Block to the main dispatch queue by the dispatch_async function three seconds later.

dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 3ull * NSEC_PER_SEC); dispatch_after(time, dispatch_get_main_queue(), ^{	NSLog(@"waited at least three seconds."); 
});
```






# 博客编写目录
* 概述：推荐使用GCD。GCD的三大步骤:把任务封装到block中，选择队列，选择控制队列的方法
* 获取队列的方式：先进行队列概述
	* 自己create队列
	* 使用内置队列

* 同步sync和异步async
* 死锁的例子
* 其他控制队列的方法

#reference
* [Concurrency Programming Guide](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ConcurrencyandApplicationDesign/ConcurrencyandApplicationDesign.html)
* [GCD概述、语法以及好的示例](https://github.com/hehonghui/iOS-tech-frontier/blob/master/issue-2/GCD%E6%A6%82%E8%BF%B0%E3%80%81%E8%AF%AD%E6%B3%95%E4%BB%A5%E5%8F%8A%E5%A5%BD%E7%9A%84%E7%A4%BA%E4%BE%8B.md?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
* [GCD 深入理解（一）](http://www.cocoachina.com/industry/20140428/8248.html)
* [GCD 深入理解（二）](http://www.cocoachina.com/industry/20140515/8433.html)
* [iOS开发多线程篇—GCD介绍](http://www.cnblogs.com/wendingding/p/3806821.html)
* [一篇专题让你秒懂GCD死锁问题!](http://my.oschina.net/doxing/blog/618132?fromerr=Z0GhpInR)
* [iOS并发编程](https://github.com/ming1016/study/wiki/iOS%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B)
* [细说GCD（Grand Central Dispatch）如何用](https://github.com/ming1016/study/wiki/%E7%BB%86%E8%AF%B4GCD%EF%BC%88Grand-Central-Dispatch%EF%BC%89%E5%A6%82%E4%BD%95%E7%94%A8?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
* [GCD 最佳实践指南](http://www.cocoachina.com/ios/20160511/16221.html)
* [Parse源码浅析系列（一）---Parse的底层多线程处理思路：GCD高级用法](https://github.com/ChenYilong/ParseSourceCodeStudy/blob/master/01_Parse%E7%9A%84%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%A4%84%E7%90%86%E6%80%9D%E8%B7%AF/Parse%E7%9A%84%E5%BA%95%E5%B1%82%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%A4%84%E7%90%86%E6%80%9D%E8%B7%AF.md?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)



