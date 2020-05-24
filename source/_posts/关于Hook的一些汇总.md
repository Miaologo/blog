---
title: 关于Hook的一些汇总
date: 2020-05-14 19:57:24
tags:
	- ASM
	- iOS
categories:
    - Jason
---



iOS 中方法替换（就是我们常说的Hook），是一种超能力，通过它可以做很多我们意想不到的事情。比如早起比较常用的 `Aspects` 框架，方法打印记录等

比如iOS原生的主线程方法调用检测就是进行方法替换和检测的

> 前言：关于方法hook，在iOS中已经提供了很多第三方相关的框架，如Aspect等，相关的轮子和优化方式也在层出不穷的出现中

# _objc_msgForward

iOS开发都知道Objective-C对象的方法调用基本都是通过 objc_msgSend 中心调用的；如果遇到method不存在就会进行消息转发，转发的三部曲，这个大家都基本了解。

我们来回顾下runtime的消息转发机制：

```
1. 调用resolveInstanceMethod:方法 (或 resolveClassMethod:)。允许用户在此时为该 Class 动态添加实现。如果有实现了，则调用并返回YES，那么重新开始objc_msgSend流程。这一次对象会响应这个选择器，一般是因为它已经调用过class_addMethod。如果仍没实现，继续下面的动作。

2. 调用forwardingTargetForSelector:方法，尝试找到一个能响应该消息的对象。如果获取到，则直接把消息转发给它，返回非 nil 对象。否则返回 nil ，继续下面的动作。注意，这里不要返回 self ，否则会形成死循环。

3. 调用methodSignatureForSelector:方法，尝试获得一个方法签名。如果获取不到，则直接调用doesNotRecognizeSelector抛出异常。如果能获取，则返回非nil：创建一个 NSlnvocation 并传给forwardInvocation:。

4. 调用forwardInvocation:方法，将第3步获取到的方法签名包装成 Invocation 传入，如何处理就在这里面了，并返回非ni。

5. 调用doesNotRecognizeSelector: ，默认的实现是抛出异常。如果第3步没能获得一个方法签名，执行该步骤。
```

因此我们就可以通过方法替换class_replaceMethod 将你自定义的方法替换为 _objc_msgForward 原生方法

利用OC的消息转发机制，选择了合适的时机，进行打桩。 相较于传统的Swizzle方法，这种方法打主桩，是有可行性的。 并且在ForwardInvocation: 处理，虽然相较其余两个转发机制调用的方法的消耗大，但是更灵活一些，最切合问题

比较典型的第三库有 Aspects，AnyMethodLog

## [AnyMethodLog](https://github.com/qhd/ANYMethodLog)



## [Aspects](https://github.com/steipete/Aspects)



## 弊端

自己经常hook的同学可能会发现，在hook时，会出现调用循环的问题。

无论是AnyMethodLog 和 Aspects 都无法同时hook 父类和子类的同一个方法到一个相同的IMP上。为什么呢？

思考一下为什么会出现循环调用？ 那必定是，调用方又被调用者调用了一次，在iOS Hook 中，如果我们hook 了 父类和子类的同一个方法，让他们拥有相同的实现，就会出现这种问题。

> [基于桥的全量方法Hook方案 - 探究苹果主线程检查实现](http://satanwoo.github.io/2017/09/24/mainthreadchecker1/) 假设我们现在对UIView、UIButton都Hook了initWithFrame:这个方法，在调用[[UIView alloc] initWithFrame:]和[[UIButton alloc] initWithFrame:]都会定向到C函数qhd_forwardInvocation中，在UIView调用的时候没问题。但是在UIButton调用的时候，由于其内部实现获取了super initWithFrame:，就产生了循环定向的问题。

Aspects 中，Hook 之前，是要对能否hook 进行检查了，对于类，有严格的限制，对于实例则没有限制。

类为什么要限制，上面已经阐释了，那么实例为什么可以呢？

这就是 实例Hook 实现方式所产生的结果。

**来理一下实例hook怎么实现的：**

1. 生成子类
2. hook 子类的forwardInvocation(这是一系列操作，不过这个尤为重要)
3. 对实例的类信息进行伪装

如果我们有 ClassA 的 实例 a, SubClassA 的 实例 suba. 对他们进行hook `viewdidload` 方法, 那么会生成两个子类，我们记为prefix_ClassA, prefix_SubClassA,我们对forwardInvocation IMP的替换，实际上是在这两个类上进行的。

当方法调用时: suba -> forwardInvocation(我们替换的IMP) ->self viewdidload(SubClassA 的IMP) -> super viewdidload(ClassA的实现) 这显然不会导致循环的问题。

##参考文章

[iOS Hook 框架 AnyMethodLog与Aspects分析](https://juejin.im/post/5a313c11f265da433562c345)

# 汇编 Hook

## 汇编基础


### ARM64 伪指令

#### .text

`.text` 用于声明以下是代码段。

#### .align

`.align` 用于将指令对齐到内存地址，对齐位置为参数的 2 的幂次方。 以 `.align 10` 举例，在没有添加对齐时：

```text
func_1:
ret
func_2:
ret
```

得到的结果，两个指令构成了连续的地址： ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-1.90fdbc4b.png)

在 `func_1` 和 `func_2` 之间加入 `.align 10`：

```text
func_1:
ret
.align 10
func_2:
ret
```

得到如下结果，如果 `.align` 只是个普通的指令，那 `func_2` 的 `ret` 对应地址应当为 `0x0000000100008060`，这里却变成了 `0x000000010000840`，`0x000000010000840`刚好是 2^10 的倍数。 ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-2.d1447eed.png)

