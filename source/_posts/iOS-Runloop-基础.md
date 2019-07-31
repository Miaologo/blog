---
title: iOS Runloop 基础
date: 2019-07-29 16:43:48
tags:
	- iOS
	- Runloop
---

# iOS Runloop 基础

>  原文blog [Runloop 基础](https://joakimliu.github.io/2019/03/09/runloop-note/)

本文是对 [Threading Programming Guide - Run Loops](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1) 文档的一个学习(其中参考了 [CFRunLoop](https://opensource.apple.com/source/CF/CF-1153.18/CFRunLoop.c.auto.html) 源码)，其他例子在后期系统了解某个技术有涉及时再说。 其他高质量文章可见

- [深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
- [解密 Runloop](http://mrpeak.cn/blog/ios-runloop/)

# 啥是 Run Loop

run loop 是与线程相关的基础架构的一部分。 它是**一个事件处理循环，用于计划工作并协调传入事件的接收**。 它的**目的是**在有工作时保持线程忙，并在没有工作时让线程进入休眠状态。 我们开发者的代码提供了用于实现 run loop 的实际循环部分的控制语句 - 换句话说，代码提供了驱动 run loop 的 while 或 for 循环。 在循环中，使用 run loop 对象**运行接收**事件处理代码，并调用已安装的处理程序。

run loop 从**两种不同类型的源接收事件**。 输入源(input sources)提供异步事件，通常是来自另一个线程或来自不同应用程序的消息。 定时器源(timer sources)提供同步事件，发生在预定时间或重复间隔。 两种类型的源都使用特定于应用程序的处理程序程序来**处理**到达事件。

下图显示了 run loop 和各种源的概念结构。 输入源将异步事件传递给相应的处理程序，并调用 `runUntilDate:` 方法（在线程的关联 NSRunLoop 对象上调用）退出。 计时器源将事件传递给其处理程序，但不会导致 run loop 退出。

![Structure of a run loop and its sources](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Art/runloop.jpg)

除了处理输入源之外， run loop 还会生成有关 run loop 行为的通知。 可以使用 Core Foundation 在线程上安装 run loop 观察器。

## Input Sources

通常分两类，它们都是以异步方式向线程传递事件，唯一区别是**它们如何发出信号**。

- Port-based input sources: 基于的端口的输入源，监听应用程序的 Mach 端口（由内核自动发出信号，源码中被称为 source1 ）。
- Custom input sources: 自定义输入源，监听自定义事件源（必须从另一个线程手动发信号通知自定义源，源码中被称为 source0 ）。

如果输入源未处于当前监听模式，则它生成的任何事件会被 hold 住，直到 run loop 以正确模式运行。

### Port-Based Sources

Cocoa 和 Core Foundation 提供 使用与端口相关的对象和函数**创建基于端口的输入源 的内置支持**。 例如，在 Cocoa 中，根本不必直接创建输入源，只需创建一个端口对象，并使用 NSPort 的方法将该端口添加到 run loop 中。 port 对象处理所需输入源的创建和配置。

在 Core Foundation 中，必须手动创建端口及其 run loop 源。 在这两种情况下，都使用与端口 opaque 类型(CFMachPortRef CFMessagePortRef 或 CFSocketRef)相关联的函数来创建适当的对象。

### Custom Input Sources

要创建自定义输入源，必须在 Core Foundation 中使用与 CFRunLoopSourceRef opaque 类型关联的函数。 可以使用多个回调函数配置**自定义输入源**。 Core Foundation 在不同的点调用这些函数来配置源，处理传入事件，并在从 run loop 中删除源时拆除源。

除了在事件到达时，定义自定义源的行为，还必须定义事件**传递机制**。 源的这一部分在一个单独的线程上运行，负责为输入源提供数据，并在数据准备好进行处理时发出信号。 事件传递机制取决于我们开发者，但不必过于复杂。

### Cocoa Perform Selector Sources

除了基于端口的源之外， Cocoa 还定义了一个自定义输入源，允许在任何线程上执行选择器(selector)。与基于端口的源类似，在目标线程上**依次执行**选择器请求，从而减轻了在一个线程上运行多个方法时可能发生的许多**同步问题**。与基于端口的源不同，选择器源在执行后**将其自身从 run loop 中移除**。

在另一个线程上执行选择器时，目标线程**必须具有** active run loop 。对于自定义创建的线程，这意味着要等到代码显式启动 run loop 。但是，因为主线程启动了自己的 run loop ，所以只要应用程序调用应用程序委托的 `applicationDidFinishLaunching:` 方法，就可以开始在该线程上发出调用。 run loop 每次通过循环处理**所有排队**的执行选择器调用，而不是在每次循环迭代期间**处理一个**。

下表列出了在 NSObject 上定义的可用于在其他线程上执行选择器的方法。因为这些方法是在 NSObject 上声明的，所以可以在**任何**可以访问 Objective-C 对象的线程中使用它们，包括 POSIX 线程。这些方法实际上**并不创建新线程来执行选择器**。

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `performSelectorOnMainThread:withObject:waitUntilDone:`, `performSelectorOnMainThread:withObject:waitUntilDone:modes:` | 在主线程的**下一个 run loop 周期**中，执行指定的选择器。 提供了阻止当前线程直到执行选择器的选项。 |
| `performSelector:onThread:withObject:waitUntilDone:`, `performSelector:onThread:withObject:waitUntilDone:modes:` | 在**具有 NSThread 对象**的线程上执行指定的选择器 。 也提供了阻止当前线程直到执行选择器的选项。 |
| `performSelector:withObject:afterDelay:`, `performSelector:withObject:afterDelay:inModes:` | 在当前线程上的下一个 run loop 周期和可选的延迟后，执行指定的选择器。 |
| `cancelPreviousPerformRequestsWithTarget:`, `cancelPreviousPerformRequestsWithTarget:selector:object:` | 给使用 `performSelector:withObject:afterDelay:` 或`performSelector:withObject:afterDelay:inModes:` 方法的当前线程发送取消消息。 |

## Timer Sources

计时器源在将来的预设时间将事件同步传递给线程。定时器是**线程通知自己做某事的一种方式**。

虽然它生成基于时间的通知，但计时器**不是实时的** （它有一个属性 tolerance (宽容度)，标示了当时间点到后，容许有多少最大误差，目的就是为了节省资源）。 与输入源类似，定时器与 run loop 的**特定模式相关联**。 如果计时器未处于 run loop 当前正在监听的模式，则在使用其中一个计时器支持的模式运行 run loop 之前，它不会触发。 类似地，如果计时器在 run loop **处于执行处理程序的过程中触发，则计时器将等待直到下一次**通过 run loop 来调用其处理程序。 如果 run loop 根本没有运行，则计时器永远不会触发。

可以将计时器配置为仅生成一次或重复生成事件。 重复计时器根据**计划的触发(fire)时间自动重新安排自己，而不是实际触发时间**。 如果触发时间延迟太多以至于错过了一个或多个预定发射时间，则计时器仅在错过的时间段内**触发一次**。 在为错过的时段触发之后，计时器被重新安排用于下一个预定的触发时间。

## Run Loop Observers

与在发生**适当的**异步或同步事件时触发的源相反， observer 在执行 run loop 期间在特殊位置触发。 特殊的位置如下：

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0), // 进入 run loop
    kCFRunLoopBeforeTimers = (1UL << 1), // run loop 即将处理 timer
    kCFRunLoopBeforeSources = (1UL << 2), // run loop 即将处理 input source
    kCFRunLoopBeforeWaiting = (1UL << 5), // run loop 即将进入休眠状态时
    kCFRunLoopAfterWaiting = (1UL << 6), // run loop 已经被唤醒，在处理唤醒它的事件之前
    kCFRunLoopExit = (1UL << 7),  // 从 run loop 退出
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```

可以使用 Core Foundation 将 observer 添加到应用程序。要创建 observer ，请创建 CFRunLoopObserverRef opaque 类型的新实例。 此类型会跟踪自定义回调函数及其感兴趣的活动。

与定时器类似， **observer 可以使用一次或重复使用**。一次性 observer 在触发后将其自身从 run loop 中移除，而重复的 observer 仍然存在。

## The Run Loop Sequence of Events

线程的 run loop 每次运行时，都会处理挂起(pending)的事件，并为所有附加的 observer 生成通知。它的执行顺序如下：

1. 通知 observer 进入 run loop
2. 通知 observer 准备好的 timer 即将触发
3. 通知 observer source0 即将触发
4. 触发准备好的 source0
5. 如果 source1 已经准备好并且等待触发，立即处理事件，并跳到第 9 步
6. 通知 observer 线程即将进入休眠状态
7. 让线程置于休眠状态，直到被以下条件唤醒：
   - source1 事件到达
   - timer 触发
   - 为 run loop 设置的超时值到了
   - run loop 被手动唤醒
8. 通知 observer 线程刚被唤醒
9. 处理挂起的事件
   - 如果用户定义 timer 被触发了，处理 timer 事件并重启 run loop 。并跳到第 2 步
   - 如果 input source 被触发了，分发事件
   - 如果 run loop 被手动唤醒但是还没超时，重启 run loop 。并跳到第 2 步
10. 通知 observer 已经退出 run loop

核心代码主要在下面两个函数

```
SInt32 CFRunLoopRunSpecific(CFRunLoopRef rl, CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) 
static int32_t __CFRunLoopRun(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFTimeInterval seconds, Boolean stopAfterHandle, CFRunLoopModeRef previousMode)
```



相关核心代码(摘取了包含前缀`__CFRunLoop` 的代码)如下：

```
// 1. 通知 observer 进入 run loop
if (currentMode->_observerMask & kCFRunLoopEntry ) __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopEntry); // 最终走 __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__

do {
    // 2. 通知 observer 准备好的 timer 即将触发
    if (rlm->_observerMask & kCFRunLoopBeforeTimers) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeTimers);
    // 3. 通知 observer source0 即将触发 
    if (rlm->_observerMask & kCFRunLoopBeforeSources) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeSources);

    // 执行 Block
    __CFRunLoopDoBlocks(rl, rlm);  // 最终走 __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__

    // 4. 触发准备好的 source0
    Boolean sourceHandledThisLoop = __CFRunLoopDoSources0(rl, rlm, stopAfterHandle); // 最终走 __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__
    if (sourceHandledThisLoop) {
        __CFRunLoopDoBlocks(rl, rlm);
	}

    // 5. 如果 source1 已经准备好并且等待触发，立即处理事件，并跳到第 9 步 （读取和 mainQueue 相关的消息，这里不会导致睡眠(入参 timeout = 0)，保证 dispatch 到 mainQueue 代码有较高的机会运行）
    Boolean poll = sourceHandledThisLoop || (0ULL == timeout_context->termTSR);
    if (MACH_PORT_NULL != dispatchPort && !didDispatchPortLastTime) {
        msg = (mach_msg_header_t *)msg_buffer;
        if (__CFRunLoopServiceMachPort(dispatchPort, &msg, sizeof(msg_buffer), &livePort, 0, &voucherState, NULL)) {
            goto handle_msg;
        }
    }

    // 6. 通知 observer 线程即将进入休眠状态
    if (!poll && (rlm->_observerMask & kCFRunLoopBeforeWaiting)) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeWaiting);


	__CFRunLoopSetSleeping(rl);
    __CFPortSetInsert(dispatchPort, waitSet);
	__CFRunLoopModeUnlock(rlm);
	__CFRunLoopUnlock(rl);
    if (kCFUseCollectableAllocator) {
        // objc_clear_stack(0);
       // <rdar://problem/16393959>
       memset(msg_buffer, 0, sizeof(msg_buffer));
    }
    msg = (mach_msg_header_t *)msg_buffer;
    // 7. 让线程置于休眠状态，直到被以下条件唤醒 (注意入参poll 的值，是否睡眠)
    __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY, &voucherState, &voucherCopy);

    __CFRunLoopLock(rl);
    __CFRunLoopModeLock(rlm);
    __CFPortSetRemove(dispatchPort, waitSet);
    __CFRunLoopSetIgnoreWakeUps(rl);
	__CFRunLoopUnsetSleeping(rl);

    // 8. 通知 observer 线程刚被唤醒
	if (!poll && (rlm->_observerMask & kCFRunLoopAfterWaiting)) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopAfterWaiting);

    // 9. 处理挂起的事件
    handle_msg:;
        __CFRunLoopSetIgnoreWakeUps(rl);

        if (MACH_PORT_NULL == livePort) { // 没事情
            CFRUNLOOP_WAKEUP_FOR_NOTHING();
            // handle nothing
        } else if (livePort == rl->_wakeUpPort) {
            CFRUNLOOP_WAKEUP_FOR_WAKEUP();
        } else if (modeQueuePort != MACH_PORT_NULL && livePort == modeQueuePort) { // (或者 livePort == rlm->_timerPort) 处理 timer
            CFRUNLOOP_WAKEUP_FOR_TIMER();
            if (!__CFRunLoopDoTimers(rl, rlm, mach_absolute_time())) { // 最终走 __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__
                // Re-arm the next timer, because we apparently fired early
                __CFArmNextTimerInMode(rlm, rl);
            }
        } else if (livePort == dispatchPort) { // dispatch mainQueue 事件
            CFRUNLOOP_WAKEUP_FOR_DISPATCH();
            __CFRunLoopModeUnlock(rlm);
            __CFRunLoopUnlock(rl);

            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg); 

            __CFRunLoopLock(rl);
            __CFRunLoopModeLock(rlm);
        } else { // source1
            CFRUNLOOP_WAKEUP_FOR_SOURCE();

            CFRunLoopSourceRef rls = __CFRunLoopModeFindSourceForMachPort(rl, rlm, livePort);
            if (rls) {
		    mach_msg_header_t *reply = NULL;
		    sourceHandledThisLoop = __CFRunLoopDoSource1(rl, rlm, rls, msg, msg->msgh_size, &reply) || sourceHandledThisLoop; // 最终走 __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__
		    if (NULL != reply) {
		        (void)mach_msg(reply, MACH_SEND_MSG, reply->msgh_size, 0, MACH_PORT_NULL, 0, MACH_PORT_NULL);
		     CFAllocatorDeallocate(kCFAllocatorSystemDefault, reply);
		    }
	        }
        }

    // 执行 Block
    __CFRunLoopDoBlocks(rl, rlm);
        
    // 判断循环条件
	if (sourceHandledThisLoop && stopAfterHandle) {
	    retVal = kCFRunLoopRunHandledSource;
        } else if (timeout_context->termTSR < mach_absolute_time()) {
            retVal = kCFRunLoopRunTimedOut;
	} else if (__CFRunLoopIsStopped(rl)) {
            __CFRunLoopUnsetStopped(rl);
	    retVal = kCFRunLoopRunStopped;
	} else if (rlm->_stopped) {
	    rlm->_stopped = false;
	    retVal = kCFRunLoopRunStopped;
	} else if (__CFRunLoopModeIsEmpty(rl, rlm, previousMode)) {
	    retVal = kCFRunLoopRunFinished;
	}

} while (0 == retVal);

