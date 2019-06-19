---
title: iOS线程生命周期监控
date: 2019-06-18 21:28:26
Tags: ios
---

iOS系统通过Core Services层的Foundation框架提供基于OC语言的NSThread和NSOperationQueue类来实现对线程和线程池的管理和使用。同时也提供了一套基于C语言的GCD线程池函数库来支持多线程的处理应用。这些高级的线程类或者函数的内部实现大部分最终都会调用POSIX标准中的pthread线程库中的pthread_xxx系列函数(#include <pthread.h>)来完成线程的创建、运行、暂停、恢复、销毁、结束等操作。用户态下的线程创建通过系统调用到达内核态的BSD层并创建bsdthread对象，而BSD层则调用Mach层的ksthread对象来完成最终线程的创建和调度的

![image-20190619103815599](/Users/miaoliu/Library/Application Support/typora-user-images/image-20190619103815599.png)

pthread库中除了提供一系列标准的线程操作API外，还提供了一个用于监控线程创建、运行、结束、销毁的函数。这个函数定义在头文件`#include <pthread/introspection.h>`中，具体的头文件路径如下：

![image-20190619103909276](/Users/miaoliu/Library/Application Support/typora-user-images/image-20190619103909276.png)

```c
/*!
 * @header
 *
 * @abstract
 * Introspection API for libpthread.
 *
 * This should only be used for introspection and debugging tools.  Do not rely
 * on it in shipping code.
 */

__BEGIN_DECLS

/*!
 * @typedef pthread_introspection_hook_t
 *
 * @abstract
 * A function pointer called at various points in a PThread's lifetime.  The
 * function must be able to be called in contexts with invalid thread state.
 *
 * @param event
 * One of the events in pthread_introspection_event_t.
 *
 * @param thread
 * pthread_t associated with the event.
 *
 * @param addr
 * Address associated with the event, e.g. stack address.
 *
 * @param size
 * Size associated with the event, e.g. stack size.
 */
typedef void (*pthread_introspection_hook_t)(unsigned int event,
		pthread_t thread, void *addr, size_t size);

/*!
 * @enum pthread_introspection_event_t
 * Events sent by libpthread about threads lifetimes.
 *
 * @const PTHREAD_INTROSPECTION_THREAD_CREATE
 * The specified pthread_t was created, and there will be a paired
 * PTHREAD_INTROSPECTION_THREAD_DESTROY event. However, there may not be
 * a START/TERMINATE pair of events for this pthread_t.
 *
 * Starting with macOS 10.14, and iOS 12, this event is always sent before
 * PTHREAD_INTROSPECTION_THREAD_START is sent. This event is however not sent
 * for the main thread.
 *
 * This event may not be sent from the context of the passed in pthread_t.
 *
 * Note that all properties of this thread may not be functional yet, and it is
 * not permitted to call functions on this thread past observing its address.
 *
 * @const PTHREAD_INTROSPECTION_THREAD_START
 * Thread has started and its stack was allocated. There will be a matching
 * PTHREAD_INTROSPECTION_THREAD_TERMINATE event.
 *
 * This event is always sent from the context of the passed in pthread_t.
 *
 * @const PTHREAD_INTROSPECTION_THREAD_TERMINATE
 * Thread is about to be terminated and stack will be deallocated. This always
 * matches a PTHREAD_INTROSPECTION_THREAD_START event.
 *
 * This event is always sent from the context of the passed in pthread_t.
 *
 * @const PTHREAD_INTROSPECTION_THREAD_DESTROY
 * pthread_t is about to be destroyed. This always matches
 * a PTHREAD_INTROSPECTION_THREAD_CREATE event, but there may not have been
 * a START/TERMINATE pair of events for this pthread_t.
 *
 * This event may not be sent from the context of the passed in pthread_t.
 */
enum {
	PTHREAD_INTROSPECTION_THREAD_CREATE = 1,
	PTHREAD_INTROSPECTION_THREAD_START,
	PTHREAD_INTROSPECTION_THREAD_TERMINATE,
	PTHREAD_INTROSPECTION_THREAD_DESTROY,
};

/*!
 * @function pthread_introspection_hook_install
 *
 * @abstract
 * Install introspection hook function into libpthread.
 *
 * @discussion
 * The caller is responsible for implementing chaining to the hook that was
 * previously installed (if any).
 *
 * @param hook
 * Pointer to hook function.
 *
 * @result
 * Previously installed hook function or NULL.
 */

__API_AVAILABLE(macos(10.9), ios(7.0))
__attribute__((__nonnull__, __warn_unused_result__))
extern pthread_introspection_hook_t
pthread_introspection_hook_install(pthread_introspection_hook_t hook);

__END_DECLS
```



* pthread_introspection_hook_install
	
	函数的作用是安装一个回调函数来挂钩线程生命周期的四个过程。因此函数的入参是一个函数指针，返回的则是老的挂钩函数的指针。回调函数是一个格式为pthread_introspection_hook_t类型的函数，
	

回调函数的每个参数的意义如下：
event：指定线程所处的状态。
thread: 线程的句柄，每个pthread线程都由一个pthread_t类型句柄来唯一标识。
addr: 为线程分配的栈内存的基地址。
size: 为线程分配的栈内存的尺寸。

上面说的每一个线程有创建、运行、终止、销毁四个状态，而event则是用来表示线程的四种状态的值，它的值是如下枚举结构的某一个值：

```
enum {
    PTHREAD_INTROSPECTION_THREAD_CREATE = 1,  //创建
    PTHREAD_INTROSPECTION_THREAD_START,    //运行
    PTHREAD_INTROSPECTION_THREAD_TERMINATE,  //终止
    PTHREAD_INTROSPECTION_THREAD_DESTROY,  //销毁
};
```

需要注意的是在函数中设置回调挂钩函数后只会监控设置之后的所有线程状态的变化。因此如果我们要监控整个应用生命周期的所有线程的状态时，需要尽可能早的进行回调函数的设置，比如可以在某个类的+load方法中，或者在某个全局C++对象的构造函数中设置等等。
回调挂钩函数中的第二个参数thread是一个类型为pthread_t线程句柄对象，这个对象的结构并没有对外公开。但是因为pthread库已经被苹果开源：https://opensource.apple.com/source/libpthread/  。因此我们可以通过线程句柄对象的内部定义来获取关于线程的更多信息。以方便我们能对线程的各种数据进行更加详细的记录。当然这里我们需要考虑到线程句柄的不同版本下的数据成员的问题

一个简单的在main函数内实现线程监控的代码示例:
```
#include <pthread/introspection.h>
#include <pthread.h>
#include <stdlib.h>
#include <stdio.h>

pthread_introspection_hook_t   g_oldpthread_introspection_hook = NULL;

void mypthread_introspection_hook(unsigned int event, pthread_t thread, void *addr, size_t size)
{
   __uint64_t threadid;
    pthread_threadid_np(thread, &threadid);
     
     printf("thread_id = %d,  addr = %p, size = %d\n", threadid, addr, size);
     switch (event)
     {
           case PTHREAD_INTROSPECTION_THREAD_CREATE:
              //dothing ..
              break;
           case PTHREAD_INTROSPECTION_THREAD_START:
             //dothing ..
              break;
           case  PTHREAD_INTROSPECTION_THREAD_TERMINATE:
             //dothing ..
              break;
           case PTHREAD_INTROSPECTION_THREAD_DESTROY:
             //dothing ..
              break;
      }

   //记得在最后或者开头调用老的hook函数
  if (g_oldpthread_introspection_hook != NULL)
    g_oldpthread_introspection_hook(event, thread, addr, size);
}


int main(int argc, char *argv[])
{

   //注册线程监控的回调函数为mypthread_introspection_hook
   g_oldpthread_introspection_hook  = pthread_introspection_hook_install(mypthread_introspection_hook);

   @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

```

>你可以通过开源代码中对pthread_t类型结构体的定义来获取线程的更多信息，但是要注意线程库的版本信息。

线程监控回调函数中的代码应该尽可能的精简和高效，包括官方的头文件中也有一段说明(实际上是可以被appstore审核通过的)：

>This should only be used for introspection and debugging tools. Do not rely
on it in shipping code.



具体参考文章 [iOS线程生命周期的监控](https://www.jianshu.com/p/813ba526204b)

