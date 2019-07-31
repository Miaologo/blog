---
title: iOS开发舆图
date: 2019-07-31 12:26:53
tags:
	- iOS
	- 面试
---

# iOS开发知识统筹总览

Collaborative  list of  open source apps

Appcoda.com

Stanford university cs 193p

Raywender 

## iOS基础

### 布局

* AutoLayout
* Snapkit
* Simplex 算法
* Flexbox
* AsyncDisplayKit
* Masonry

### 列表

* UITableView
* UICollectionView
* Expandable Cell
* Header
* PlaceHolder
* CollectionView Layout
* LGListKit

### 图表

* charts

### 图形

* Core Graphics
* Gif
* 图片
* 图片处理
* PDF

### 多媒体

* video

* audio

* Game

* ARKit

* Core Graphics

* Core Image

* Metal

* GPUImage

* SceneKit

* Image I/O

* SiriKit

### 动画

* Core Animation
* UIBezierPath
* Pop
* Spring
* Lottie ios

### App Services

* 定位
* 通知
* webview
* web socket
* icloud
* watchKit
* 照片
* 相机
* 文件管理
* 手势
* 键盘
* 蓝牙
* NFC
* Force Touch
* 地图
* Apple TV
* Email
* Passbook
* 支付
* 权限
* Core Motion
* iBeacon
* 硬件

## iOS 系统

* 《Max OSX and iOS internals》
* XNU
* dyld
* Mach-O

### Foundation

#### 界面

* UIViewController

* UINavigationController

* UIScrollView

* UIButton

* UILabel

* UITableView & UICollectionView

* UINavigationBar

* UITabbar

* UISegmentControl

* UIPageControl

* UITextview

* Core text

* UISider

* 下拉刷新

* 登陆

* 进度条

* 日历

* 卡片

* 自定义控件

* 浮层

* 弹窗

* 标签

* 通知中心

  

## 编程语言

### Swift

* 集合
* 结构体和类
* 字符串
* 正则表达式
* 枚举
* 属性
* Optional
* Mirror
* 泛型
* defer
* Never
* Result
* 类型擦除
* 元编程
* 动态性
* IDE
* Swift NIO
* Swift UI
* Combine
* Foundation Builder
* 命令行
* REPL
* Docker Swift
* 时间
* 数学
* Swift 源码

### Objective C

* 字符串
* 集合
* 值对象
* method
* KVC
* KVO
* Runtime
* Block
* metal class

### 结构

* SOLID
* MVVM
* VIPER
* CMediator
* router 路由
* 消息总线

### 编程范式

* 函数式编程
* 面向协议编程
* 响应式编程（ReactiveCocoa，RxSwift）

### 设计模式

#### 原则

* 依赖导致
* 开闭原则

#### 模式

* 工厂方法
* 单例模式
* 观察者模式
* 备忘录模式
* 外观模式
* 适配器模式

### 代码规范

* swiftSyntax
* SwiftLint
* OCLint
* Modern Code Review: A Case at Google

### 开发工具

#### Xcode

* wwdc
* Instrument

#### 编译

* LLVM
* Driver
* Clang
* LLVM IR
* 链接器



### 静态分析

* 方法跟踪
* FavxPas
* LSP（language server protocol）
* Coverity
* Inter
* Clang Static Analyzer
* Lib Clang
* Lib Tooling
* Clang Plugin
* AST matchers

### 包管理

* Swift package manager
* cocoapods
* carthage

## 开发完成

### debug

* In Appview Debugger
* MTHawkeye
* Dtrace
* LLDB
* Injection for xcode
* Lookin

### 集成

* Jenkins
* Fastlane
* Venom
* EasyBox
* Build
* Cmake
* Appstore  & Appstore connect
* 项目管理
* 包大小

### 测试

#### 单元测试

* BDD
* TDD
* XCtest
* kiwi
* Specta
* Expecta
* OCMock
* OHHttpstubs

A/B 测试

界面测试

代码覆盖率

自动化测试

## 上线后

### APM

* DoraemonKit
* Matrix
* GodEye
* 卡顿监控
* 内存监控
* 启动（fishhook, Messier）
* 电量