// 10. 通知 observer 已经退出 run loop 
if (currentMode->_observerMask & kCFRunLoopExit ) __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
```

可以参考高峰大佬的[解密 Runloop](http://mrpeak.cn/blog/ios-runloop/) 里面的流程图

![run loop sequence](https://github.com/JoakimLiu/BlogPhoto/blob/master/runloop/runloop-sequence-mrpeak.png?raw=true)

由于 timer 和 input sources 的 observer 通知是在这些事件**实际发生之前传递的**，因此通知时间与实际事件的时间之间可能存在差距。 如果这些事件之间的时间关系很重要，可以使用**休眠和唤醒休眠通知来帮助关联**实际事件之间的时间。

可以使用 run loop 对象手动唤醒它，其他事件也可能导致 run loop 被唤醒。例如，添加另一个非基于端口的输入源会唤醒 run loop ，以便可以立即处理输入源，而不是等到其他事件发生。

## Run Loop Modes

这个概念很重要， run loop mode(模式) 是要监听的输入源(input sources)和计时器(timers)**的集合**，以及要通知的 run loop observers **的集合**。每次运行 run loop 时，都指定（显式或隐式）运行的特定模式。在 run loop 的传递过程中，**仅监听与该模式关联的源并允许其传递事件**。 类似地，只有与该模式相关联的 observers 被通知 run loop 的进度。 与其他模式相关联的源保持新事件，直到后续以适当模式通过循环。

在代码中，可以按名称识别模式。 Cocoa 和 Core Foundation 都定义了默认模式和几种常用模式(见下表)，以及用于在代码中指定这些模式的字符串。 只需为模式名称指定自定义字符串**即可定义自定义模式**。 虽然为自定义模式指定的名称是任意的，但**这些模式的内容不是**。 必须确保将一个或多个输入源，计时器或 run loop 观察器**添加到为其创建的任何模式中**才有用。 (ps: 模式中要有相关源和观察器才有用)

| Mode           | Name                                                         | Description                                                  |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Default        | NSDefaultRunLoopMode (Cocoa), kCFRunLoopDefaultMode (Core Foundation) | 用于大多数操作的模式，大多数情况，应该使用它启动 run loop 并配置输入源。 |
| Connection     | NSConnectionReplyMode (Cocoa)                                | 与 NSConnection 对象结合使用去监听回复，很少用到。           |
| Modal          | NSModalPanelRunLoopMode (Cocoa)                              | 识别用于模态面板(modal panels)的事件。                       |
| Event tracking | NSEventTrackingRunLoopMode (Cocoa)                           | 限制 在鼠标拖动循环和其他用户界面跟踪循环期间 的传入事件。   |
| Common modes   | NSRunLoopCommonModes (Cocoa), kCFRunLoopCommonModes (Core Foundation) | 可配置的常用模式。 将输入源与此模式相关联，也会将其与组中的**每个模式相关联**。 对于 Cocoa 应用程序，此集合默认包括 Default ，Modal 和 Event tracking 模式。 Core Foundation 最初只包含 Default 模式，可以使用 CFRunLoopAddCommonMode 函数将自定义模式添加到集合中。 |

可以使用模式在特定的运行循环期间过**滤掉不需要的**来源中的事件。 大多数情况下，需要在系统定义的“默认”模式下运行运行循环。 但是，模态面板可能以“模态”模式运行，在此模式下，只有与模态面板相关的源才会将事件传递给线程。对于次级线程，可以使用自定义模式来**防止低优先级源在时间关键操作期间传递事件**。

注意：**模式根据事件的来源而不是事件的类型进行区分**。

run loop mode 结构体定义如下。

```
struct __CFRunLoopMode {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;	/* must have the run loop locked before locking this */
    CFStringRef _name;
    Boolean _stopped;
    char _padding[3];
    CFMutableSetRef _sources0; 
    CFMutableSetRef _sources1;
    CFMutableArrayRef _observers;
    CFMutableArrayRef _timers;
    CFMutableDictionaryRef _portToV1SourceMap;
    __CFPortSet _portSet;
    CFIndex _observerMask;
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    dispatch_source_t _timerSource;
    dispatch_queue_t _queue;
    Boolean _timerFired; // set to true by the source when a timer has fired
    Boolean _dispatchTimerArmed;
#endif
#if USE_MK_TIMER_TOO
    mach_port_t _timerPort;
    Boolean _mkTimerArmed;
#endif
#if DEPLOYMENT_TARGET_WINDOWS
    DWORD _msgQMask;
    void (*_msgPump)(void);
#endif
    uint64_t _timerSoftDeadline; /* TSR */
    uint64_t _timerHardDeadline; /* TSR */
};
```

我们可以看到结构体里面有定义 _sources0 _sources1 _observers _timers ，唯独没有定义 mainQueue ，因为 mainQueue 的执行与 mode 没关系。 前面 run loop 执行的流程代码，在调用 `__CFRunLoopDoObservers XXXXXX` 方法通知 observers 回调之前，都会用 `(currentMode->_observerMask & CFRunLoopActivity)` 做判断，只有 _observerMask 里面有相关类型值，才会回调 observers 。 用下面方法添加 observers 的时候， _observerMask 都会更新

```
void CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef rlo, CFStringRef modeName)
```

# 什么时候需要 Run Loop?

手动运行 run loop 的**唯一时间是为应用程序创建次级线程**。 应用程序主线程的 run loop 是一个至关重要的基础架构，在启动程序的时候，主循环就已经创建好了。

对于次级线程，需要确定是否需要 run loop ，如果是，**请自行配置并启动它**。 如果使用线程执行某些长时间运行且预定义的任务，则可以**不需要启动** run loop 。**run loop 适用于希望与线程进行更多交互的情况**。 例如：

- 使用端口或自定义 input source 与其他线程通信
- 在线程上使用 timer
- 在 Cocoa 应用程序中使用任何 `performSelector...` 方法
- 保持线程以执行定期任务

如果确实选择使用 run loop ，则配置和设置非常简单。与所有线程编程一样，应该在适当情况下退出次级线程，而不是强行退出。

# 使用 Run Loop 对象

run loop 对象提供了主要接口，用于将 input sources ，timers 和 observers **添加到** run loop ，然后运行它。 **每个线程都有一个与之关联的 run loop 对象**。 在 Cocoa 中，是 [NSRunLoop](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSRunLoop/Description.html#//apple_ref/occ/cl/NSRunLoop) 类的实例。 在低级应用程序中，它是指向 [CFRunLoopRef](https://developer.apple.com/documentation/corefoundation/cfrunloopref) opaque 类型的指针。

## Getting a Run Loop Object

获取当前线程的 run loop ，可以使用下面的其中一个方法

- 在 Cocoa App 中，调用 NSRunLoop 的 `currentRunLoop` 类方法，获取 NSRunLoop 对象
- 使用 `CFRunLoopGetCurrent` 函数

虽然它们**不是** toll-free bridged 类型，但可以在需要时从 NSRunLoop 对象获取 CFRunLoopRef opaque 类型。
因为两者**都引用相同的** run loop 对象，所以可以混合使用。

## Configuring the Run Loop

在次级线程上运行 run loop 之前，**必须至少添加一个输入源或计时器**。如果 run loop 没有要监听的任何源，则在尝试运行它时**会立即退出**。

除了安装源之外，还可以安装运行 observer 并使用它们来检测 run loop 的不同执行阶段。下面的代码展示了一个将 observer 附加到其 run loop 的线程的主程序。如何创建 observer ，以及用它来监听 run loop 的所有活动，没有显示监听回调函数 `myRunLoopObserver` 。

```
// Listing 3-1 Creating a run loop observer
- (void)threadMain
{
    // The application uses garbage collection, so no autorelease pool is needed.
    NSRunLoop* myRunLoop = [NSRunLoop currentRunLoop];
 
    // Create a run loop observer and attach it to the run loop.
    CFRunLoopObserverContext  context = {0, self, NULL, NULL, NULL};
    CFRunLoopObserverRef    observer = CFRunLoopObserverCreate(kCFAllocatorDefault,
            kCFRunLoopAllActivities, YES, 0, &myRunLoopObserver, &context);
 
    if (observer)
    {
        CFRunLoopRef    cfLoop = [myRunLoop getCFRunLoop];
        CFRunLoopAddObserver(cfLoop, observer, kCFRunLoopDefaultMode);
    }
 
    // Create and schedule the timer.
    [NSTimer scheduledTimerWithTimeInterval:0.1 target:self
                selector:@selector(doFireTimer:) userInfo:nil repeats:YES];
 
    NSInteger    loopCount = 10;
    do
    {
        // Run the run loop 10 times to let the timer fire.
        [myRunLoop runUntilDate:[NSDate dateWithTimeIntervalSinceNow:1]];
        loopCount--;
    }
    while (loopCount);
}
```

为长期存在的线程配置 run loop 时，最好添加**至少一个输入源来接收消息**。 **虽然可以连接一个定时器**进入 run loop ，但一旦定时器触发，它通常会失效，这会导致运行循环退出。 重复计时器可以使 run loop 运行更长的时间，但是会涉及**定期触发计时器以唤醒线程，这实际上是另一种形式的轮询**。 相比之下，**输入源会等待事件发生，让线程保持睡眠状态**。

## Starting the Run Loop

只用次级线程才需要启动 run loop 。 run loop 必须至少有一个输入源或者计时器才能进行监听，如果没有，则**会立即退出**。有以下几种方法可以启动 run loop

- 无条件 (`run`)
- 设置时限 (`runUntilDate:`)
- 在特定模式下 (`runMode:beforeDate:`)

无条件是最简单的，但也是最不可取的。它让 run loop 置于永久循环中，使得我们开发者**无法控制它**。我们可以添加和删除输入源或者计时器，但是**停止它的唯一方法是杀掉他**。并且它**也没法在自定义模式下运行**。

最好设置时限运行 run loop ，它**将一直运行，直到事件到达或分配的时间到期**。如果事件到达，则将该事件分派给处理程序进行处理，然后退出 run loop 。可以重新启动它以处理下一个事件。如果指定的时间到期了，只需重新启动它或使用时间进行任何所需的内务(housekeeping)处理。

可以使用特定模式运行 run loop 。它与设置时限不是互斥的，模式**限制将事件传递到 run loop 的源类型**。

下面的代码展示了线程主入口程序的框架，显示了 run loop 的基本结构。 实质上，将输入源和计时器添加到 run loop 中，然后重复调用其中一个程序以启动 run loop 。 每次 run loop 程序返回时，都会检查是否出现了可能需要退出该线程的任何条件。 该示例使用 Core Foundation 运行 run loop ，以便它可以检查返回结果并确定运行循环退出的原因。 如果使用 Cocoa 并且**不需要检查返回值**，也可以使用 NSRunLoop 类的方法以类似的方式运行 run loop (后面 Listing 3-14 有代码)。

```
// Listing 3-2 Running a run loop
- (void)skeletonThreadMain
{
    // Set up an autorelease pool here if not using garbage collection.
    BOOL done = NO;
 
    // Add your sources or timers to the run loop and do any other setup.
 
    do
    {
        // Start the run loop but return after each source is handled.
        SInt32    result = CFRunLoopRunInMode(kCFRunLoopDefaultMode, 10, YES);
 
        // If a source explicitly stopped the run loop, or if there are no
        // sources or timers, go ahead and exit.
        if ((result == kCFRunLoopRunStopped) || (result == kCFRunLoopRunFinished))
            done = YES;
 
        // Check for any other exit conditions here and set the
        // done variable as needed.
    }
    while (!done);
 
    // Clean up code here. Be sure to release any allocated autorelease pools.
}
```

可以**递归地运行** run loop 。 换句话说，可以调用 CFRunLoopRun ， CFRunLoopRunInMode 或任何 NSRunLoop 方法，以便从输入源或计时器的**处理程序程序中启动 run loop** 。 执行此操作时，可以使用任何要运行嵌套 run loop 的模式，包括外部 run loop 使用的模式。

## Exiting the Run Loop

在处理事件之前，有两种方法可以使 run loop 退出：

- 使用超时值
- 告诉 run loop 停止

推荐使用超时值，因为我们可以管理 run loop 。它能让 run loop **完成所有的正常处理**，包括给 observer 发送通知。

使用 `CFRunLoopStop` 函数可以手动停止 run loop。 run loop 发出剩下的通知后退出。用这个方法可以停止无条件启动的 run loop 。

虽然删除 run loop 的输入源和定时器也可能导致运行循环退出，但这不是停止 run loop 的可靠方法。一些系统程序将输入源添加到 run loop 以处理所需的事件。我们**可能不知道它们**，所以删除的时候就会漏掉它们。

## Thread Safety and Run Loop Objects

Core Foundation 中的函数通常是线程安全的，可以从任何线程调用。但是，如果要执行更改 run loop 配置的操作，**尽可能从拥有 run loop 的线程执行此操作**。

Cocoa NSRunLoop 类不像 Core Foundation 对应的那样具有内在的线程安全性。 如果使用 NSRunLoop 类来修改 run loop ，**必须在拥有该 run loop 的同一线程执行此操作**。 将输入源或计时器添加到属于不同线程的 run loop 可能会导致代码崩溃或以意外方式运行。
(ps: 在拥有 run loop 的线程中做相关更改 run loop 的操作。)

# Configuring Run Loop Sources

## Defining a Custom Input Source

创建自定义输入源涉及到以下定义：

- The information you want your input source to process. (希望输入源处理的信息)
- A scheduler routine to let interested clients know how to contact your input source. (让感兴趣的客户端知道如何联系输入源的 scheduler 程序)
- A handler routine to perform requests sent by any clients. (执行 客户端发送的请求的 处理程序)
- A cancellation routine to invalidate your input source. (使输入源无效的取消程序)

由于是自定义创建输入源，所以实际配置的设计非常灵活。 scheduler handler cancellation 这些程序的实现**是关键**。 但是，大多数输入源行为的其余部分都发生在这些处理程序之外，可以定义 将数据传递到输入源以及将输入源传递给其他线程的 机制。

下图显示了自定义输入源的示例配置。主线程维护对输入源的引用，以及该输入源的自定义命令缓冲区(custom command buffer)以及安装输入源的 run loop 。当主线程有一个任务，想要传递给工作线程时，它会向命令缓冲区发布一个命令以及工作线程启动任务所需的任何信息。 （因为主线程和工作线程的输入源都可以访问命令缓冲区，所以必须同步该访问。） 一旦命令发布，主线程就会发出信号输入源并唤醒工作线程的 run loop 。收到唤醒命令后， run loop 调用输入源的处理程序，该处理程序处理命令缓冲区中的命令。

![Operating a custom input source](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Art/custominputsource.jpg)

以下部分介绍了上图中自定义输入源的实现，并显示了需要实现的关键代码。(ps: 具体 demo 见 [RunLoopDemo](https://github.com/JoakimLiu/RunLoopDemo))

### Defining the Input Source

定义自定义输入源需要使用 Core Foundation 程序来配置 run loop 源并将其附加到 run loop 。 虽然基本处理程序是基于 C 的函数，但这并不妨碍为这些函数编写包装器并使用 Objective-C 或 C++ 来实现。

下面的代码中， RunLoopSource 对象管理命令缓冲区并使用该缓冲区从其他线程接收消息。 RunLoopContext 是一个容器对象，用于传递 RunLoopSource 对象和对应用程序主线程的 run loop 引用。

```
// Listing 3-3  The custom input source object definition
@interface RunLoopSource : NSObject
{
    CFRunLoopSourceRef runLoopSource;
    NSMutableArray* commands;
}
 
