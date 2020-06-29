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
* [Hook objc_msgSend -- 从 0.5 到 1](https://blog.gocy.tech/2019/07/08/hook-msgSend-advance/)



# APM

* 美图的开源性能工具 [MTHawkeye](https://github.com/meitu/MTHawkeye)

[蘑菇街移动端全链路跟踪保障体系](https://link.jianshu.com?t=http%3A%2F%2Fwww.infoq.com%2Fcn%2Fpresentations%2Fmobile-terminal-full-link-tracking-and-security-system)

[美团外卖移动端性能监测体系实现](https://link.jianshu.com?t=http%3A%2F%2Fmp.weixin.qq.com%2Fs%2FMwgjpHj_5RaG74Z0JjNv5g)

[微信读书 iOS 质量保证及性能监控](https://link.jianshu.com?t=https%3A%2F%2Fwereadteam.github.io%2F2016%2F12%2F12%2FMonitor%2F)

[网易NeteaseAPM iOS SDK技术实现分享](https://link.jianshu.com?t=http%3A%2F%2Fwww.infoq.com%2Fcn%2Farticles%2Fnetease-ios-sdk-neteaseapm-technology-share)

[阿里百川码力APP监控来了 重量级选手进入APM市场](https://link.jianshu.com?t=http%3A%2F%2Fwww.imooc.com%2Farticle%2F14205%3Fblock_id%3Dtuijian_wz)

[APM最佳实践系列文章专题合辑](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fjoy0304%2FJoy-Blog%2Fblob%2Fmaster%2FiOS%20Collection.md)

[手机淘宝：亿级用户APP的快速运维交付实践](https://link.jianshu.com?t=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzAxNDEwNjk5OQ%3D%3D%26mid%3D2650400312%26idx%3D1%26sn%3Dce8468991c70ab2e06634f59cd2b6865%26chksm%3D83952e20b4e2a736f701853a483da535312a258a56ca87d65b8ef77e8cf012dab9145659a0aa%26scene%3D0%26key%3D459eeebe1b51063320bc30b7024529048032de1a4d3a8e7cf01dbfc995da8f74fe85688c8be0471b1fdcb82d9b875d163a62f42e9ca04946e2c899194097fb93632ca7790f6fb7395d897442b9272213%26ascene%3D0%26uin%3DMTY3NzkzNjI0NA%3D%3D%26devicetype%3DiMac%2BMacBookPro12%2C1%2BOSX%2BOSX%2B10.12.2%2Bbuild(16C67)%26version%3D12020010%26nettype%3DWIFI%26fontScale%3D100%26pass_ticket%3DJE5tAT8H%2BfKdFzHQq72mWMIv%2BitHWOqOma3xmX5OeGGPWz2mPXxz3kaQE1WSKJlw)

[移动端监控体系之技术原理剖析](https://www.jianshu.com/p/8123fc17fe0e)

* https://github.com/Tencent/matrix