### Crash

* PLCrashReporter
* KSCrash
* Crash日志解析
* Watch Dog
* 后台崩溃
* 野指针
* buggle.qq.com

### 日志 log

* Swiffy Beaver
* Cocoa lvmberjack
* XLog
* Logan
* 去侵入埋点

### 动画化 patch

* waxPatch
* JSPatch
* DynamicCocoa
* OCS
* MABlockClosure
* Aspects
* OEcal
* Libffi
* Hybird
* JSBridge
* React Native
* Weex

## 计算机基础

### 汇编

* PC Assembly Book
* x64 Cheat Sheet
* Stanford CS107  to x86-64
* Call Convention
* Assembly
* ARM64
* C in ASM (ARM64) 系列
* mikeash.com
* Dissection objc_msgSend On ARM64

### 算法

* 链表
* KMP
* 二叉树
* 二叉排序树
* AVL 二叉平衡树
* B 树
* 队列
* Recursion 递归
* 动态规划
* Greedy 贪婪算法
* NP Complete
* Bit Manipulation
* Big-O notaion

### 数据结构

* stack
* Array
* List
* Map
* Multimap
* Set
* Multiset
* Graph
* Queue

### 内存

* 循环引用
* FBMemoryProfile
* 内存管理 MRC ARC
* 内存布局
* Tagged Pointer
* 引用计数
* 弱引用表
* 自动释放池 AutoreleasePool
* 内存映射
* MMKV
* OOM

### 多线程

* NSThread
* GCD
* Posix
* synchronized
* NSLock
* NSRecursiveLock
* Pthread_mutex
* oosplinlock
* 后台模式

### Mach-O

### Runloop

* CFRunloop
* NSRunloop
* NSMachPort
* NSPortMessage
* Matrix

## 通用知识

### 渲染

* Core Animation Pipeline
* Vsync & 双缓冲机制
* Skia
* Vulkan
* Blink Rendering
* OpenGL ES
* GPU

### 数据库

* SQLite swift
* Core Data
* Realm
* FMDB
* WCDB



### 网络

* TCP
* UDP
* DNS解析
* HTTP Protocol
* URL Protocol
* NetworkExtension
* 网络安全
* Charles
* mitmproxy
* Wireshark
* Multipart
* Alamofire
* NSURLSession
* OAuth2
* OAuth Swift
* Graph QL

### 数据表示

* JSON
* JSONModel
* YYModel
* JSONDecoder
* Simdjson
* CVS
* XML

### 人工智能

* TensorFlow
* BNNS
* MPSCNN
* Create ML
* Core ML
* ML Kit
* 图像识别
* YOLO

### 安全

* iOS 代码签名
* Crypto Swift
* Keychain Access
* Valet
* RNCryptor
* 混淆加固
* Encryption

### 服务器侧

* Vapor
* Petect
* Kitura
* Dock swift on arm

### 小程序

* taro

## 技术提升

### 手册

* Apple Develop Document
* Swift Programming Language

### 开源

* ios example .com
* Cocoa controls.com
* 开源团队，作者

### 网站

* NSHipster
* Objc.io
* Raywenderlich.com
* Mikeash.com
* Big nerd ranch

### Podcasts

* Debug
* App story
* Mobile Couch
* iPhreaks
* Fireside Swift

### 技术会议

* GMTC
* @swift
* try ! Swift  conference
* LLVM Developers Meeting

### 兴趣小组

* iOS 知识小集
* Swfit GG
* 老司机 iOS 周报



### 书籍

* 代码大全
* 程序员修炼之道
* 设计模式
* 人月神话
* 代码整洁之道
* 重构
* SICP
* the little schemer
* C++ Primer Plus
* 计算机组成与体系结构
* 深入理解计算机系统
* TCP/IP 详解 卷1 协议
* ARM Cortex-M3 与 Cortex-M4 权威指南
* iOS Programming
* Advanced Swift
* Functional Swift
* Swift in Depth
* 编程大师访谈录
* 编程人生：15位软件先驱访谈录
* 浪潮之巅
* Hard Drive
* 硅谷革命 成就苹果公司的疯狂往事
* 乔布斯传
* Effective Objective - C 2.0
* Objective-C 高级编程
* Practical Modern C++
* 全球科技通史
* 硅谷之谜
* 数学之美
* 编码
* 代码的未来