- (id)init;
- (void)addToCurrentRunLoop;
- (void)invalidate;
 
// Handler method
- (void)sourceFired;
 
// Client interface for registering commands to process
- (void)addCommand:(NSInteger)command withData:(id)data;
- (void)fireAllCommandsOnRunLoop:(CFRunLoopRef)runloop;
 
@end
 
// These are the CFRunLoopSourceRef callback functions.
void RunLoopSourceScheduleRoutine (void *info, CFRunLoopRef rl, CFStringRef mode);
void RunLoopSourcePerformRoutine (void *info);
void RunLoopSourceCancelRoutine (void *info, CFRunLoopRef rl, CFStringRef mode);
 
// RunLoopContext is a container object used during registration of the input source.
@interface RunLoopContext : NSObject
{
    CFRunLoopRef        runLoop;
    RunLoopSource*        source;
}
@property (readonly) CFRunLoopRef runLoop;
@property (readonly) RunLoopSource* source;
 
- (id)initWithSource:(RunLoopSource*)src andLoop:(CFRunLoopRef)loop;
@end
```

当将 run loop 源添加到 run loop 时，就调用 `RunLoopSourceScheduleRoutine` 函数。之前有说过，输入源只有一个客户端，那就是主线程，所以将 RunLoopContext 传递过去，方便 delegate 和输入源通信。

```
// Listing 3-4 Scheduling a run loop source
void RunLoopSourceScheduleRoutine (void *info, CFRunLoopRef rl, CFStringRef mode)
{
    RunLoopSource* obj = (RunLoopSource*)info;
    AppDelegate*   del = [AppDelegate sharedAppDelegate];
    RunLoopContext* theContext = [[RunLoopContext alloc] initWithSource:obj andLoop:rl];
 
    [del performSelectorOnMainThread:@selector(registerSource:)
                                withObject:theContext waitUntilDone:NO];
}
```



当输入源被发出信号时，调用 `RunLoopSourcePerformRoutine` 处理数据，它是最重要的回调函数。 这里只是简单的调用 RunLoopSource 的 `sourceFired` 方法，后面会讲到。

```
// Listing 3-5  Performing work in the input source
void RunLoopSourcePerformRoutine (void *info)
{
    RunLoopSource*  obj = (RunLoopSource*)info;
    [obj sourceFired];
}
```

如果使用 `CFRunLoopSourceInvalidate` 函数从 run loop 中删除输入源，系统将调用输入源的取消程序 `RunLoopSourceCancelRoutine` 。

```
// Listing 3-6  Invalidating an input source
void RunLoopSourceCancelRoutine (void *info, CFRunLoopRef rl, CFStringRef mode)
{
    RunLoopSource* obj = (RunLoopSource*)info;
    AppDelegate* del = [AppDelegate sharedAppDelegate];
    RunLoopContext* theContext = [[RunLoopContext alloc] initWithSource:obj andLoop:rl];
 
    [del performSelectorOnMainThread:@selector(removeSource:)
                                withObject:theContext waitUntilDone:YES];
}
```

### Installing the Input Source on the Run Loop

下面显示了 RunLoopSource 类的 `init` 和 `addToCurrentRunLoop` 方法。 `init` 方法创建 CFRunLoopSourceRef opaque 类型，**该类型必须附加到 run loop** 。 它将 RunLoopSource 对象本身作为上下文信息传递，以便回调程序具有指向该对象的指针。 在工作线程调用 `addToCurrentRunLoop` 方法时才会安装输入源，此时将调用 `RunLoopSourceScheduleRoutine` 回调函数。 一旦输入源被添加到 run loop 中，线程就可以运行其 run loop 来等待它。

```
// Listing 3-7  Installing the run loop source
- (id)init
{
    CFRunLoopSourceContext    context = {0, self, NULL, NULL, NULL, NULL, NULL,
                                        &RunLoopSourceScheduleRoutine,
                                        RunLoopSourceCancelRoutine,
                                        RunLoopSourcePerformRoutine};
 
    runLoopSource = CFRunLoopSourceCreate(NULL, 0, &context);
    commands = [[NSMutableArray alloc] init];
 
    return self;
}
 
