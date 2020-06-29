---
title: xcode调试技巧
date: 2020-05-22 17:07:49
tags:
	- iOS
	- Xcode
	- LLVM
---

日常开发调试技巧

## LLVM 常用命令

### expression

- expression 可以改变一个值，例如expression s
- expression可以使用e来代替
- e -p — dataArray 也可以打印对象的description方法的结果，等同于po

### call

```
call self.backgroundColor = [UIColor clear]
```

### print

在 LLDB 中有两个常见的打印指令 **p** 与 **po**。

- 1、**p** 通常用于打印基本数据类型的值。这个指令会默认生出一个临时变量，如**$1**，学习过 **Shell** 的小伙伴看到这个应该很激动。
- 2、**po** 打印变量的内容，如果是对象，其打印的内容由 **-debugDescription**  决定。

```
po $arg1
po (SEL)$arg2

```

- print 打印需要查看的变量，例如print totalCount
- print 还能使用简写prin, pri, p
- po(print object)可以打印对象的description方法的结果
- 打印不同格式可以用p/x number打印十六进制，p/t number打印二进制，p/c char打印字符。这里是完整清单https://sourceware.org/gdb/onlinedocs/gdb/Output-Formats.html

### memory 操作

对内存的操作，无非就是读写操作。 修改内存中的值：

> memory  write  内存地址  数值

```
memory write 0x7ffee685dba8 25
```

读取内存操作：

> **memory read/数量 _ 格式 _ 字节数  内存地址**

或者

> **x/数量 _ 格式 _ 字节数  内存地址**

#### 格式

- **x** ：代表16进制
- **f** ：代表浮点数
- **d** ：代表10进制

#### 字节大小

- **b** ：byte            代表1个字节
- **h** ：half word     代表2个字节
- **w** ：word            代表4个字节
- **g** ：giant word    代表8个字节

如：

> **memory read/1wx 0x7ffee14a5ba8** 
>
> **memory read/1wd 0x7ffee14a5ba8**

寓意是：**读取 0x7ffee14a5ba8 中 4  个字节的内容。**


### bt

bt 返回所有的调用栈， 形如：

```
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = step over
  * frame #0: 0x000000010758b6fd LLDBDev`-[ViewController viewDidLoad](self=0x00007fedad7057e0, _cmd="viewDidLoad") at ViewController.m:27
  **中间省略较多**
    frame #34: 0x000000010758b79f LLDBDev`main(argc=1, argv=0x00007ffee8674108) at main.m:14
    frame #35: 0x000000010c2d1d81 libdyld.dylib`start + 1
    frame #36: 0x000000010c2d1d81 libdyld.dylib`start + 1
```

这个指令很强大

###watch

断点进入 lldb 环境之后，可以执行如下指令：

> watchpoint set variable **self->_name**

log 为：

```
Watchpoint created: Watchpoint 1: addr = 0x7fcfaf9061d0 size = 8 state = enabled type = w
    watchpoint spec = 'self->_name'
    new value: 0x0000000000000000
```

即为断点成功。

log 能将旧值、新值一起打印出来。

当然，设置内存断点的指令，还可以这样：

> watchpoint set expression **&_name**

**内存断点**， 在分析数据流转的特别有用，比如就想知道某个变量在什么情况下为 *nil* 了。

想监视vMain变量什么时候被重写了，监视这个地址什么时候被写入

```
(lldb) p (ptrdiff_t)ivar_getOffset((struct Ivar *)class_getInstanceVariable([MyView class], "vMain"))
(ptrdiff_t) $0 = 8
(lldb) watchpoint set expression -- (int *)$myView + 8
Watchpoint created: Watchpoint 3: addr = 0x7fa554231340 size = 8 state = enabled type = w
new value: 0x0000000000000000
```



## 流程控制

- continue会取消暂停，继续执行下去到达下一个断电，LLDB中使用process continue，别名continue，或者使用缩写c
- step over会执行当前这个函数，然后继续。LLDB中使用thread step-over，next或者缩写n
- step into指跳进一个函数调试。LLDB中使用thread step in，step或者s
- step out会继续执行到下一个返回语句，然后再次停止
- thread return会在当前断点处直接返回出函数，函数剩余部分不会被执行。LLDB中使用thread return NO

## 断点调试

文章 Xcode llvm断点

##调试用例

### UI 控件查看

自动布局约束问题，在控制台会给出这样的提示。如果界面简单，那么很好排查，如果界面复杂，那么就很难定位问题所在。那么如何找到具体的视图呢？可以这样来：

通过命令行实时的定位到是界面上的哪个UI 了，具体的命令如下：

```
(lldb) e id $hgView = (id)0x7fdfc66127f0
(lldb) e (void)[$hgView setBackgroundColor:[UIColor redColor]]
(lldb) e (void)[CATransaction flush]
```

**注意**：后面的那个命令一定要执行，否则在 lldb 的状态下是看不到效果的。

###符号断点

```
breakpoint set -one-shot true --name "-[UILabel setText:]"
```

其中关键的命令是这样的 `breakpoint set -one-shot true --name "-[UILabel setText:]"` 这句命令的大意是如果在 `btn1` 方法中有 `-[UILabel setText:]` 操作的话，会被自动触发跟踪。

### 查找按钮的 target

想象你在调试器中有一个 `$myButton` 的变量，可以是创建出来的，也可以是从 UI 上抓取出来的，或者是你停止在断点时的一个局部变量。你想知道，按钮按下的时候谁会接收到按钮发出的 action。非常简单：

```
(lldb) po [$myButton allTargets]
{(
    <MagicEventListener: 0x7fb58bd2e240>
)}
(lldb) po [$myButton actionsForTarget:(id)0x7fb58bd2e240 forControlEvent:0]
<__NSArrayM 0x7fb58bd2aa40>(
_handleTap:
)
```

现在你或许想在它发生的时候加一个断点。在 `-[MagicEventListener _handleTap:]` 设置一个符号断点就可以了，在 Xcode 和 LLDB 中都可以，然后你就可以点击按钮并停在你所希望的地方了。

### 观察实例变量的变化

假设你有一个 `UIView`，不知道为什么它的 `_layer` 实例变量被重写了 (糟糕)。因为有可能并不涉及到方法，我们不能使用符号断点。相反的，我们想**监视**什么时候这个地址被写入。

首先，我们需要找到 `_layer` 这个变量在对象上的相对位置：

```
(lldb) p (ptrdiff_t)ivar_getOffset((struct Ivar *)class_getInstanceVariable([MyView class], "_layer"))
(ptrdiff_t) $0 = 8
```

现在我们知道 `($myView + 8)` 是被写入的内存地址：

```
(lldb) watchpoint set expression -- (int *)$myView + 8
Watchpoint created: Watchpoint 3: addr = 0x7fa554231340 size = 8 state = enabled type = w
    new value: 0x0000000000000000