#具体文档

## 实例

学习 iOS 开发最好是从学习一个完整的 App 入手，GitHub上的[Open-Source iOS Apps](https://github.com/dkhamsing/open-source-ios-apps)
项目，收录了大量开源的完整 App 例子，比如 [Hacker News Reader](https://github.com/Dimillian/SwiftHN) 等已经上架了 App Store 的应用程序，所有例子都会标注是否上架 App Store的、所使用开发语言、推荐等级等信息，有利于进行选择学习。

开发一个完整的 App 也有最佳实践，这里有份[最佳实践](https://github.com/futurice/ios-good-practices)可以参考。

下面两个教程网站都会手把手通过实例教你怎么动手学习 iOS 各个知识点。

1. [AppCoda](https://www.appcoda.com/)
2. [Raywenderlich](https://www.raywenderlich.com/library)

## iOS 基础

完整开发了多个 App 后，为了更好、更快的掌握开发，你就会有需要了解更多 iOS 基础知识的诉求，包括列表的优化、高效界面布局开发、图表图形、图片处理、动画多媒体等等。

图形渲染 Metal 框架的学习可以参看下面四篇文章

1. [Metal](https://objccn.io/issue-18-2/)
2. [基于 Metal 的 ARKit 使用指南（上）](https://juejin.im/post/5a225ffcf265da432153daa4)
3. [基于 Metal 的 ARKit 使用指南（下）](https://juejin.im/post/59bb2a99f265da0650750e56)
4. [基于 Metal 的现代渲染技术](https://xiaozhuanlan.com/topic/6927418053)

## iOS 系统

iOS 基础学习到一定程度就需要了解 App 是如何在系统中工作的，系统提供了什么基础功能，提供了哪些界面控件等等。

扩展知识可以阅读下面四本书：

1. 《深入解析Mac OS X & iOS操作系统》
2. 《现代操作系统》
3. 《深入理解计算机系统》
4. 《程序员的自我修养》

## 编程语言

编程语言的学习可以参考官方手册，对于 Runtime 的扩展文章阅读：

1. [Objective-C 消息发送与转发机制原理](http://yulingtianxia.com/blog/2016/06/15/Objective-C-Message-Sending-and-Forwarding/)
2. [神经病院Objective-C Runtime入院第一天——isa和Class](https://halfrost.com/objc_runtime_isa_class/)（ https://halfrost.com/objc_runtime_isa_class/ ）
3. [神经病院 Objective-C Runtime 住院第二天——消息发送与转发](https://halfrost.com/objc_runtime_objc_msgsend/)
4. [神经病院 Objective-C Runtime 出院第三天——如何正确使用](https://halfrost.com/how_to_use_runtime/)

编程语言 Swift 推荐阅读书籍是《Swift 进阶》、《函数式Swift》。大量Swift Playground 可以了解 Swift 编程语言的特性，这里有份整理 [GitHub - uraimo/Awesome-Swift-Playgrounds: A List of Awesome Swift Playgrounds](https://github.com/uraimo/Awesome-Swift-Playgrounds) 。这份资料汇总了 SwiftUI 的资料 [GitHub - Juanpe/About-SwiftUI: Gathering all info published, both by Apple and by others, about new framework SwiftUI.](https://github.com/Juanpe/About-SwiftUI) 。这里有本在线书详细讲解了 Combine
[《Using Combine》](https://heckj.github.io/swiftui-notes/) 。

架构相关扩展阅读可以参看 [iOS 组件化相关讨论文章汇总 | KANGZUBIN](https://kangzubin.com/ios-component-articles/) 。

设计模式推荐书籍如下：

- 《设计模式 可复用面向对象软件的基础》
- 《Objective-C 编程之道：iOS设计模式解析》
- 《Head First 设计模式》
- 《大话设计模式》

## 开发工具

开发的代码多了，开发效率和开发质量的提升就越来越离不开开发工具了。

iOS 开发工具 Xcode、Instrument 的使用学习，推荐看苹果开发者大会 WWDC 的 Session 视频。

扩展阅读文章推荐：

1. [The Architecture of Open Source Application](http://www.aosabook.org/en/llvm.html)
2. [Writing AST matchers for libclang](https://manu343726.github.io/2017-02-11-writing-ast-matchers-for-libclang/)
3. [使用 OCLint 自定义 MVVM 规则](http://yulingtianxia.com/blog/2019/01/27/MVVM-Rules-for-OCLint/)
4. [iOS 增量代码覆盖率检测实践](https://mp.weixin.qq.com/s/vCzUNHyLfjQKF23Biq9z-g)

阅读书籍推荐：

1. 《Getting Started with LLVM Core Libraries》
2. 《Modern Compiler Implementation in C》
3. 《Compiler》

## 开发完成

当开发完成后就需要进行调试、持续化交付、测试。

LLDB 调试推荐先看[官方指南](https://developer.apple.com/library/archive/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/Introduction.html)，再看这篇[“与调试器共舞 - LLDB 的华尔兹”](https://objccn.io/issue-19-2/)。为了更好的调试体验扩展 LLDB 可以参看这篇文章[“How to Extend LLDB to Provide a Better Debugging Experience”](https://pspdfkit.com/blog/2018/how-to-extend-lldb-to-provide-a-better-debugging-experience/)。

另外，这个[网址](https://github.com/MattPD/cpplinks/blob/master/debugging.md)收录了各种调试资料。

持续化交付可以参看各大公司的实践，比如：

1. [知乎 iOS 客户端工程化工具 - Venom](https://zhuanlan.zhihu.com/p/69526642)
2. [百度App iOS工程化实践: EasyBox破冰之旅](https://mp.weixin.qq.com/s/Oa52PvsHw8wS-OvYb3ArZg)
3. [如何实现“持续集成”？淘系闲鱼把研发效率翻了个翻](https://mp.weixin.qq.com/s/6WAq_fM0znjO5eY12vjxFw)

测试扩展阅读推荐文章如下：

1. [TDD的iOS开发初步以及Kiwi使用入门](https://onevcat.com/2014/02/ios-test-with-kiwi/)
2. [Kiwi 使用进阶 Mock, Stub, 参数捕获和异步测试](https://onevcat.com/2014/05/kiwi-mock-stub-test/)
3. [Unit Testing](https://nshipster.com/unit-testing/)
4. [客户端自动化测试研究](https://tech.meituan.com/2017/06/23/mobile-app-automation.html)
5. [PICK一下，iOS自动化测试新方案出道](https://mp.weixin.qq.com/s/5rt-uxApK-MeKYn0eKLVcQ?from_safari=1&scene=40#wechat_redirect)
6. [爱奇艺基于AI的移动端自动化测试框架的设计](https://mp.weixin.qq.com/s?__biz=MzIzMzk2NDQyMw==&mid=2247488757&idx=1&sn=22465225abd30e56181ad68cdcb60e88&chksm=e8fcc21ddf8b4b0bc97126bb41d95ab8df24c9db20eb9b6efd36ba969ce2e1df000b702ef639&mpshare=1&scene=1&srcid=0221Rpn12CYSXPIE0D1ndrNi&pass_ticket=QDFhyHIsSuU8LkeDk3P%2Bsli%2FL%2BwfD5Y55dNIk2PcCwbawmrDVExKHNGlflrh0dhL#rd)

## 上线后

开发完成后，做完测试，就可以上线了。上线后还需要做大量监控保证用户使用 App 的高可用性和好体验。动态化保证发版灵活和问题的快速修复。

推荐的扩展阅读如下：

1. [iOS应用启动性能优化资料](https://everettjf.github.io/2018/08/06/ios-launch-performance-collection/)
2. [iOS启动时间优化](http://www.zoomfeng.com/blog/launch-time.html)
3. [Understanding and Analyzing Application Crash Reports](https://developer.apple.com/library/archive/technotes/tn2151/_index.html)
4. [PLCrashreporter源码分析其一](http://www.zoomfeng.com/blog/plcrashreporter-1.html)
5. [PLCrashreporter源码分析其二](http://www.zoomfeng.com/blog/plcrashreporter-2.html)
6. [How Not to Crash](https://inessential.com/hownottocrash)
7. [Logan：美团点评的开源移动端基础日志库](https://mp.weixin.qq.com/s/XM4bhncHzRFB7zMJa-g2-Q)
8. [Hook Objective-C Block with Libffi](http://yulingtianxia.com/blog/2018/02/28/Hook-Objective-C-Block-with-Libffi/)
9. [Hot or Not? The Benefits and Risks of iOS Remote Hot Patching](https://www.fireeye.com/blog/threat-research/2016/01/hot_or_not_the_bene.html)

## 计算机基础

经历多次 App 开发到上线后的过程，碰到问题，解决问题，越发觉得计算机基础的重要性。牢固的基础能有利于碰到问题时快速定位和解决。

推荐扩展阅读文章和资源如下：

1. [Algorithms and data structures in Swift, with explanations!](https://github.com/raywenderlich/swift-algorithm-club)
2. [iOS Memory Deep Dive](https://developer.apple.com/videos/play/wwdc2018/416/)
3. [iOS App Performance: Memory](https://developer.apple.com/videos/play/wwdc2012/242/)
4. [No pressure, Mon! Handling low memory conditions in iOS and Mavericks](http://newosxbook.com/articles/MemoryPressure.html)
5. [从零构建 Dispatch Queue](https://swift.gg/2017/09/07/friday-qa-2015-09-04-lets-build-dispatch_queue/)
6. [Threading Programming Guide(1)](http://yulingtianxia.com/blog/2017/08/28/Threading-Programming-Guide-1/)
7. [Threading Programming Guide(2)](http://yulingtianxia.com/blog/2017/09/17/Threading-Programming-Guide-2/)
8. [Threading Programming Guide(3)](http://yulingtianxia.com/blog/2017/10/08/Threading-Programming-Guide-3/)
9. [Swift 中的锁和线程安全](https://swift.gg/2018/06/07/friday-qa-2015-02-06-locks-thread-safety-and-swift/)
10. [浅谈一种解决多线程野指针的新思路](http://satanwoo.github.io/2016/10/23/multithread-dangling-pointer/)
11. [深入理解 GCD](https://bestswifter.com/deep-gcd/)
12. [深入浅出GCD](https://xiaozhuanlan.com/Grand-Central-Dispatch)
13. [解密 Runloop](http://mrpeak.cn/blog/ios-runloop/)
14. [Matrix-iOS 卡顿监控](https://mp.weixin.qq.com/s/gPZnR7sF_22KSsqepohgNg)

## 通用知识

iOS 开发中还有很多和其他计算机领域相通的知识，比如渲染、数据库、网络等。

推荐扩展阅读文章有：

1. [深入理解 iOS Rendering Process](https://lision.me/ios_rendering_process/)
2. [绘制像素到屏幕上](https://objccn.io/issue-3-1/)
3. [手把手教你封装网络层](https://swift.gg/2017/04/25/how-do-I-build-a-network-layer/)
4. [A high performance JSON library in Swift](https://github.com/Ikiga/IkigaJSON)

网络相关文章推荐如下：

1. [百度App网络深度优化系列《三》弱网优化](https://mp.weixin.qq.com/s/BIfya6eVaWZW9ZEVz8RRcg)
2. [iOS 流量监控分析 | 周小鱼のCODE_HOME](http://zhoulingyu.com/2018/05/30/ios-network-traffic/)
3. [TCP/IP（一）：数据链路层](https://github.com/bestswifter/blog/blob/master/articles/tcp-ip-1.md)
4. [TCP/IP（二）：IP 协议](https://github.com/bestswifter/blog/blob/master/articles/tcp-ip-2.md)
5. [TCP/IP（三）：IP 协议相关技术](https://github.com/bestswifter/blog/blob/master/articles/tcp-ip-3.md)
6. [TCP/IP（四）：TCP 与 UDP 协议简介](https://github.com/bestswifter/blog/blob/master/articles/tcp-ip-4.md)
7. [TCP/IP（五）：TCP 协议详解](https://github.com/bestswifter/blog/blob/master/articles/tcp-ip-5.md)
8. [TCP/IP（六）：HTTP 与 HTTPS 简介](https://github.com/bestswifter/blog/blob/master/articles/tcp-ip-6.md)
9. [携程App的网络性能优化实践](http://chuansong.me/n/2577464)
10. [美团点评移动网络优化实践](http://tech.meituan.com/SharkSDK.html)
11. [万人低头时代，支付宝APP无线网络性能该如何保障](http://course.tuicool.com/course/details/58058f15a826b5f9e86678fb)
12. [QQ空间在生产环境使用QUIC协议的经验](https://mp.weixin.qq.com/s/qD9-Xj0CEil0Wtwq5eiPTg)
13. [美图HTTPS优化探索与实践](https://mp.weixin.qq.com/s/mRcz8o0usoqm_cEoGg9btg)
14. [九个问题从入门到熟悉 HTTPS](https://github.com/bestswifter/blog/blob/master/articles/https-9-questions.md)
15. [试图取代 TCP 的 QUIC 协议到底是什么](https://github.com/bestswifter/blog/blob/master/articles/quic.md)
16. [小谈 HTTP 中的编码](https://github.com/bestswifter/blog/blob/master/articles/http-encoding.md)
17. [利用 WireShark 深入调试网络请求](https://github.com/bestswifter/blog/blob/master/articles/wireshark.md)
18. [关于 iOS HTTP2.0 的一次学习实践 - 掘金](https://juejin.im/post/59caf86ef265da06484467e5)
19. [移动 APP 网络优化概述 « bang’s blog](http://blog.cnbang.net/tech/3531/)
20. [GYHttpMock：iOS HTTP请求模拟工具 | WeRead团队博客](http://wereadteam.github.io/2016/02/25/GYHttpMock/)
21. [YTKNetwork源码解析 | J*Knight*](https://knightsj.github.io/2017/07/18/YTKNetwork源码解析/)
22. [LefexWork/iOS 需要掌握的 HTTP 知识.md at master · lefex/LefexWork · GitHub](https://github.com/lefex/LefexWork/blob/master/blog/iOS/iOS 需要掌握的 HTTP 知识.md)
23. [LefexWork/以不一样的方式理解SDWebImage.md at master · lefex/LefexWork · GitHub](https://github.com/lefex/LefexWork/blob/master/blog/iOS/以不一样的方式理解SDWebImage.md)
24. [Alamofire的设计之道 - Leo的专栏 - CSDN博客](https://blog.csdn.net/Hello_Hwc/article/details/72853786)

## 专有知识

专有知识我就不展开说了，参考上面舆图中标注的知识点去检索你需要的就可以了。

## 视野

推荐手册：

1. [Apple Developer Documentation](https://developer.apple.com/documentation/)
2. [Swift 开发手册](https://swiftgg.gitbook.io/swift/huan-ying-shi-yong-swift)

开源控件：

1. [iOS Example](https://iosexample.com/)
2. [Cocoa Controls](https://www.cocoacontrols.com/)
3. [awesome-swift](https://github.com/matteocrippa/awesome-swift)
4. [Swift 开源项目精选 - 应用架构角度](https://xiaozhuanlan.com/topic/5796328014)
5. [Swift 开源项目精选导图](https://xiaozhuanlan.com/topic/5271086934)
6. [SwiftGuide](https://github.com/ipader/SwiftGuide)
7. [Swift 开源项目团队介绍](https://xiaozhuanlan.com/topic/7314260859)
8. [Swift 知名开发者介绍](https://xiaozhuanlan.com/topic/9687124530)

视频推荐 [Swift Talk - objc.io](https://talk.objc.io/) 。

这里的 iOS 博客都很值得订阅，[GitHub - awesome-tips/blogs: 行业优质博客汇总](https://github.com/awesome-tips/blogs)。