- (void)addToCurrentRunLoop
{
    CFRunLoopRef runLoop = CFRunLoopGetCurrent();
    CFRunLoopAddSource(runLoop, runLoopSource, kCFRunLoopDefaultMode);
}
```



### Coordinating with Clients of the Input Source

要使输入源有用，需要对其进行操作，并从另一个线程发出信号。输入源的**重点是**将其关联的线程置于休眠状态，直到有事情要做才唤醒。所以，需要让应用程序中的**其他线程知道输入源**，并有办法与之通信。

通知客户端输入源的一种方法是，在输入源首次安装在其 run loop 上时**发出注册请求**。 可以使用任意数量的客户端注册输入源，或者只需将其注册到某个中央代理商，然后将输入源发送给感兴趣的客户。下面代码显示了应用程序委托定义的注册方法，并在调用 RunLoopSource 对象的调度程序函数时调用。 此方法接收 RunLoopSource 对象提供的 RunLoopContext 对象，并将其添加到其源列表中。 还显示了从 run loop 中删除输入源时用于取消注册的程序。

```
// Listing 3-8  Registering and removing an input source with the application delegate
- (void)registerSource:(RunLoopContext*)sourceInfo;
{
    [sourcesToPing addObject:sourceInfo];
}
 
- (void)removeSource:(RunLoopContext*)sourceInfo
{
    id    objToRemove = nil;
 
    for (RunLoopContext* context in sourcesToPing)
    {
        if ([context isEqual:sourceInfo])
        {
            objToRemove = context;
            break;
        }
    }
 
    if (objToRemove)
        [sourcesToPing removeObject:objToRemove];
}
```

### Signaling the Input Source

在将数据移交给输入源之后，**客户端必须向源发信号并唤醒其 run loop** 。 信号源使 run loop 知道源已准备好进行处理。 并且因为线程可能在信号发生时处于睡眠状态，所以手动唤醒 run loop 。 如果不这样做可能会导致处理输入源的延迟。

下面代码显示了 RunLoopSource 对象的 `fireCommandsOnRunLoop` 方法。 当客户端准备好处理他们添加到缓冲区的命令时，客户端会调用此方法。

```
// Listing 3-9  Waking up the run loop
- (void)fireCommandsOnRunLoop:(CFRunLoopRef)runloop
{
    CFRunLoopSourceSignal(runLoopSource);
    CFRunLoopWakeUp(runloop);
}
```

## Configuring Timer Sources

要创建计时器源，所要做的就是创建一个计时器对象并在 run loop 上安排(schedule)它。在 Cocoa 中，使用 NSTimer 创建计时器对象，在 Core Foundation 中使用 CFRunLoopTimerRef opaque 类型。在内部， NSTimer 类只是 Core Foundation 的扩展，它提供了一些便利功能。用以下方法可以创建

- `cheduledTimerWithTimeInterval:target:selector:userInfo:repeats:`
- `scheduledTimerWithTimeInterval:invocation:repeats:`

这些方法创建计时器并将其以默认模式（NSDefaultRunLoopMode）**添加到当前线程**的 run loop 中。如果需要，还可以手动调度计时器，方法是创建 NSTimer 对象，然后使用 NSRunLoop 的 `addTimer:forMode:` 方法将其添加到 run loop 中。这两种技术基本上都是一样的，但是可以对计时器的配置进行不同程度的控制。例如，如果创建计时器并手动将其添加到 run loop ，则可以使用默认模式以外的模式执行此操作。下面的代码显示了如何使用这两种技术创建计时器。第一个计时器的初始延迟为 1 秒，但之后每 0.1 秒定时触发一次。第二个计时器在最初的 0.2秒延迟后开始触发，然后每 0.2 秒触发一次。

```
// Listing 3-10  Creating and scheduling timers using NSTimer
NSRunLoop* myRunLoop = [NSRunLoop currentRunLoop];
 
