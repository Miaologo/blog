---
title: iOS二进制重排
date: 2020-05-11 23:38:10
tags:
	- iOS
	- 二进制重排
categories:
    - Jason
---

## 二进制重排

根据 [Apple 官方文档](https://pewpewthespells.com/blog/buildsettings.html)，其中提到了对重排符号的设置方法：

> The path to a file which alters the order in which functions and data are laid out. For each section in the output file, any symbol in that section that are specified in the order file is moved to the start of its section and laid out in the same order as in the order file. Order files are text files with one symbol name per line. Lines starting with a # are comments. A symbol name may be optionally preceded with its object file leafname and a colon (e.g. foo.o:_foo). This is useful for static functions/data that occur in multiple files. A symbol name may also be optionally preceded with the architecture (e.g. ppc:_foo or ppc:foo.o:_foo). This enables you to have one order file that works for multiple architectures. Literal c-strings may be ordered by quoting the string in the order file (e.g. “Hello, world”). Generally you should not specify an order file in Debug or Development configurations, as this will make the linked binary less readable to the debugger. Use them only in Release or Deployment configurations.

简言之，即创建一个文本文件，每行一个符号，将 order_file 的路径配置到 Xcode 的 Build Settings 中的 Order File 配置项，随后链接器就会按照 order_file 中的顺序来排列符号了。**注意这个地方有个坑，如果符号都是用汇编写的，order_file 是不会生效的**


## __attribute__ 使用


## 参考文档

[抖音研发实践：基于二进制文件重排的解决方案 APP启动速度提升超15%](https://mp.weixin.qq.com/s/Drmmx5JtjG3UtTFksL6Q8Q)

[iOS基于二进制重排的启动优化](https://juejin.im/post/5e701ed5e51d4526e91f6916)

[App 二进制文件重排已经被玩坏了](http://yulingtianxia.com/blog/2019/09/01/App-Order-Files/)

[我是如何让微博绿洲的启动速度提升30%的](https://mp.weixin.qq.com/s?__biz=MzA5NzMwODI0MA==&mid=2647766465&idx=1&sn=e9598765d3314a0d17dc98069344b6a2&scene=21#wechat_redirect)

[基于 Mach-O 符号重排减少缺页中断次数来提升 iOS App 启动速度的可行性分析](https://juejin.im/post/5d5a05255188251f4705fb8b)

[一行代码解决！iOS 二进制重排启动优化](https://www.infoq.cn/article/P9gE3zuDHLRrRfW4hvRP)

[手淘架构组最新实践 | iOS基于静态库插桩的⼆进制重排启动优化](https://mp.weixin.qq.com/s/YDO0ALPQWujuLvuRWdX7dQ)

[iOS基于二进制重排启动优化](https://juejin.im/post/5eeeccd6f265da02e47d900b#heading-11)