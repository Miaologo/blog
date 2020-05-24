---
title: iOS汇编基础
date: 2020-05-18 14:05:47
tags:
	- iOS
	- 汇编
---

> **ARM64汇编为了提高访问效率要求按照16字节进行对齐**

## iOS 计算机基本概念

### 寄存器

如果你还不知道什么是寄存器，建议先Google一下。 这里不再详细说明，寄存器是CPU中的高速存储单元，要比内存中存取要快的多。

这里说明一下arm64有哪些寄存器：

#### **R0 – R30**

`r0 - r30` 是31个通用整形寄存器。每个寄存器可以存取一个64位大小的数。 当使用 `x0 - x30`访问时，它就是一个64位的数。当使用 `w0 - w30`访问时，访问的是这些寄存器的低32位，如图：

![1.png](https://blog.cnbluebox.com/images/arm64-start/1.png)

其实通用寄存器有32个，第32个寄存器x31，在指令编码中，使用来做 `zero register`, 即`ZR`, `XZR/WZR`分别代表64/32位，`zero register`的作用就是0，写进去代表丢弃结果，拿出来是0.

其中 `r29` 又被叫做 `fp` (frame pointer). `r30` 又被叫做 `lr` (link register)。其用途会在下一节《栈》中讲到。

#### **SP**

SP寄存器其实就是 x31，在指令编码中，使用 `SP/WSP`来进行对SP寄存器的访问。

#### **PC**

PC寄存器中存的是当前执行的指令的地址。在arm64中，软件是不能改写PC寄存器的。

#### **V0 – V31**

`V0 - V31` 是向量寄存器，也可以说是浮点型寄存器。它的特点是每个寄存器的大小是 128 位的。 分别可以用`Bn Hn Sn Dn Qn`的方式来访问不同的位数。如图：

![2.png](https://blog.cnbluebox.com/images/arm64-start/2.png)

`Bn Hn Sn Dn Qn`可以这样理解记忆, 基于一个word是32位，也就是4Byte大小：

> Bn: 一个Byte的大小
> Hn: half word. 就是16位
> Sn: single word. 32位
> Dn: double word. 64位
> Qn: quad word. 128位

#### **SPRs**

SPRs是状态寄存器，用于存放程序运行中一些状态标识。不同于编程语言里面的if else.在汇编中就需要根据状态寄存器中的一些状态来控制分支的执行。状态寄存器又分为 `The Current Program Status Register (CPSR)` 和 `The Saved Program Status Registers (SPSRs)`。 一般都是使用`CPSR`， 当发生异常时， `CPSR`会存入`SPSR`。当异常恢复，再拷贝回`CPSR`。

还有一些系统寄存器，还有 `FPSR` `FPCR`是浮点型运算时的状态寄存器等。基本了解上面这些寄存器就可以了。

CPSR (Current Program Status Register)是程序状态寄存器，cpsr 是一个32bit 的寄存器
![img](https://images.xiaozhuanlan.com/photo/2018/54be72f36471426505b9e412c31fd0eb.jpeg)

##### N

- Negative 标志位,当用两个补码表示的带符号数进行运算时，N=1 表示运算的结果为负数；N=0 表示运算的结果为正数或零.

##### Z

- Zero 标志位,Z=1 表示运算的结果为零；Z=0表示运算的结果为非零.如果结果为零，通常表示比较的结果相等。

##### C

- Carry 标志位，有以下3种情况
  1、无符号加法运算和cmn指令，如果产生进位，则C=1，否则C=0；
  2、无符号减法运算和cmp指令，如果产生借位，则C=0，否则C=1；
  3、进行移位操作的时候，C中保存最后一位移出的值。
  说明：当一条指令中同时含有算术运算指令和移位指令时，影响C的值是算术运算而不是移位操

##### V

- 溢出标志位。进行有符号运算时如果发生错误，则V=1，否则V=0。

一些指令如cmn、cmp等会无条件的刷新cpsr中的条件标志位，其他指令必须要在指令后面加上S后缀才会改变CPSR中的条件标志位。比如 `add x0, x0, #0x1` 写成 `adds x0, x0, #0x1`

你可以在[ARM文档](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0042f/IHI0042F_aapcs.pdf)里了解更多关于ARM调用约定的信息

想知道更多关于指令的信息，可以看看[这个文档](http://infocenter.arm.com/help/topic/com.arm.doc.qrc0001l/QRC0001_UAL.pdf)，或者[看其他的中文](http://read.pudn.com/downloads151/sourcecode/embed/654540/Cortex_M3_Guide/chpt04-05.pdf)

## 反调试

> [反调试及绕过](http://jmpews.github.io/2017/08/09/darwin/反调试及绕过/)

### 反调试

以反调试为例，我们知道，通过调用ptrace函数可以阻止调试器依附。

```objective-c
ptrace(31, 0, 0, 0)
```

这种方式能够被函数hook轻易破解，例如使用facebook的[fishhook](https://github.com/facebook/fishhook)。

###asm

// volatile修饰符能够防止汇编指令被编译器忽略

### volatile

// volatile修饰符能够防止汇编指令被编译器忽略

```asm
// 使用inline方式将函数在调用处强制展开，防止被hook和追踪符号
static __attribute__((always_inline)) void anti_debug() {
// 判断是否是ARM64处理器指令集
#ifdef __arm64__
    // volatile修饰符能够防止汇编指令被编译器忽略
    __asm__ __volatile__(
                         "mov x0, #31\n"
                         "mov x1, #0\n"
                         "mov x2, #0\n"
                         "mov x3, #0\n"
                         "mov x16, #26\n"
                         "svc #0x80\n"
                         );
#endif
}
```

其中x0-x3存储的为函数入参，x16存储的为函数编号，通过Apple提供的[System Call Table](https://www.theiphonewiki.com/wiki/Kernel_Syscalls) 可以查出ptrace的编号为26，最后一句指令发起了系统调用。 通过使用__asm__指令能够将汇编代码嵌入函数中，构成反调试方法。



# iOS汇编

## GNU平台无关

### 符号定义伪指令

```
.global`,`.local`,`.set`,`.equ
```

#### .global

使得符号对连接器可见，变为对整个工程可用的全局变量

```
.global symbol
```

`.globl` 可以让一个标签对链接器可见，可以供其他链接对象模块使用。 换句话说 `.global _my_func` 可以在代码中调用到 `my_func`。

#### .local

表示符号对外部不可见，只对本文件可见

```
.local symbol
```

#### .set

给一个全局变量或局部变量赋值，和`.equ`的功能一样

```
.set symbol expr
.set start, 0x40
.set start, 0x50
mov r1, #start      ;r1里面是0x50
```

#### .equ

和`.set`一样，只是格式不同

```
symbol .equ  expr
start  .equ, 0x40
start  .equ, 0x50
mov r1, #start      ;r1里面是0x50
```

### 数据定义伪指令

```
.byte`,`.short`,`.long`,`.quad`,`.float`,`.string`,`.asciz`,`.ascii`,`.rept
```

#### .byte

在存储器中分配**1个字节**，用指定的数据对存储单元进行初始化

```
label:  .byte   expr    ;label是程序标号，expr可以是-128~255的数字，也可是字符
a:  .byte   #1  ;等价于C中的char a=1;
```

#### .short

在存储器中分配**2个字节**，用指定的数据对存储单元进行初始化

```
a: .short 0x1234
```

#### .word / .long

在存储器中分配**4个字节**，用指定的数据对存储单元进行初始化

```
a: .word 0x12345678
```

#### .long

在存储器中分配**个字节**，用指定的数据对存储单元进行初始化

#### .quad

在存储器中分配**8个字节**，用指定的数据对存储单元进行初始化

```
a: .quad 0x12345678 ;等价于C中的long a=0x1234567812345678
```

#### .float

在存储器中分配**4个字节**，用指定的**浮点**数据对存储单元进行初始化

```
a: .float 1.11
```

#### .space/.skip

用于分配一块连续的存储区域并初始化为指定的值，如果后面的填充值省略不写则在后面填充为0;

```
label: .space size,expr     ;expr可以是4字节以内的浮点数 
a:  space 8, 0x1
```

#### .string

定义一个字符串，默认是string8，还有string16，string32，string64

```
a: .space "hello world!"
```

#### .rept

重复执行接下来的指令，以.rept开始，以.endr结束

```
.rept cnt   ;cnt是重复次数
...
.endr
```

`.rept` 和 `.endr` 是一组循环伪指令。

```text
func_1:
.rept 5
add x0, x0, x1
.endr
ret
```

上述指令得到内容如下： ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-4.8d465430.png) 生成了 5 个连续的 `add` 指令。

### 汇编控制伪操作

流程控制伪指令主要yy`.if .else .endif` `.macro .endm .exitm`

#### .if .else .endif

```
.if logical-expression
...
.elseif logical-expression2
...
.else
...
.endif
```

#### .macro .endm .exitm

该伪指令可以将一段代码定义为一个整体，称为宏指令，然后就可以在程序中通过宏指令多次调用该段代码，而`.exitm`指令用来退出当前的宏指令，宏指令可以使用一个或多个参数，当宏操作被展开时，这些参数被相应的值替换。
包含在`.macro`和`。endm`之间的指令序列称为宏定义体。在宏定义体的第一行应声明宏的原型，包含宏名所需的参数，然后就可以在汇编程序中通过宏名来调用该指令序列，在源程序被编译时，汇编器将宏调用展开，用宏定义中的指令序列代替程序中的宏调用，并将实际参数的值传递给宏定义中的形式参数

```
.macro macroname macargs ...
;code
.endm
```

### 杂项

```
.align      用于使程序当前位置满足一定的对齐方式
.section    用来定义一个段的伪指令
.data       用来定义一个数据段
.text       用来定义一个代码段
.include    用来包含一个头文件   
.arm        定义以下代码使用arm指令集编译
.code 32    同.arm
.code 16    同.thumb
.thumb      定义以下代码使用thumb指令集编译
.extern     用于声明一个外部符号，用于兼容性其他汇编
.weak       用于声明一个弱符号，如果这个符号没有定义，编译就忽略，而不会报错
.end        表示汇编结束
```

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

### 标签

如下代码：

```text
_my_label:
add x0, x0, x1
```

`_my_label` 会指向 `add x0, x0, x1` 的地址，用于辅助标记命令地址。

## GNU平台相关

#### adr

把标签所在的地址加载到寄存器中，这个指令将基于PC相对偏移的地址值或基于寄存器相对偏移的地址值读取到寄存器中。当地址值是字节对齐的时候，取值范围是-255~255B;当地址值是字对齐的时候，取值范围为-1020~1020B。当地址值是16字节对齐时，取值范围更大。 该指令等价于`add , pc , offset`

```
adr <reg> <label>
```

#### adrl

用于将中等范围地址读取到寄存器中

```
ADRL <reg> <label>
```

#### adrp



#### stur





#### ldr

装载一个32位的常数和一个地址寄存器

```
LDR reg, =expr
```

reg:目标寄存器
expr：32位常量表达式。汇编器根据expr的取值情况，对LDR伪指令做如下处理:

1. 当expr表示的指令地址值没有超过MOV指令或MVN指令的地址取值范围时，汇编器用一对MOV和MVN代替LDR指令
2. 当超过了的时候，汇编器将常数放入缓存吃，同时用一条基于PC的LDR读取该常数

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

str 全称是store register，即将寄存器的值存储到内存中， 因此第一个参数都是寄存器，第二个参数都是内存地址，注意这里的数字都是以字节为单位的偏移量

两个入栈指令，stp 可以同时操作两个寄存器。

这里涉会及到一些寻址的格式，有 3 种方式：

```asm
[x10, #0x10]      // signed offset。 意思是从 x10 + 0x10的地址取值
[sp, #-16]!       // pre-index。  意思是从 sp-16地址取值，取值完后在把 sp-16  writeback 回 sp，即开辟一段新栈空间
[sp], #16         // post-index。 意思是从 sp 地址取值，取值完后在把 sp+16 writeback 回 sp，即释放一些栈空间
```

结合上面几种寻址方式，搭配入栈指令，用法如下：

```asm
str x8, [sp, #-16]! ;将 x8 存到 sp - 16 的位置，并将 sp -= 16，
stp x4, x5, [sp, #-16]! ;将 x4 x5 的值存到 sp - 16 的位置，并将 sp -= 16
```

#### ldr 和 ldp

ldr的全称是load register，即将内存中的值读到寄存器，因此他们的第一个参数都是寄存器，第二个参数都是内存地址，注意这里的数字都是以字节为单位的偏移量

两个出栈指令，ldp 可以同时操作两个寄存器，用法如下：

```asm
ldr x8, [sp], #16 ;将 sp 位置的值取出来，存入 x8 中，并将 sp += 16
ldp x4, x5, [sp], #16 ;将 sp 位置的值取出来，存入 x4 x5 中，并将 sp += 16
```

#### LDXR & STXR 

----

LDXR 即 LDR 的 Exclusive 版本，它的用法与 LDR 完全一致，区别在于它含有 Load-Exclusive 语义，即将读取的内存单元状态置为 Exclusive。

STXR 即 STR 的 Exclusive 版本，由于需要是否 Store 成功，他相比于 STR 多了一个 32 位寄存器的参数用于接收执行结果，用法为：

```
STXR  Ws, Xt, [Xn|SP{,#0}]
复制代码
```

即尝试将 `Xt` 写入 `[Xn|SP{,#0}]`，如果写入成功则将 0 写入 Ws，否则将非 0 写入，它常常和 CBZ 指令搭配，如果写入失败则跳回到 LDXR，重新执行一遍 LDXR & STXR 操作，直至成功

#### LDAXR & STLXR

----

除了 Exclusive 语义外，LDXR & STXR 还有其 Acquire-Release 语义的 LDAXR & STLXR 版本，用于保证执行顺序。

对于单纯的 Atomic Add 操作，前者已经足够；如果涉及到类似于 [上一篇文章](https://juejin.im/post/5d9891abf265da5b926bc2b7) 提到的读写等待操作，则需要通过后者强保证不被乱序执行干扰。

#### CAS & CASA & CASAL

---

ARM 提供了多条指令直接完成 Compare and Swap 操作，其中 CAS 是最基础的版本，它的使用方法如下[4]：

```
CAS Xs, Xt, [Xn|SP{,#0}] ; 64-bit, no memory ordering
复制代码
```

尝试将 `Xt` 与内存中的值进行交换，首先比较 `Xs` 是否等于内存中的 `[Xn|SP{,#0}]`，如果相等则将 `Xt` 写入内存，同时将内存中的值写回到 `Xs`，因此只要在 CAS 之后判断 `Xs` 是否等于 `Xt` 即可知道是否写入成功，如果写入失败则 `Xs` 的值应为原始值，即 `Xs` ≠ `Xt`，如果写入成功则内存中的值已被更新，即 `Xs` = `Xt`。

下面的例子采用 CAS 方式同样实现了原子加一操作：

```
; extern int cas_add(int *val);
_cas_add:
mov x9, x0
ldr w10, [x9]
mov w11, w10 ; w11 is used to check cas status
add w10, w10, #1
cas w11, w10, [x9]
cmp w10, w11 ; if cas succeed, w11 = <new value in memory> = w10
b.ne _cas_add
mov w0, w10
ret
复制代码
```

**注意：为了在 iOS 系统上编译包含 CAS 指令的内容，需要给 .s 文件添加一个 Compile Flag: `-march=armv8.1-a`**[5]。

> 同样的，CAS 也有其含有 Acquire-Release 语义的版本，分别是含有 Acquire 语义的 CASA, 含有 Release 语义的 CASL，和同时包含 Acquire-Release 两种语义的 CASAL。



#### br

----

跳转指令，直接跳转到指定地址，跳转完不返回。有些类似在一个函数末尾调用了另外的函数。用法如下：

```asm
br x10 ;跳转到 x10 中保存的地址
```

#### bl

跳转指令，将 bl 的下一个指令地址保存到 lr 寄存器，然后跳转到指定地址，因为将下一个指令保存到 lr 了，所以跳转完会回来执行下一个指令。用法如下：

```asm
mov x0, x1
bl 0x100221232 ;跳转到 0x100221232，执行完毕回来执行下一条指令
sub x0, x0, 0x10
```

#### blr

跳转指令，和 bl 类似，但可以使用动态地址，可以跳转到寄存器的值保存的地址，用法如下：

```asm
mov x8, 0x100221232
blr x8
```



#### b.ne



#### lr



#### csel

 `csel` 指令，该指令是 ARM 中的三目运算：

```asm
csel   x21, x21, xzr, ne

csel  Wd, Wn, Wm, cond 
# 等价于
Wd = cond ? Wn : Wm
```



#### cbz



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

## 内存屏障 volatile

-----

内存屏障是一条指令，它能够明确地保证屏障之前的所有内存操作均已完成（可见）后，才执行屏障后的操作，但是它不会影响其他指令（非内存操作指令）的执行顺序。

因此我们只要在 flag 置位前放置内存屏障，即可保证运算结果全部写入内存后才置位 flag，进而也就保证了逻辑的正确性。

###放置内存屏障

-----

可以通过内联汇编的形式插入一个内存屏障：''

```c
void calculate(FlagsCalculate *ctx) {
    ctx->a = (20 * mulA - mulB) / divC;
    ctx->b = 30 + addD;
    for (NSInteger i = 0; i < 10000; i++) {
        ctx->a += i * mulA - mulB;
        ctx->a *= divC;
        ctx->b += i * mulB / mulA - mulB;
        ctx->b /= divC;
    }
    ctx->c = mulA + mulB * divC + 120;
    ctx->d = addD + mulA + mulB + 5;
    __asm__ __volatile__("dmb sy");
    ctx->e = 1;
    ctx->f = 1;
    ctx->g = 1;
}
复制代码
```

`__asm__ __volatile__("dmb sy")`，内存屏障限制了 CPU 乱序执行对正常逻辑的影响。

### volatile 与内存屏障

-----

常常听说 volatile 是一个内存屏障，那么它的屏障作用是否与上述 DMB 指令一致呢，我们可以试着用 volatile 修饰 3 个 flag，再做一次实验：

```c
typedef struct FlagsCalculate {
    int a;
    int b;
    int c;
    int d;
    volatile int e;
    volatile int f;
    volatile int g;
} FlagsCalculate;
复制代码
```

结果最后触发了断言异常，这是为何呢？**因为 volatile 在 C 环境下仅仅是编译层面的内存屏障，仅能保证编译器不优化和重排被 volatile 修饰的内容**，但是在 Java 环境下 volatile 具有 CPU 层面的内存屏障作用[4]。不同环境表现不同，这也是 volatile 让我们如此费解的原因。

在 C 环境下，volatile 常常用来保证内联汇编不被编译优化和改变位置，例如我们通过内联汇编放置一个编译层面的内存屏障时，通过 `__volatile__` 修饰汇编代码块来保证内存屏障的位置不被编译器改变：

```asm
__asm__ __volatile__("" ::: "memory");
```

###  volatile 相关 汇编命令




## 反汇编命令

反汇编 `dis -a xxxx`

```asm
(lldb) dis -a 0x18c1391a8
Foundation`+[NSUnitElectricPotentialDifference megavolts]:
0x18c1391a8 <+0>: adrp   x16, 195909
0x18c1391ac <+4>: ldr    x16, [x16, #0x8b8]
0x18c1391b0 <+8>: br     x16

```

这里取出了一个符号地址存入 x16 来执行，我们来看看 x16 里到底是什么：

```asm
(lldb) p/x 0x18c139000 + (195909 << 12) + 0x8b8
(long) $3 = 0x00000001bbe7e8b8
(lldb) memory read 0x00000001bbe7e8b8
0x1bbe7e8b8: 80 23 15 8b 01 00 00 00 38 06 03 8b 01 00 00 00  .#......8.......
0x1bbe7e8c8: f0 23 15 8b 01 00 00 00 1c 1c 15 8b 01 00 00 00  .#..............
(lldb) dis -a 0x018b152380
libsystem_platform.dylib`_platform_strlen:
    0x18b152380 <+0>:  and    x1, x0, #0xfffffffffffffff0
    0x18b152384 <+4>:  ldr    q0, [x1]
```

一顿操作后发现原来是 `strlen`，如果你一味地轻信 Xcode 显示的注释，分析就无法进行下去了，这告诉我们做事情一定要抱着怀疑的态度。



## 参考文档

Apple提供的[System Call Table](https://www.theiphonewiki.com/wiki/Kernel_Syscalls)
ARM公司提供的[官方文档](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0802a/STUR_fpsimd.html)
[iOS调试进阶](https://zhuanlan.zhihu.com/c_142064221)
[iOS开发同学的arm64汇编入门](https://blog.cnbluebox.com/blog/2017/07/24/arm64-start/)
[volatile 与内存屏障总结](https://zhuanlan.zhihu.com/p/43526907)