// Create and schedule the first timer.
NSDate* futureDate = [NSDate dateWithTimeIntervalSinceNow:1.0];
NSTimer* myTimer = [[NSTimer alloc] initWithFireDate:futureDate
                        interval:0.1
                        target:self
                        selector:@selector(myDoFireTimer1:)
                        userInfo:nil
                        repeats:YES];
[myRunLoop addTimer:myTimer forMode:NSDefaultRunLoopMode];
 
// Create and schedule the second timer.
[NSTimer scheduledTimerWithTimeInterval:0.2
                        target:self
                        selector:@selector(myDoFireTimer2:)
                        userInfo:nil
                        repeats:YES];
```

下面的代码展示了使用 Core Foundation 函数配置计时器所需的代码。 虽然此示例未在上下文结构中传递任何用户定义的信息，但可以使用此结构传递计时器所需的任何自定义数据。

```
// Listing 3-11  Creating and scheduling a timer using Core Foundation
CFRunLoopRef runLoop = CFRunLoopGetCurrent();
CFRunLoopTimerContext context = {0, NULL, NULL, NULL, NULL};
CFRunLoopTimerRef timer = CFRunLoopTimerCreate(kCFAllocatorDefault, 0.1, 0.3, 0, 0,
                                        &myCFTimerCallback, &context);
 
