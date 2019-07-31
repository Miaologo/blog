---
title: iOS 一些汇编知识点记录
date: 2019-07-24 10:59:16
tags: iOS，汇编
---

iOS 设备使用基于 ARM 结构的 CPU

ARM处理器有16个寄存器，从r0到r15，每一个都是32位比特。调用约定指定他们其中的一些寄存器有特殊的用途，例如：

- r0-r3：用于存放传递给函数的参数；
- r4-r11：用于存放函数的本地参数；
- r12：是内部程序调用暂时寄存器。这个寄存器很特别是因为可以通过函数调用来改变它；
- r13：栈指针sp(stack pointer)。在计算机科学内栈是非常重要的术语。寄存器存放了一个指向栈顶的指针。看[这里](https://link.jianshu.com?t=http://en.wikipedia.org/wiki/Call_stack)了解更多关于栈的信息；
- r14：是链接寄存器lr(link register)。它保存了当目前函数返回时下一个函数的地址；
- r15：是程序计数器pc(program counter)。它存放了当前执行指令的地址。在每个指令执行完成后会自动增加

你可以在[ARM文档](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0042f/IHI0042F_aapcs.pdf)里了解更多关于ARM调用约定的信息

想知道更多关于指令的信息，可以看看[这个文档](http://infocenter.arm.com/help/topic/com.arm.doc.qrc0001l/QRC0001_UAL.pdf)，或者[看其他的中文](http://read.pudn.com/downloads151/sourcecode/embed/654540/Cortex_M3_Guide/chpt04-05.pdf)

