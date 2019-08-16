---
title: iOS开发之锁
date: 2019-08-08 20:50:00
tags:
	- iOS
	- 锁
	- 多线程
---

获取锁失败，线程有两种状态：

- 空转状态：线程执行空任务循环等待，当锁可用时立即获取锁。
- 挂起状态：线程挂起，当锁可用时需要其他线程唤醒。

唤醒线程比较耗时，线程空转需要消耗 CPU 资源并且时间越长消耗越多，由此可知空转适合少量任务、挂起适合大量任务。

实际上互斥锁和读写锁都有空转锁的特性，它们在获取锁失败时会先空转一段时间，然后才会挂起，而空转锁也不会永远的空转，在特定的空转时间过后仍然会挂起，所以通常情况下不用刻意去使用空转锁



## @synchronized(obj)



## NSLock



## semaphore

**dispatch_semaphore_signal**



## NSRecursiveLock





## NSConditionLock



## NSCondition



## pthread_mutex

**pthread_mutex_init(pthread_mutex_t \* mutex,const pthread_mutexattr_t attr);**初始化锁变量mutex。attr为锁属性，NULL值为默认属性。

**pthread_mutex_lock(pthread_mutex_t*mutex);**加锁

**pthread_mutex_tylock(pthread_mutex_t*mutex);**加锁，但是与2不一样的是当锁已经在使用的时候，返回为EBUSY，而不是挂起等待。

**pthread_mutex_unlock(pthread_mutex_t*mutex);**释放锁

**pthread_mutex_destroy(pthread_mutex_t**mutex);**使用完后释放






## OSSpinLock



**OSSpinLock**自旋锁，性能最高的锁。原理很简单，就是一直 **do while**忙等。它的缺点是当等待时会消耗大量 CPU 资源，所以它不适用于较长时间的任务。 不过最近YY大神在自己的博客[不再安全的 OSSpinLock](https://link.juejin.im/?target=https%3A%2F%2Fblog.ibireme.com%2F2016%2F01%2F16%2Fspinlock_is_unsafe_in_ios%2F)中说明了**OSSpinLock**已经不再安全，请大家谨慎使用





作者：聪莞链接：https://juejin.im/post/5d22f34de51d4555fd20a3c5来源：掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。