CFRunLoopAddTimer(runLoop, timer, kCFRunLoopCommonModes);
```

## Configuring a Port-Based Input Source

Cocoa 和 Core Foundation 都提供了基于端口的对象，**用于线程之间或进程之间的通信**。 以下部分介绍如何使用多种不同类型的端口设置端口通信。

### Configuring an NSMachPort Object

要与 NSMachPort 对象建立本地连接，请创建端口对象并将其添加到主线程的 run loop 中。 启动次级线程时，将同一对象传递给线程的入口函数。 次级线程可以使用相同的对象将消息发送回主线程。

#### IMPLEMENTING THE MAIN THREAD CODE

下面的代码展示了启动次级工作线程的主要代码。因为 Cocoa 框架执行许多配置端口和 run loop 的干预步骤，所以 `launchThread` 方法明显比其 Core Foundation 等效方法要短（见后面 Listing 3-17）; 然而，两者的行为几乎完全相同。 一个区别是，该方法不是直接向工作线程发送本地端口的名称，而是直接发送 NSPort 对象。

```
// Listing 3-12  Main thread launch method
- (void)launchThread
{
    NSPort* myPort = [NSMachPort port];
    if (myPort)
    {
        // This class handles incoming port messages.
        [myPort setDelegate:self];
 
        // Install the port as an input source on the current run loop.
        [[NSRunLoop currentRunLoop] addPort:myPort forMode:NSDefaultRunLoopMode];
 
        // Detach the thread. Let the worker release the port.
        [NSThread detachNewThreadSelector:@selector(LaunchThreadWithPort:)
               toTarget:[MyWorkerClass class] withObject:myPort];
    }
}
```

为了在线程之间建立双向通信通道，可能希望让工作线程在 check-in 消息中将其自己的本地端口发送到主线程。 通过接收 check-in 消息，主线程可以知道在启动工作线程时一切顺利，并且还提供了向该线程发送更多消息的方法。

下面的代码显示了主线程的 `handlePortMessage:` 方法。 当数据到达线程自己的本地端口时，将调用此方法。 当 check-in 消息到达时，该方法直接从端口消息中检索次级线程的端口并保存以供以后使用。

```
// Listing 3-13  Handling Mach port messages

