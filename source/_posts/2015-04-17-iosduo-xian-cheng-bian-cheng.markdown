---
layout: post
title: "IOS多线程编程"
date: 2015-03-08 13:48:54 +0800
comments: true
categories: IOS探索

---

IOS多线程编程对于初学者来说，总是会觉得很难理解和掌握，下面就为大家更加系统全面的介绍IOS多线程编程，希望对大家有所帮助。

<!--more-->

###一.首先简单介绍什么叫线程

1.可并发执行的，拥有最小系统资源，共享进程资源的基本调度单位。

2.共用堆，自有栈(官方资料说明iOS主线程栈大小为1M，其它线程为512K)。

3.并发执行进度不可控，对非原子操作易造成状态不一致，加锁控制又有死锁的风险。

###二.IOS中的线程

1.IOS主线程(UI线程)，我们的大部分业务逻辑代码运行于主线程中。

2.没有特殊需求，不应引入线程增加程序复杂度。

3.应用场景:逻辑执行时间过长，严重影响交互体验(界面卡死)等。

4.IOS支持三个层次的线程编程,从底层到高层(层次越高使用越方便,越简洁)分别是:

1).NSThread

2).NSOperation

3).GCD(全称 Grand Central Dispatch)

###三.下面分别介绍这三种线程编程方法

1.NSThread:

优点：NSThread 比其他两个轻量级
缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销

NSThread实现的技术有下面三种：Cocoa threads、POSIX threads和Multiprocessing Services，一般使用Cocoa threads技术。

2.NSOperation:

优点：不需要关心线程管理，数据同步的事情，可以把精力放在自己需要执行的操作上。
Cocoa operation 相关的类是 NSOperation ，NSOperationQueue。
NSOperation是个抽象类，使用它必须用它的子类，可以实现它或者使用它定义好的两个子类：NSInvocationOperation 和 NSBlockOperation。
创建NSOperation子类的对象，把对象添加到NSOperationQueue队列里执行。

使用 NSOperation的方式有两种，一种是用定义好的两个子类：NSInvocationOperation 和 NSBlockOperation；另一种是继承NSOperation。

3.GCD:Grand Central Dispatch 简称（GCD）是苹果公司开发的技术，以优化的应用程序支持多核心处理器和其他的对称多处理系统的系统。这建立在任务并行执行的线程池模式的基础上的。它首次发布在Mac OS X 10.6 ，iOS 4及以上也可用。

GCD的工作原理是：让程序平行排队的特定任务，根据可用的处理资源，安排他们在任何可用的处理器核心上执行任务。
 
一个任务可以是一个函数(function)或者是一个block。 GCD的底层依然是用线程实现，不过这样可以让程序员不用关注实现的细节。
 
GCD中的FIFO队列称为dispatch queue，它可以保证先进来的任务先得到执行。

dispatch queue分成以下三种：

1).运行在主线程的Main queue，通过dispatch_get_main_queue获取.可以看出，dispatch_get_main_queue也是一种dispatch_queue_t。

2).并行队列global dispatch queue，通过dispatch_get_global_queue获取，由系统创建三个不同优先级的dispatch queue。并行队列的执行顺序与其加入队列的顺序相同。

3).串行队列serial queues一般用于按顺序同步访问，可创建任意数量的串行队列，各个串行队列之间是并发的。

当想要任务按照某一个特定的顺序执行时，串行队列是很有用的。串行队列在同一个时间只执行一个任务。我们可以使用串行队列代替锁去保护共享的数据。和锁不同，一个串行队列可以保证任务在一个可预知的顺序下执行。

serial queues通过dispatch_queue_create创建，可以使用函数dispatch_retain和dispatch_release去增加或者减少引用计数。

GCD的用法：

```
//  后台执行：
 dispatch_async(dispatch_get_global_queue(0, 0), ^{
      // something
 });

 // 主线程执行：
 dispatch_async(dispatch_get_main_queue(), ^{
      // something
 });

 // 一次性执行：
 static dispatch_once_t onceToken;
 dispatch_once(&onceToken, ^{
     // code to be executed once
 });

 // 延迟2秒执行：
 double delayInSeconds = 2.0;
 dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, delayInSeconds * NSEC_PER_SEC);
 dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
     // code to be executed on the main queue after delay
 });

 // 自定义dispatch_queue_t
 dispatch_queue_t urls_queue = dispatch_queue_create("blog.devtang.com", NULL);
 dispatch_async(urls_queue, ^{  
　 　// your code 
 });
 dispatch_release(urls_queue);

 // 合并汇总结果
 dispatch_group_t group = dispatch_group_create();
 dispatch_group_async(group, dispatch_get_global_queue(0,0), ^{
      // 并行执行的线程一
 });
 dispatch_group_async(group, dispatch_get_global_queue(0,0), ^{
      // 并行执行的线程二
 });
 dispatch_group_notify(group, dispatch_get_global_queue(0,0), ^{
      // 汇总结果
 });
 ```
 一个应用GCD的例子:
 
 ```
 dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{   
    NSURL * url = [NSURL URLWithString:@"http://avatar.csdn.net/2/C/D/1_totogo2010.jpg"];   
    NSData * data = [[NSData alloc]initWithContentsOfURL:url];   
    UIImage *image = [[UIImage alloc]initWithData:data];   
    if (data != nil) {   
        dispatch_async(dispatch_get_main_queue(), ^{   
            self.imageView.image = image;   
         });   
    }   
});
```

GCD的另一个用处是可以让程序在后台较长久的运行。

在没有使用GCD时，当app被按home键退出后，app仅有最多5秒钟的时候做一些保存或清理资源的工作。但是在使用GCD后，app最多有10分钟的时间在后台长久运行。这个时间可以用来做清理本地缓存，发送统计数据等工作。

让程序在后台长久运行的示例代码如下：

```
// AppDelegate.h文件
@property (assign, nonatomic) UIBackgroundTaskIdentifier backgroundUpdateTask;

// AppDelegate.m文件
- (void)applicationDidEnterBackground:(UIApplication *)application
{
    [self beingBackgroundUpdateTask];
    // 在这里加上你需要长久运行的代码
    [self endBackgroundUpdateTask];
}

- (void)beingBackgroundUpdateTask
{
    self.backgroundUpdateTask = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
        [self endBackgroundUpdateTask];
    }];
}

- (void)endBackgroundUpdateTask
{
    [[UIApplication sharedApplication] endBackgroundTask: self.backgroundUpdateTask];
    self.backgroundUpdateTask = UIBackgroundTaskInvalid;
}
```