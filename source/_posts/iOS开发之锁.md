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

## 优先级反转

简单从字面上来说，就是低优先级的任务先于高优先级的任务执行了，优先级搞反了。那在什么情况下会生这种情况呢？

假设三个任务准备执行，A，B，C，优先级依次是A>B>C；

首先：C处于运行状态，获得CPU正在执行，同时占有了某种资源；

其次：A进入就绪状态，因为优先级比C高，所以获得CPU，A转为运行状态；C进入就绪状态；

第三：执行过程中需要使用资源，而这个资源又被等待中的C占有的，于是A进入阻塞状态，C回到运行状态；

第四：此时B进入就绪状态，因为优先级比C高，B获得CPU，进入运行状态；C又回到就绪状态；

第五：如果这时又出现B2，B3等任务，他们的优先级比C高，但比A低，那么就会出现高优先级任务的A不能执行，反而低优先级的B，B2，B3等任务可以执行的奇怪现象，而这就是优先反转。



## @synchronized(obj)

`@synchronized(id obj){}`锁的是对象`obj`，使用该锁的时候，底层是对象计算出来的值作为`key`，生成一把锁，不同的资源的读写可以使用不同`obj`作为锁对象。



## NSLock

```objective-c
//协议NSLocking
@protocol NSLocking

- (void)lock;
- (void)unlock;

@end

@interface NSLock : NSObject <NSLocking> {
@private
    void *_priv;
}
- (BOOL)tryLock;//尝试加锁
- (BOOL)lockBeforeDate:(NSDate *)limit;//在某个日期前加锁，
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
@end
```



## dispatch_queue



## dispatch_semaphore

**dispatch_semaphore_signal**

当我们有大量任务需要并发执行，而且同时最大并发量为5个线程，这样子又该如何控制呢？`dispatch_semaphore`信号量正好可以满足我们的需求。`dispatch_semaphore`可以控制并发线程的数量，当设置为1时，可以作为同步锁来用，设置多个的时候，就是异步并发队列



## dispatch_barrier_async

这个函数传入的并发队列必须是通过`dispatch_queue_create`创建，如果传入的是一个串行的或者全局并发队列，这个函数便等同于`dispatch_async`的效果。



## NSRecursiveLock

```objective-c
- (BOOL)tryLock;//尝试加锁
- (BOOL)lockBeforeDate:(NSDate *)limit;//日期前加锁
```





## NSConditionLock

`NSConditionLock`是可以实现多个子线程进行线程间的依赖，A依赖于B执行完成，B依赖于C执行完毕则可以使用`NSConditionLock`来解决问题

```objective-c
@property (readonly) NSInteger condition;//条件值
- (void)lockWhenCondition:(NSInteger)condition;//当con为condition进行锁住
//尝试加锁
- (BOOL)tryLock;
//当con为condition进行尝试锁住
- (BOOL)tryLockWhenCondition:(NSInteger)condition;
//当con为condition进行解锁
- (void)unlockWithCondition:(NSInteger)condition;
//NSDate 小余 limit进行 加锁
- (BOOL)lockBeforeDate:(NSDate *)limit;
//条件为condition 在limit之前进行加锁
- (BOOL)lockWhenCondition:(NSInteger)condition beforeDate:(NSDate *)limit;

```



条件锁的使用，在`lockWhenCondition:(NSInteger)condition`的条件到达的时候才能进行正常的加锁和`unlockWithCondition:(NSInteger)condition`解锁，否则会阻塞线程。







## NSCondition

```objective-c
- (void)wait;//等待
- (BOOL)waitUntilDate:(NSDate *)limit;
- (void)signal;//唤醒一个线程
- (void)broadcast;//唤醒多个线程
```



## pthread_mutex

**pthread_mutex_init(pthread_mutex_t \* mutex,const pthread_mutexattr_t attr);**初始化锁变量mutex。attr为锁属性，NULL值为默认属性。

**pthread_mutex_lock(pthread_mutex_t*mutex);**加锁

**pthread_mutex_tylock(pthread_mutex_t*mutex);**加锁，但是与2不一样的是当锁已经在使用的时候，返回为EBUSY，而不是挂起等待。