#define kCheckinMessage 100
 
// Handle responses from the worker thread.
- (void)handlePortMessage:(NSPortMessage *)portMessage
{
    unsigned int message = [portMessage msgid];
    NSPort* distantPort = nil;
 
    if (message == kCheckinMessage)
    {
        // Get the worker thread’s communications port.
        distantPort = [portMessage sendPort];
 
        // Retain and save the worker port for later use.
        [self storeDistantPort:distantPort];
    }
    else
    {
        // Handle other messages.
    }
}
```

#### IMPLEMENTING THE SECONDARY THREAD CODE

对于次级工作线程，必须配置线程并使用指定的端口将信息传递回主线程。

下面代码显示了设置工作线程的代码。 在为线程创建自动释放池之后，该方法创建一个工作对象来驱动线程执行。 worker 对象的 `sendCheckinMessage:`方法（如后面 Listing 3-15 所示）为工作线程创建一个本地端口，并将一个 check-in 发送回主线程。

```
// Listing 3-14  Launching the worker thread using Mach ports

+(void)LaunchThreadWithPort:(id)inData
{
    NSAutoreleasePool*  pool = [[NSAutoreleasePool alloc] init];
 
    // Set up the connection between this thread and the main thread.
    NSPort* distantPort = (NSPort*)inData;
 
    MyWorkerClass*  workerObj = [[self alloc] init];
    [workerObj sendCheckinMessage:distantPort];
    [distantPort release];
 
    // Let the run loop process things.
    do
    {
        [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode
                            beforeDate:[NSDate distantFuture]];
    }
    while (![workerObj shouldExit]);
 
    [workerObj release];
    [pool release];
}
```

使用 NSMachPort 时，本地和远程线程可以使用相同的端口对象进行线程之间的单向通信。 换句话说，由一个线程创建的本地端口对象成为另一个线程的远程端口对象。

下面代码显示了次级线程的 check-in 程序。 此方法为将来的通信设置自己的本地端口，然后将 check-in 消息发送回主线程。 该方法使用 `LaunchThreadWithPort:` 方法中接收的端口对象作为消息的目标。

```
// Listing 3-15  Sending the check-in message using Mach ports

// Worker thread check-in method
- (void)sendCheckinMessage:(NSPort*)outPort
{
    // Retain and save the remote port for future use.
    [self setRemotePort:outPort];
 
    // Create and configure the worker thread port.
    NSPort* myPort = [NSMachPort port];
    [myPort setDelegate:self];
    [[NSRunLoop currentRunLoop] addPort:myPort forMode:NSDefaultRunLoopMode];
 
    // Create the check-in message.
    NSPortMessage* messageObj = [[NSPortMessage alloc] initWithSendPort:outPort
                                         receivePort:myPort components:nil];
 
    if (messageObj)
    {
        // Finish configuring the message and send it immediately.
        [messageObj setMsgId:setMsgid:kCheckinMessage];
        [messageObj sendBeforeDate:[NSDate date]];
    }
}
```



### Configuring an NSMessagePort Object

要与 NSMessagePort 对象建立本地连接，不能简单地在线程之间传递端口对象。 必须按名称获取远程消息端口。 在 Cocoa 中实现这一点需要使用特定名称注册本地端口，然后将该名称传递给远程线程，以便它可以获取适当的端口对象进行通信。 下面代码显示了在要使用消息端口的情况下的端口创建和注册过程。

```
// Listing 3-16  Registering a message port

NSPort* localPort = [[NSMessagePort alloc] init];
 
// Configure the object and add it to the current run loop.
[localPort setDelegate:self];
[[NSRunLoop currentRunLoop] addPort:localPort forMode:NSDefaultRunLoopMode];
 
// Register the port using a specific name. The name must be unique.
NSString* localPortName = [NSString stringWithFormat:@"MyPortName"];
[[NSMessagePortNameServer sharedInstance] registerPort:localPort
                     name:localPortName];
