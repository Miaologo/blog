---
title: iOS汇编基础篇-汇编函数调用示例
date: 2020-05-24 14:07:23
tags:
	- iOS
	- 汇编

---



##汇编代码命令行

Objective C源文件(.m)的编译器是Clang + LLVM，Swift源文件的编译器是swift + LLVM。

所以借助clang命令，我们可以查看一个.c或者.m源文件的汇编结果

```
clang -S Demo.m
```

这是是x86架构的汇编，对于ARM64我们可以借助`xcrun`，

```
xcrun --sdk iphoneos clang -S -arch arm64 Demo.m
xcrun --sdk iphoneos clang -S -arch arm64 helloworld.c
```

**本文所有的汇编代码都基于ARM64架构CPU**

> **在Xcode中，Product -> Perform Action -> Assemble 来生成汇编文件。**
>
> **也可以在Xcode中，Debug -> Debug Workflow -> Always show Disassembly 来查看汇编运行代码。**











> 原文链接 [arm64 架构之入栈/出栈操作](http://blog.shenyuanluo.com/Arm64PushAndPopStack.html)



## 函数调用

> 每个函数调用，都会有 **入栈** 和 **出栈** 操作。

### 例子: PushAndPop.c

#### 源代码

[![img](http://chevereto.shenyuanluo.com/images/PushAndPopSource.png)](http://chevereto.shenyuanluo.com/images/PushAndPopSource.png)

```
#include <stdio.h>

void TestPushAndPop()
{
    printf("Push an Pop !");
}
```

#### 汇编代码

- 通过 Xcode “`Product——>Perform Action——>Assemble PushAndPop.c`“ 查看其对应的汇编代码：

  [![img](http://chevereto.shenyuanluo.com/images/PushAndPopAssembly1.png)](http://chevereto.shenyuanluo.com/images/PushAndPopAssembly1.png)

- 也可以通过 `clang` 编译成汇编代码：

  ```
  // 注意，以下代码将默认生成pc版的汇编指令
  clang -S PushAndPop.c
      
  // arm64汇编需要如下命令，指定架构和系统头文件所在的目录，请务必将isysroot的sdk版本修改为对应的 xcode 中存在的版本！
  clang -S -arch arm64 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS11.1.sdk PushAndPop.c
  ```

  [![img](http://chevereto.shenyuanluo.com/images/PushAndPopAssembly2.png)](http://chevereto.shenyuanluo.com/images/PushAndPopAssembly2.png)

> 去除一大堆 **不相干东西** 得到对应汇编代码，如下：

```
sub	sp, sp, #32             ; 更新栈顶寄存器的值，（可以看出：申请 32 字节占空间作为新用）
stp	x29, x30, [sp, #16]     ; 保存调用该函数前的栈顶寄存器的值和该函数结束返回后下一将执行指令地址值
add	x29, sp, #16            ; 更新栈底寄存器的值，(可以看出：还剩余 16 字节空间给该函数用)
adrp     x0, l_.str@PAGE		; 获取 ‘l_.str’ 标签所在的页的地址 
add x0, x0, l_.str@PAGEOFF	; 获取 ‘l_.str’ 标签对应页地址的偏移
bl	_printf									; 调用 ‘printf’ 函数进行打印
stur	w0, [x29, #-4]        ; 将 w0 寄存器的值('bl' 函数调用的返回值)保存到 [x29 - 4] 的内存地址中
ldp	x29, x30, [sp, #16]     ; 恢复调用该函数之前栈底寄存器的值
add	sp, sp, #32             ; 恢复调用该函数之前栈顶寄存器的值
ret													; 返回
```

> 对与上面的汇编代码，分配了 `32` 自己空间，其中 `16` 字节是用作 **入栈操作**，剩下的 `16` 字节是用于存储临时变量的。
>
> **疑问：**例子函数命名是没有临时变量，为什么还会需要申请占空间？
>
> **解释：**虽然该函数没有临时变量，但是调用 `printf` 函数后，编译器自动会加上 该函数*返回值* 的处理，由于 **arm64 规定了整数型返回值放在 `x0` 寄存器里**，因此会隐藏有一个局部变量 `int return_value;` 的声明在，该临时变量占用 `4`字节空间；又因为 **arm64 下对于使用 `sp` 作为地址基址寻址的时候，必须要 `16byte-alignment`（对齐）**，所以申请了 `16`字节空间作为临时变量使用。具体参见 [这里](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/using-the-stack-in-aarch64-implementing-push-and-pop)。

- 其 **入栈操作** 汇编代码流程解析如下：

  [![img](http://chevereto.shenyuanluo.com/images/PushStackGraph.png)](http://chevereto.shenyuanluo.com/images/PushStackGraph.png)

- 其 **出栈操作** 汇编代码流程解析如下：

  [![img](http://chevereto.shenyuanluo.com/images/PopStackGraph.png)](http://chevereto.shenyuanluo.com/images/PopStackGraph.png)

  > **注意：**对栈的 `分配/释放` 操作只会对栈指针做加减法, 而不会对栈内存中的内容做任何修改(也不会把释放的栈空间设置为 0)。
  >
  > 

## otool

```
otool -tv SimpleApp
```



```
otool -dv SimpleApp
```



动态符号表，***Dynamic Symbol Table*** ，其中 ***仅存储了符号位于Symbol Table中的下标*** ，而非符号数据结构，因为符号的结构仅存储在 ***Symbol Table*** 而已

使用 ***otool*** 命令可以查看动态符号表中的符号位于符号表中的下标。因此动态符号也叫做 ***Indirect symbols***

```
otool -I swift-hello.out
```



## 符号表

使用strings命令可以查看二进制中的可以打印出来的字符串，String Table里边的字符串当然也在其中了

```
strings - find the printable strings in a object, or other binary, file


strings - find libSimpleLib.a
```





## 参考文章

- [iOS开发同学的arm64汇编入门](https://blog.cnbluebox.com/blog/2017/07/24/arm64-start/)
- [iOS逆向之旅（基础篇） — 汇编（一）— 汇编基础](https://juejin.im/post/5bd197f5e51d457a2b7b248b)
- [[C in ASM(ARM64)\]第一章 一些实例](https://zhuanlan.zhihu.com/p/31168191)
- [Using the Stack in AArch64: Implementing Push and Pop](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/using-the-stack-in-aarch64-implementing-push-and-pop)
- [arm指令帮助文档](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0802a/BRK.html)