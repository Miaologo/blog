---
title: iOS汇编基础篇-汇编LLDB调试
date: 2020-05-24 14:30:16
tags:
	- iOS
	- lldb
---



## LLDB 调试

### otool

要想查看静态库的代码段，就需要将静态库进行反汇编。

最简单的办法是使用 LLVM 工具链中的 otool，在 Shell 中输入 `otool -tv xxxx.a`，就可以得到反汇编的结果

### Xcode lldb 命令

通过Xcode lldb 读写寄存器

```
(lldb) register read x0

      x0 = 0x0000000000000001

(lldb) register read w0

       w0 = 0x00000001

(lldb) register write X0 0x1100000000000022



(lldb) register read x0

      x0 = 0x1100000000000022

(lldb) register read w0

      w0 = 0x00000022

(lldb) po $x0

1224979098644774946



(lldb) po/x $x0

0x1100000000000022  

(lldb) po/x $x0 - 1

0x1100000000000021    
```

最后一条命令是取出寄存器x0的值减去1后得到的值。

查看所有寄存器

```
(lldb) register read -f d
(lldb) register read -f x
General Purpose Registers:
        x0 = 0x1100000000000022
        x1 = 0x0000000000000002
        x2 = 0x0000000000000003
        x3 = 0x0000000000000004
        x4 = 0x0000000000000005
        x5 = 0x0000000000000006
        x6 = 0x0000000000000007
        x7 = 0x0000000000000008
        x8 = 0x000000000000000c
        x9 = 0x000000000000000b
       x10 = 0x000000000000000a
       x11 = 0x0000000000000009
       x12 = 0x000000000000000b
       x13 = 0x000000000000000c
       x14 = 0x0001800000018103
       x15 = 0x0000000000000000
       x16 = 0x000000010036031c  test`main at main.m:22
       x17 = 0x00000000ffffffff
       x18 = 0x0000000000000000
       x19 = 0x0000000000000000
       x20 = 0x0000000000000000
       x21 = 0x0000000000000000
       x22 = 0x0000000000000000
       x23 = 0x0000000000000000
       x24 = 0x0000000000000000
       x25 = 0x0000000000000000
       x26 = 0x0000000000000000
       x27 = 0x0000000000000000
       x28 = 0x000000016faa7a10
        fp = 0x000000016faa7980
        lr = 0x0000000100360380  test`main + 100 at main.m:24
        sp = 0x000000016faa78e0
        pc = 0x0000000100360264  test`executeLotsOfArguments + 76 at main.m:17

      cpsr = 0x60000000
```

sp (stack pointer)是栈顶指针
pc (program counter)存储着当前cpu正在执行的指令地址
cpsr 是程序状态寄存器，介绍 cmp 指令的时候有详细介绍。

这里解释一下寄存器pc，会在跳转指令 `bl` 的时候用到。
ARM处理器有3个阶段来处理指令，取指 -> 译码 -> 执行

- 取指从存储器装载一条指令
- 译码识别将要被执行的指令
- 执行处理指令并将结果写会寄存器

所以它处理时实际是这样的，ARM正在执行第1条指令的同时对第2条指令进行译码，并将第3条指令从存储器中取出。
所以，ARM7流水线只有在取第4条指令时，第1条指令才算完成执行。
无论处理器处于何种状态，程序计数器 pc 总是指向“正在取指”的指令，而不是指向“正在执行”的指令或者正在“译码”的指令。

简单了解通用寄存器后，再了解了解简单常用的汇编指令。
咱先在Xcode中 创建两个文件，arm.s 和 arm.h,声明一个 test() 函数，然后在main方法中调用，并打一短点
arm.h
```
void test();
```

arm.s
```
.text
.global _test

_test:
mov x0 ,#0x8
mov x1 ,x0
ret
```
main.m
```
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
#import "arm.h"

int main(int argc, char * argv[])
{
    test(); //打断点处
    @autoreleasepool {
    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
arm.s 就是写的汇编代码，里面有 test 函数的实现。
连接手机，运行程序到断点处，执行lldb命令 si 进入 test() 函数
```
(lldb) si
```
执行查看寄存器命令 register read
```
(lldb) register read
General Purpose Registers:
        x0 = 0x0000000000000001
        x1 = 0x000000016b927a20
        ...
```
可以看到通用寄存器 x0、x1 的值，
执行单步调试命令 ni （指令级），再查看一下寄存器的值，x0 就变成 0x08 了。
```
(lldb) ni
(lldb) register read
General Purpose Registers:
        x0 = 0x0000000000000008
        x1 = 0x000000016b927a20
```
再次执行 ni 命令下一步，然后查看寄存器的值
```
(lldb) ni
(lldb) register read
General Purpose Registers:
        x0 = 0x0000000000000008
        x1 = 0x0000000000000008
```
通过执行结果可以看出，mov x0 ,#0x8 指令是将立即数0x8存储到寄存器x0中，mov x1 ,x0 指令是将寄存器x0中的值传送到x1寄存器。
mov 指令可完成从另一个寄存器或将一根立即数加载到目的寄存器
mov x0 ,#0x8 也可以写成orr x0, xzr, #0x8 ，xzr 或 wzr 是零寄存器,里面存储的是0，xzr代表64位的零寄存器，wzr代表32位的零寄存器，orr 是逻辑或指令。
意思就是将0x8与 0 或运算后传递给 x0 寄存器。

ret 指令就相当于函数中的 return。