```

这被以 `wivar $myView _layer` 加入到 [Chisel](https://github.com/facebook/chisel) 中。

### 非重写方法的符号断点

假设你想知道 `-[MyViewController viewDidAppear:]` 什么时候被调用。如果这个方法并没有在`MyViewController` 中实现，而是在其父类中实现的，该怎么办呢？试着设置一个断点，会出现以下结果：

```
(lldb) b -[MyViewController viewDidAppear:]
Breakpoint 1: no locations (pending).
WARNING:  Unable to resolve breakpoint to any actual locations.
```

因为 LLDB 会查找一个**符号**，但是实际在这个类上却找不到，所以断点也永远不会触发。你需要做的是为断点设置一个条件 `[self isKindOfClass:[MyViewController class]]`，然后把断点放在 `UIViewController` 上。正常情况下这样设置一个条件可以正常工作。但是这里不会，因为我们没有父类的实现。

`viewDidAppear:` 是苹果实现的方法，因此没有它的符号；在方法内没有 `self` 。如果想在符号断点上使用 `self`，你必须知道它在哪里 (它可能在寄存器上，也可能在栈上；在 x86 上，你可以在 `$esp+4` 找到它)。但是这是很痛苦的，因为现在你必须至少知道四种体系结构 (x86，x86-64，armv7，armv64)。想象你需要花多少时间去学习命令集以及它们每一个的[调用约定](http://en.m.wikipedia.org/wiki/Calling_convention)，然后正确的写一个在你的超类上设置断点并且条件正确的命令。幸运的是，这个在 [Chisel](https://github.com/facebook/chisel) 被解决了。这被成为 `bmessage`：

```
(lldb) bmessage -[MyViewController viewDidAppear:]
Setting a breakpoint at -[UIViewController viewDidAppear:] with condition (void*)object_getClass((id)$rdi) == 0x000000010e2f4d28
Breakpoint 1: where = UIKit`-[UIViewController viewDidAppear:], address = 0x000000010e11533c
```

### LLDB 和 Python

LLDB 有内建的，完整的 [Python](http://lldb.llvm.org/python-reference.html) 支持。在LLDB中输入 `script`，会打开一个 Python REPL。你也可以输入一行 python 语句作为 `script 命令` 的参数，这可以运行 python 语句而不进入REPL：

```
(lldb) script import os
(lldb) script os.system("open http://www.objc.io/")
```

这样就允许你创造各种酷的命令。把下面的语句放到文件 `~/myCommands.py` 中：

```
def caflushCommand(debugger, command, result, internal_dict):
  debugger.HandleCommand("e (void)[CATransaction flush]")
```

然后再 LLDB 中运行：

```
command script import ~/myCommands.py
```

或者把这行命令放在 `/.lldbinit` 里，这样每次进入 LLDB 时都会自动运行。[Chisel](https://github.com/facebook/chisel) 其实就是一个 Python 脚本的集合，这些脚本拼接 (命令) 字符串 ，然后让 LLDB 执行。很简单，不是吗？

## 参考文章

[Xcode-LLVM-调试技巧](https://zhuanlan.zhihu.com/p/63629659)

[**与调试器共舞 - LLDB 的华尔兹**](https://objccn.io/issue-19-2/)

https://github.com/facebook/chisel