在本文中，会有一个妙用点，我们在 `func_1` 也加入 `.align 10`： ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-3.77e22018.png) 可以看到 `0x8800 - 0x8400 = 0x400`，0x400 刚好是 2^10，这说明我只要知道 `func_2` 的 `ret` 指令内存地址，就可以得到 `func_1` 的 `ret` 指令地址。

#### [#](https://blog.dianqk.org/2020/05/11/trampolinehook-study-notes/#rept-和-endr).rept 和 .endr

`.rept` 和 `.endr` 是一组循环伪指令。

```text
func_1:
.rept 5
add x0, x0, x1
.endr
ret
```

上述指令得到内容如下： ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-4.8d465430.png) 生成了 5 个连续的 `add` 指令。

#### 标签

如下代码：

```text
_my_label:
add x0, x0, x1
```

`_my_label` 会指向 `add x0, x0, x1` 的地址，用于辅助标记命令地址。

#### .globl

`.globl` 可以让一个标签对链接器可见，可以供其他链接对象模块使用。 换句话说 `.global _my_func` 可以在代码中调用到 `my_func`。

### ARM64 指令

#### nop

什么也不做的指令，愣一下。没什么用途，但可以偏移指令地址。 ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-5.4799e6e3.png) 可以看到 nop 也占了 4 个字节，原本 `0000000100013fe` 对应的指令应当是 ret。

#### sub

减法指令，可以拿来做一些地址偏移计算，用法如下：

```text
sub x1, x1, #0x8 ;相当于 x1 = x1 - 0x8
sub x2, x1, x2 ;相当于 x2 = x1 - x2 
```

#### mov

赋值指令，用法如下：

```text
mov x1, x0 ;相当于 x1 = x0
```

#### str 和 stp

两个入栈指令，stp 可以同时操作两个寄存器。

这里涉会及到一些寻址的格式，有 3 种方式：

```text
[x10, #0x10] ;从 x10 + 0x10 的地址取值
[sp, #-16]! ;从 sp - 16 地址取值，取值完后在把 sp 向低地址移 -16 字节，即开辟一段新栈空间
[sp], #16 ;从 sp 地址取值，取值完后在把 sp 向高地址偏移 -16 字节，即释放一些栈空间
```

结合上面几种寻址方式，搭配入栈指令，用法如下：

```text
str x8, [sp, #-16]! ;将 x8 存到 sp - 16 的位置，并将 sp -= 16
stp x4, x5, [sp, #-16]! ;将 x4 x5 的值存到 sp - 16 的位置，并将 sp -= 16
```

#### ldr 和 ldp

两个出栈指令，ldp 可以同时操作两个寄存器，用法如下：

```text
ldr x8, [sp], #16 ;将 sp 位置的值取出来，存入 x8 中，并将 sp += 16
ldp x4, x5, [sp], #16 ;将 sp 位置的值取出来，存入 x4 x5 中，并将 sp += 16
```

#### br

跳转指令，直接跳转到指定地址，跳转完不返回。有些类似在一个函数末尾调用了另外的函数。用法如下：

```text
br x10 ;跳转到 x10 中保存的地址
```

#### bl

跳转指令，将 bl 的下一个指令地址保存到 lr 寄存器，然后跳转到指定地址，因为将下一个指令保存到 lr 了，所以跳转完会回来执行下一个指令。用法如下：

```text
mov x0, x1
bl 0x100221232 ;跳转到 0x100221232，执行完毕回来执行下一条指令
sub x0, x0, 0x10
```

#### blr