**pthread_mutex_unlock(pthread_mutex_t*mutex);**释放锁

**pthread_mutex_destroy(pthread_mutex_t**mutex);**使用完后释放



## 读写锁 pthread_rwlock

```objective-c
//初始化读写锁
int pthread_rwlock_init(pthread_rwlock_t * __restrict,
		const pthread_rwlockattr_t * _Nullable __restrict)
//读上锁
pthread_rwlock_rdlock(pthread_rwlock_t *)
//尝试加锁读
pthread_rwlock_tryrdlock(pthread_rwlock_t *)
//尝试加锁写
int pthread_rwlock_trywrlock(pthread_rwlock_t *)
//写入加锁
pthread_rwlock_wrlock(pthread_rwlock_t *)
//解锁
pthread_rwlock_unlock(pthread_rwlock_t *)
//销毁锁属性
pthread_rwlockattr_destroy(pthread_rwlockattr_t *)
//销毁锁
pthread_rwlock_destroy(pthread_rwlock_t * )

```

`pthread_rwlock_t`使用很简单，只需要在读之前使用`pthread_rwlock_rdlock`，读完解锁`pthread_rwlock_unlock`,写入前需要加锁`pthread_rwlock_wrlock`，写入完成之后解锁`pthread_rwlock_unlock`，任务都执行完了可以选择销毁`pthread_rwlock_destroy`或者等待下次使用。







## OSSpinLock

