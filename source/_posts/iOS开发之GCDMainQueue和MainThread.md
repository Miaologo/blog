---
title: iOS开发之GCDMainQueue和MainThread
date: 2019-08-15 20:58:25
tags:
	- iOS 
	- main thread
---

# GCD's Main Queue vs  Main Thread （译文）

> [GCD's Main Queue vs. Main Thread] http://blog.benjamin-encz.de/post/main-queue-vs-main-thread/
>
> [关于 dispatch_main_async_safe]https://www.jianshu.com/p/bdbc7d6618a3



----



## 结论

原文这样说道：

> Calling an API from a non-main queue that is executing on the main thread will lead to issues if the library (like VektorKit) relies on checking for execution on the main queue.

意思大概是如果在**主线程**执行**非主队列**调度的API，而这个API需要检查是否由**主队列上**调度，那么将会出现问题。

SDWebImage 就是从判断**是否在主线程执行**改为判断**是否由主队列上调度**。而由于**主队列是一个串行队列**，无论任务是异步同步都不会开辟新线程，所以**当前队列是主队列等价于当前在主线程上执行**。可以这样说，**在主队列调度的任务肯定在主线程执行，而在主线程执行的任务不一定是由主队列调度的**

-----

在苹果的开发者日常工作中，常常会遇到一个问题，如果正确保证代码运行在主线程或者主队列中。在ReativeCocoa 和 MapKit的这周issue中又遇到了

## 问题