```

### Configuring a Port-Based Input Source in Core Foundation

本节介绍如何使用 Core Foundation 在应用程序的主线程和工作线程之间建立双向通信通道。

下面代码显示了应用程序主线程调用以启动工作线程的代码。 代码所做的第一件事就是设置一个 CFMessagePortRef opaque 类型来监听来自工作线程的消息。 工作线程需要端口的名称来建立连接，以便将字符串值传递给工作线程的入口函数。 端口名称在当前用户上下文中通常应该是唯一的； 否则，可能会遇到冲突。

```
// Listing 3-17  Attaching a Core Foundation message port to a new thread

#define kThreadStackSize        (8 *4096)
 
OSStatus MySpawnThread()
{
    // Create a local port for receiving responses.
    CFStringRef myPortName;
    CFMessagePortRef myPort;
    CFRunLoopSourceRef rlSource;
    CFMessagePortContext context = {0, NULL, NULL, NULL, NULL};
    Boolean shouldFreeInfo;
 
    // Create a string with the port name.
    myPortName = CFStringCreateWithFormat(NULL, NULL, CFSTR("com.myapp.MainThread"));
 
    // Create the port.
    myPort = CFMessagePortCreateLocal(NULL,
                myPortName,
                &MainThreadResponseHandler,
                &context,
                &shouldFreeInfo);
 
    if (myPort != NULL)
    {
        // The port was successfully created.
        // Now create a run loop source for it.
        rlSource = CFMessagePortCreateRunLoopSource(NULL, myPort, 0);
 
        if (rlSource)
        {
            // Add the source to the current run loop.
            CFRunLoopAddSource(CFRunLoopGetCurrent(), rlSource, kCFRunLoopDefaultMode);
 
            // Once installed, these can be freed.
            CFRelease(myPort);
            CFRelease(rlSource);
        }
    }
 
    // Create the thread and continue processing.
    MPTaskID        taskID;
    return(MPCreateTask(&ServerThreadEntryPoint,
                    (void*)myPortName,
                    kThreadStackSize,
                    NULL,
                    NULL,
                    NULL,
                    0,
                    &taskID));
}
```

安装端口并启动线程后，主线程可以在等待线程 check in 时继续其常规执行。当 check-in 消息到达时，它将被分派到主线程的 `MainThreadResponseHandler`函数，如下面代码所示，此函数提取工作线程的端口名称，并为将来的通信创建管道。

```
// Listing 3-18  Receiving the checkin message

#define kCheckinMessage 100
 
// Main thread port message handler
CFDataRef MainThreadResponseHandler(CFMessagePortRef local,
                    SInt32 msgid,
                    CFDataRef data,
                    void* info)
{
    if (msgid == kCheckinMessage)
    {
        CFMessagePortRef messagePort;
        CFStringRef threadPortName;
        CFIndex bufferLength = CFDataGetLength(data);
        UInt8* buffer = CFAllocatorAllocate(NULL, bufferLength, 0);
 
        CFDataGetBytes(data, CFRangeMake(0, bufferLength), buffer);
        threadPortName = CFStringCreateWithBytes (NULL, buffer, bufferLength, kCFStringEncodingASCII, FALSE);
 
        // You must obtain a remote message port by name.
        messagePort = CFMessagePortCreateRemote(NULL, (CFStringRef)threadPortName);
 
        if (messagePort)
        {
            // Retain and save the thread’s comm port for future reference.
            AddPortToListOfActiveThreads(messagePort);
 
            // Since the port is retained by the previous function, release
            // it here.
            CFRelease(messagePort);
        }
 
        // Clean up.
        CFRelease(threadPortName);
        CFAllocatorDeallocate(NULL, buffer);
    }
    else
    {
        // Process other messages.
    }
 
    return NULL;
}
```

配置主线程后，剩下的唯一事情就是新创建的工作线程创建自己的端口并 check in 。下面代码展示了工作线程的入口函数。 该函数提取主线程的端口名称，并使用它创建一个返回主线程的远程连接。 然后，该函数为自己创建一个本地端口，在该线程的 run loop 上安装该端口，并向包含本地端口名称的主线程发送一个 check-in 消息。

```
// Listing 3-19  Setting up the thread structures

OSStatus ServerThreadEntryPoint(void* param)
{
    // Create the remote port to the main thread.
    CFMessagePortRef mainThreadPort;
    CFStringRef portName = (CFStringRef)param;
 
    mainThreadPort = CFMessagePortCreateRemote(NULL, portName);
 
    // Free the string that was passed in param.
    CFRelease(portName);
 
    // Create a port for the worker thread.
    CFStringRef myPortName = CFStringCreateWithFormat(NULL, NULL, CFSTR("com.MyApp.Thread-%d"), MPCurrentTaskID());
 
    // Store the port in this thread’s context info for later reference.
    CFMessagePortContext context = {0, mainThreadPort, NULL, NULL, NULL};
    Boolean shouldFreeInfo;
    Boolean shouldAbort = TRUE;
 
    CFMessagePortRef myPort = CFMessagePortCreateLocal(NULL,
                myPortName,
                &ProcessClientRequest,
                &context,
                &shouldFreeInfo);
 
    if (shouldFreeInfo)
    {
        // Couldn't create a local port, so kill the thread.
        MPExit(0);
    }
 
    CFRunLoopSourceRef rlSource = CFMessagePortCreateRunLoopSource(NULL, myPort, 0);
    if (!rlSource)
    {
        // Couldn't create a local port, so kill the thread.
        MPExit(0);
    }
 
    // Add the source to the current run loop.
    CFRunLoopAddSource(CFRunLoopGetCurrent(), rlSource, kCFRunLoopDefaultMode);
 
    // Once installed, these can be freed.
    CFRelease(myPort);
    CFRelease(rlSource);
 
    // Package up the port name and send the check-in message.
    CFDataRef returnData = nil;
    CFDataRef outData;
    CFIndex stringLength = CFStringGetLength(myPortName);
    UInt8* buffer = CFAllocatorAllocate(NULL, stringLength, 0);
 
    CFStringGetBytes(myPortName,
                CFRangeMake(0,stringLength),
                kCFStringEncodingASCII,
                0,
                FALSE,
                buffer,
                stringLength,
                NULL);
 
    outData = CFDataCreate(NULL, buffer, stringLength);
 
    CFMessagePortSendRequest(mainThreadPort, kCheckinMessage, outData, 0.1, 0.0, NULL, NULL);
 
    // Clean up thread data structures.
    CFRelease(outData);
    CFAllocatorDeallocate(NULL, buffer);
 
    // Enter the run loop.
    CFRunLoopRun();
}
```

一旦进入其 run loop，发送到该线程端口的所有未来事件都由 `ProcessClientRequest` 函数处理。 该函数的实现取决于线程的工作类型，这里没有显示。