**OSSpinLock**自旋锁，性能最高的锁。原理很简单，就是一直 **do while**忙等。它的缺点是当等待时会消耗大量 CPU 资源，所以它不适用于较长时间的任务。 不过最近YY大神在自己的博客[不再安全的 OSSpinLock](https://link.juejin.im/?target=https%3A%2F%2Fblog.ibireme.com%2F2016%2F01%2F16%2Fspinlock_is_unsafe_in_ios%2F)中说明了**OSSpinLock**已经不再安全，请大家谨慎使用

```objective-c
//初始化 一般是0，或者直接数字0也是ok的。
#define	OS_SPINLOCK_INIT    0
//锁的初始化
OSSpinLock lock = OS_SPINLOCK_INIT;
//尝试加锁
bool ret = OSSpinLockTry(&lock);
//加锁
OSSpinLockLock(&lock);
//解锁
OSSpinLockUnlock(&lock);

```



## os_unfair_lock

`os_unfair_lock`被系统定义为低级锁，一般低级锁都是闲的时候在睡眠，在等待的时候被内核唤醒，目的是替换已弃用的`OSSpinLock`，而且必须使用`OS_UNFAIR_LOCK_INIT`来初始化，加锁和解锁必须在相同的线程，否则会中断进程，使用该锁需要系统在`__IOS_AVAILABLE(10.0)`，锁的数据结构是一个结构体

```objective-c
OS_UNFAIR_LOCK_AVAILABILITY
typedef struct os_unfair_lock_s {
	uint32_t _os_unfair_lock_opaque;
} os_unfair_lock, *os_unfair_lock_t;

```



## atomic 原子操作

给属性添加`atmoic`修饰，可以保证属性的`setter`和`getter`都是原子性操作，也就保证了`setter`和`getter`的内部是线程同步的。 原子操作是最终调用了`static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy) objc-accessors.mm 48行`，我们进入到函数内部

```objective-c
//设置属性原子操作
void objc_setProperty_atomic(id self, SEL _cmd, id newValue, ptrdiff_t offset)
{
    reallySetProperty(self, _cmd, newValue, offset, true, false, false);
}
//非原子操作设置属性
void objc_setProperty_nonatomic(id self, SEL _cmd, id newValue, ptrdiff_t offset)
{
    reallySetProperty(self, _cmd, newValue, offset, false, false, false);
}

static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy)
{//偏移量等于0则是class指针
    if (offset == 0) {
        object_setClass(self, newValue);
        return;
    }
//其他的value
    id oldValue;
    id *slot = (id*) ((char*)self + offset);

    if (copy) {
    //如果是copy 用copyWithZone:
        newValue = [newValue copyWithZone:nil];
    } else if (mutableCopy) {
        //mutableCopy则调用mutableCopyWithZone:
        newValue = [newValue mutableCopyWithZone:nil];
    } else {
    //如果赋值和原来的相等 则不操作
        if (*slot == newValue) return;
        newValue = objc_retain(newValue);
    }

    if (!atomic) {//非原子操作 直接赋值
        oldValue = *slot;
        *slot = newValue;
    } else {//原子操作 加锁
    //锁和属性是一一对应的->自旋锁
        spinlock_t& slotlock = PropertyLocks[slot];
        slotlock.lock();
        oldValue = *slot;
        *slot = newValue;//赋值
        slotlock.unlock();//解锁
    }
    objc_release(oldValue);
}


id objc_getProperty(id self, SEL _cmd, ptrdiff_t offset, BOOL atomic) {
    if (offset == 0) {
        return object_getClass(self);
    }

    // Retain release world
    id *slot = (id*) ((char*)self + offset);
    if (!atomic) return *slot;//非原子操作 直接返回值
        
    // Atomic retain release world
	//原子操作 加锁->自旋锁
    spinlock_t& slotlock = PropertyLocks[slot];
    slotlock.lock();//加锁
    id value = objc_retain(*slot);
    slotlock.unlock();//解锁
    
    // for performance, we (safely) issue the autorelease OUTSIDE of the spinlock.
    return objc_autoreleaseReturnValue(value);
}

//以属性的地址为参数计算出key ，锁为value
StripedMap<spinlock_t> PropertyLocks;

```

从源码了解到设置属性读取是`self`+属性的偏移量，当`copy`或`mutableCopy`会调用到`[newValue copyWithZone:nil]`或`[newValue mutableCopyWithZone:nil]`，如果新旧值相等则不进行操作，非原子操作直接赋值，原子操作则获取`spinlock_t& slotlock = PropertyLocks[slot]`进行加锁、赋值、解锁操作。而且`PropertyLocks`是一个类，类有一个数组属性，使用`*p`计算出来的值作为`key`。

由上面得知`atmoic`仅仅是对方法`setter()`和`getter()`安全，对成员变量不保证安全，对于属性的读写一般使用`nonatomic`，性能好，`atomic`读取频率高的时候会导致线程都在排队，浪费CPU时间



## 几种锁性能对比

大概使用者几种锁分别对卖票功能进行了性能测试， 性能分别1万次、100万次、1000万次锁花费的时间对比，单位是秒。(仅供参考，不同环境时间略有差异)

|     锁类型      |  1万次   | 100万次  | 1000万次  |
| :-------------: | :------: | :------: | :-------: |
| pthread_mutex_t | 0.000309 | 0.027238 | 0.284714  |
| os_unfair_lock  | 0.000274 | 0.028266 | 0.285685  |
|   OSSpinLock    | 0.030688 | 0.410067 | 0.437702  |
|   NSCondition   | 0.005067 | 0.323492 | 1.078636  |
|     NSLock      | 0.038692 | 0.151601 | 1.322062  |
| NSRecursiveLock | 0.007973 | 0.151601 | 1.673409  |
|  @synchronized  | 0.008953 | 0.640234 | 2.790291  |
| NSConditionLock | 0.229148 | 5.325272 | 10.681123 |
|    semaphore    | 0.094267 | 0.415351 | 24.699100 |
|   SerialQueue   | 0.213386 | 9.058581 | 50.820202 |



## 自旋锁&互斥锁比较

自旋锁和互斥锁各有优劣，

* 代码执行频率高，CPU充足，可以使用自旋锁
* 频率低，代码复杂则需要互斥锁。

#### 自旋锁

- 自旋锁在等待时间比较短的时候比较合适
- 临界区代码经常被调用，但竞争很少发生
- CPU不紧张
- 多核处理器

#### 互斥锁

- 预计线程等待时间比较长
- 单核处理器
- 临界区IO操作
- 临界区代码比较多、复杂，或者循环量大
- 临界区竞争非常激烈

## 



https://juejin.im/post/5d22f34de51d4555fd20a3c5

https://juejin.im/post/5d395318f265da1b8608ca98