---
title: iOS基础篇-静态链接和动态链接
date: 2020-05-26 10:37:59
tags:
	- iOS
	- DYLD
---

##静态链接





##动态链接

> 名称解析：
>
> 1. dyld：the dynamic link editor 。后面dyld表示动态链接器
> 2. dylib：动态链接库或者称共享对象

Mach-O文件的通过dyld加载的时候并没有确定每一个函数的具体地址在哪里，而是在真正调用该函数的时候通过**过程连接表**(procedure linkage table)，后面简称PLT，来进行一次**lazybind**。



具体到macho文件，因为模块间的数据访问很少（模块间还提供很多全局变量给其它模块用，那耦合度太大了，所以这样的情况很少见），所以外部数据地址，都是放到got（也称Non-Lazy Symbol Pointers）数据段，非惰性的，动态链接阶段，就寻找好所有数据符号的地址；而模块间函数调用就太频繁了，就用了延迟绑定技术，将外部函数地址都放在la_symbol_ptr(Lasy Symbol Pointers)数据段，惰性的，程序第一次调用到这个函数，才寻址函数地址，然后将地址写入到这个数据段。



### stub桩机制总结

所有的外部函数引用都会在`DATA`段`la_symbol_ptr`区中产生一个占位符，其初始值为`dyld_stub_binder`区中对应的编号地址。当第一个调用时，就会进入符号的动态链接过程，一旦找到其地址后，就会将`DATA`段`la_symbol_ptr`区中的占位符改为找到后的地址。这样就完成了只需要一个符号绑定。

stub桩机制的巧妙之处也在此，首先当产生一个外部符号调用时，直接跳到对应的stub桩位置，然后由里面保存的地址来判断是第一次调用还是已经找到符号的地址。就像桩这个名字含义一样，一个占位符的思想

### 参考文档

[深入理解iOS动态链接](http://4ch12dy.site/2019/09/14/iOS-dyld-dynamic-link/iOS-dyld-dynamic-link/)
[Mach-o动态链接](http://4ch12dy.site/2017/04/10/macho-dyld-link/macho-dyld-link/)
[Mach-O的动态链接](http://turingh.github.io/2016/03/10/Mach-O的动态链接/)
http://www.newosxbook.com/articles/DYLD.html#footnote
https://github.com/opensource-apple/dyld
https://github.com/gdbinit/MachOView
https://blog.gocy.tech/2018/08/01/behindthescenes-symbol-resolve/
https://blog.gocy.tech/2019/07/08/hook-msgSend-advance/