跳转指令，和 bl 类似，但可以使用动态地址，可以跳转到寄存器的值保存的地址，用法如下：

```text
mov x8, 0x100221232
blr x8
```

#### ret

返回指令，子程序（函数调用）返回指令，返回到寄存器 lr 保存的地址。

### 函数、汇编、寄存器

汇编指令是对寄存器和栈进行各种操作，如何对应到我们日常编写的函数是关键。 寄存器相当于全局变量，在汇编中没有传入参数这样的形式，但我们可以用寄存器进行传递参数。为此，我们进行了一系列约定：

- x0 - x7：用于传递子程序参数和结果，使用时不需要保存，多余参数采用堆栈传递，子程序返回结果写入到 x0
- x8：用于保存子程序返回地址
- x9 - x15：临时寄存器
- x16 - x17：子程序内部调用寄存器
- x18：平台寄存器，它的使用与平台相关
- x19 - x28：临时寄存器
- x29：帧指针寄存器 fp（栈底指针），用于连接栈帧
- x30：链接寄存器 lr，保存了子程序返回的地址
- x31：堆栈指针寄存器 sp

连续调用多个子程序会面临寄存器不够用的问题，大家都要用 lr 作为返回地址，都要用 x0 传递参数。 栈来了！如果后面的操作会对寄存器有修改，先入栈保存起来，等执行完相关操作，在出栈读出来就好了。 比如 lr 是必须入栈的寄存器（当然你脾气硬，不怕其他寄存器被修改，保存到其他寄存器也可以，但一定别这样写，万一啥时候被修改了呢），如果没有保存 lr，调用子程序后，lr 不是当初的 lr 了，会找不到返回的位置。

如下汇编没有保存 lr：

```text
add_func:
add x0, x0, x1
ret

.globl _my_add_func
_my_add_func: ;没有保存 lr，调用后回不去了
bl add_func
ret
```

调用后：

```text
FOUNDATION_EXTERN int my_add_func(int x, int y);

printf("开始执行 my_add_func\n");
int result = my_add_func(1, 1);
printf("执行 my_add_func 结果: %d\n", result);
```

得到的输出：

```text
开始执行 my_add_func
```

为 `_my_add_func` 增加个 lr 入栈就行了：

```text
_my_add_func:
str lr, [sp, #-16]!
bl add_func
ldr lr, [sp], #16
ret
```

此时得到的输出为：

```text
开始执行 my_add_func
执行 my_add_func 结果: 2
```

更复杂的调用场景，需要将更多的寄存器入栈，避免寄存器内容被污染。

## Hook框架

### iOS开源代码推荐 - HookZz

本周推荐代码库 [HookZz](https://github.com/jmpews/HookZz) . 这是一个做安全的大神同事写的库。有兴趣同学可以深入研究，绝对牛逼，绝对**黑科技**。利用的好可以做很多事情。

文章详细介绍: [HookZz框架 · jmpews](https://jmpews.github.io/2017/08/01/pwn/HookZz%E6%A1%86%E6%9E%B6/)

Start: [jmpews/HookZz](https://github.com/jmpews/HookZz/blob/master/docs/hookzz-docs.md)

这里简单介绍下其功能、原理、和可以怎么利用。

**功能**： 让代码可以被Hook， 不仅仅限于oc方法，包括 OC/C/C++/Swift。只要是被编译成二进制的点。

**原理**： 基于对编译好的汇编方法替换首、尾等指令，的方式，让所有的方法都可以被hook.

**优点:** 脱离语言的限制，让所有方法都可以hook.

**用处**：

\1. 适合用于做代码覆盖率。

\2. 对 c/c++/swift等代码做可patch支持。

## 参考文章

* [TrampolineHook 学习笔记](https://blog.dianqk.org/2020/05/11/trampolinehook-study-notes/)
* [基于桥的全量方法 Hook 方案（2） - 全新升级](http://satanwoo.github.io/2020/04/22/NewBridgeHook/)
* [基于桥的全量方法 Hook 方案（3）- TrampolineHook](http://satanwoo.github.io/2020/04/26/TrampolineHookOpenSource/)
* [TrampolineHook](https://github.com/SatanWoo/TrampolineHook)
* [Implementing imp_implementationWithBlock()](https://landonf.org/code/objc/imp_implementationWithBlock.20110413.html)
* [HOOKZZ框架](http://jmpews.github.io/2017/08/01/pwn/HookZz框架/)
* [Dobby](https://github.com/jmpews/Dobby)
* [iOS源码解析: 聊一聊iOS中的hook方案](https://juejin.im/post/5e3ec8f2518825493f6cd5fa)





