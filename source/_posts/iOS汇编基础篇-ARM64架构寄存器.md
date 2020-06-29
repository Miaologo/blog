---
title: iOS汇编基础篇-ARM64架构寄存器和指令
date: 2020-05-24 11:44:34
tags:
	- iOS
	- arm64
---



# ARM64 汇编基础

> ARM的全称是Advanced RISC Machine，翻译过来是高级精简指令集机器。
>
> iOS设备CPU架构都是基于ARM的，比如经常看到的一些名词：arm64，arm7…它们指的都是CPU指令集。iPhone 5s及以后的iOS设备的CPU都是ARM 64架构的。

## 术语介绍

| 术语                                                    | 意义                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| A32                                                     | 在ARMv7架构中，使用32位固定长度指令的ARM指令集。             |
| A64                                                     | AArch64可用时的指令集。                                      |
| AAPCS64                                                 | AArch64程序调用标准。（PCS：Procedure Call Standard）        |
| AArch32                                                 | ARMv8中的32位通用寄存器，兼容ARMv7-A。                       |
| AArch64                                                 | ARMv8中的64位通用寄存器                                      |
| ABI（Application Binary Interface）                     | 汇编接口规范，跟执行环境相关，比如Linux ABI，说的是Linux环境下的汇编接口规范； |
| ARM-based                                               | 基于ARM                                                      |
| Floating point                                          | 根据上下文有这三种意思：（1）遵循IEEE 754 2008的浮点运算; （2）ARMv8浮点指令集; （3）一个被ARMv8浮点指令集和ARMv8 SIMD指令集共享的寄存器组。 |
| Q-o-I                                                   | Quality of Implementation                                    |
| SIMD                                                    | Single Instruction Multiple Data 一条指令操作多个数据        |
| T32                                                     | T32使用可变16bit和32bit                                      |
| Routine, subroutine                                     | Routine：调用者；subroutine：被调用者                        |
| Procedure                                               | 没有返回值的函数                                             |
| Function                                                | 有返回值的函数                                               |
| PIC, PID                                                | Position-independent code, position-independent data.        |
| Program state                                           | 指程序内存和寄存器的值                                       |
| Caller- saved register                                  | 调用者在调用函数之前，保存寄存器（一般入栈），函数返回后恢复寄存器（一般出栈） |
| Callee-saved register                                   | 被调用者（函数内部），在起始地方保存寄存器，在结束时，恢复寄存器 |
| NGRN（The Next General-purpose Register Number ）       | 可以理解为，记录x0-x7（见下文寄存器）使用个数，参数传递前设为0，每放一个参数进入寄存器（整型寄存器），值加1。当等于8时候，说明x0-x7寄存器使用完了，再有参数，只能放入内存了。 |
| NSRN (The Next SIMD and Floating-point Register Number) | 同上，记录x0-x7使用个数                                      |
| NSAA （The next stacked argument address）              | 记录参数放入内存，参数传递前设为SP，所以内存中参数范围应该是 sp~NSAA。详细见下文参数传递 |



##基本数据类型和对齐

| Type Class           | Machine Type   | Byte size | Natural Alignment (bytes) |
| -------------------- | -------------- | --------- | ------------------------- |
| Integral             | Unsigned byte  | 1         | 1                         |
| Signed byte          | 1              | 1         |                           |
| Unsigned half- word  | 2              | 2         |                           |
| Signed half- word    | 2              | 2         |                           |
| Unsigned word        | 4              | 4         |                           |
| Signed word          | 4              | 4         |                           |
| Unsigned double-word | 8              | 8         |                           |
| Signed double- word  | 8              | 8         |                           |
| Unsigned quad- word  | 16             | 16        |                           |
| Signed quad- word    | 16             | 16        |                           |
| Floating Point       | Half precision | 2         | 2                         |
| Single precision     | 4              | 4         |                           |
| Double precision     | 8              | 8         |                           |
| Quad precision       | 16             | 16        |                           |
| Short vector         | 64-bit vector  | 8         | 8                         |
| 128-bit vector       | 16             | 16        |                           |
| Pointer              | Data pointer   | 8         | 8                         |
| Code pointer         | 8              | 8         |                           |

# 程序调用规则

## 进程、内存、栈

一个进程的内存可分为5类：

1. 代码区。只能被进程读，不可写。
2. 可写静态数据。
3. 只读静态数据。
4. 堆。
5. 栈。

可写静态数据可以细分为初始化，零初始化和未初始化数据。 除了栈之外，其它4类内存不需要占用连续的内存。 进程必须具有一些代码和栈，其它3类不是必须有。 堆是由进程管理的内存区域， 通常用于创建动态数据对象。

### 内存地址

地址空间包括一个或多个不相交的区域。 区域不能跨越零地址，但是可以从零开始。 标记寻址（tagged addressing）的使用是特定平台解释的。 当禁用标记寻址时，指针的所有64位都被传递到地址转换系统。 启用标记寻址时，为了进行地址转换，将忽略指针的前八位。注意：此tagged addressing，非iOS里的Tagged Pointer。

### 栈

栈是连续的内存空间，可用于存储局部变量和参数传递（用于传递参数的寄存器不够用时候）。栈地址是从高到低，栈的地址保存在SP中。 栈使用限制：

1. Stack-limit < SP <= stack-base
2. 进程只能访问这个范围内的栈空间：[SP, stack-base – 1]
3. SP mod 16 = 0

## 函数调用

A64指令集包含函数调用指令BL和BLR。 执行BL：PC（program counter）顺序的下一个值，也就是返回地址（函数调用完成返回要执行指令的地址），存放到LR中，将跳转地址传给PC。BLR跟BL类似，只不过PC的值是从寄存器中读取。

## 参数传递

参数可通过x0-x7、q0-q7，栈来传递；如果参数个数不多，且参数可放进寄存器，那仅用寄存器传递参数。

### 可变参数

可变参数可分为命名参数（已声明的）和匿名参数（可选的参数）。 当可变参数的函数，调用时候，没有可选参数时候（只有已声明的参数），调用过程和固定参数的函数一样的。

### 参数传递规则

参数传递从概念上可以分为2阶段：

1. 从源语言参数类型到机器类型的映射（不同源语言，映射规则不同）
2. 整理机器类型，生成最终参数列表

参数传递过程分为3个阶段：

- 阶段A – 初始化 （在开始处理参数之前，该阶段仅执行一次）
  1. NGRN = 0  （NGRN意义，见术语）
  2. NSRN = 0   （NSRN意义，见术语）
  3. NSAA = SP（NSAA意义，见术语）
- 阶段B - 预填充和扩展参数  （把参数列表中的每一个参数，去匹配下面规则，第一个被匹配到的规则，应用到该参数上。）
  1. 如果参数类型是复合类型，调用者和被调用者都不能确定其大小，则将参数复制到内存中，并将参数替换为指向该内存的指针。 （C / C ++语言中没有这样的类型，其它语言存在。）
  2. 如果参数是HFA或HVA类型，则参数不修改。
  3. 如果参数是大于16个字节的复合类型，调用者申请一个内存，将参数复制到内存里去，并将参数替换为指向该内存的指针。
  4. 如果参数是复合类型，则参数的大小向上舍入为最接近8个字节的倍数。（例如参数大小为9字节，修改为16字节）
- 阶段C- 把参数放到寄存器或栈里  （参数列表中的每个参数，将依次应用以下规则，直到参数放到寄存器或栈里，此参数处理完成，然后再从参数列表中取参数。注： 将参数分配给寄存器时，寄存器中未使用的位的值不确定。 将参数分配给栈时，未填充字节的值不确定。）
  1. 如果参数是half(16bit)，single(16bit)，double(32bit)或quad(64bit)浮点数或Short Vector Type，并且NSRN小于8，则将参数放入寄存器q[NSRN]的最低有效位。 NSRN增加1。 此参数处理完成。
  2. 如果参数是HFA(homogeneous floating-point aggregate)或HVA(homogeneous short vector aggregate)类型，且NSRN + （HFA或HVA成员个数） ≤ 8，则每个成员依次放入SIMD and Floating-point 寄存器，NSRN=NSRN+ HFA或HVA成员个数。此参数处理完成。
  3. 如果参数是HFA(homogeneous floating-point aggregate)或HVA(homogeneous short vector aggregate)类型，但是NSRN已经等于8（说明x0-x7被使用完毕）。则参数的大小向上舍入为最接近8个字节的倍数。（例如参数大小为9字节，修改为16字节）
  4. 如果参数是HFA(homogeneous floating-point aggregate)、HVA(homogeneous short vector aggregate)、quad(64bit)浮点数或Short Vector Type，NSAA = NSAA+max(8, 参数自然对齐大小)。
  5. 如果参数是half(16bit)，single(16bit)浮点数，参数扩展到8字节（放入最低有效位，其余bits值不确定）
  6. 如果参数是HFA(homogeneous floating-point aggregate)、HVA(homogeneous short vector aggregate)、half(16bit)，single(16bit)，double(32bit)或quad(64bit)浮点数或Short Vector Type，参数copy到内存，NSAA=NSAA+size（参数）。此参数处理完成。
  7. 如果参数是整型或指针类型、size(参数)<=8字节，且NGRN小于8，则参数复制到x[NGRN]中的最低有效位。 NGRN增加1。 此参数处理完成。
  8. 如果参数对齐后16字节，NGRN向上取偶数。（例如：NGRN为2，那值保持不变；假如NGRN为3，则取4。 注：iOS ABI没有这个规则）
  9. 如果参数是整型，对齐后16字节，且NGRN小于7，则把参数复制到x[NGRN] 和 x[NGRN+1]，x[NGRN]是低位。NGRN = NGRN + 2。 此参数处理完成。
  10. 如果参数是复合类型，且参数可以完全放进x寄存器（8-NGRN>= 参数字节大小/8）。从x[NGRN]依次放入参数（低位开始）。未填充的bits的值不确定。NGRN = NGRN + 此参数用掉的寄存器个数。此参数处理完成。
  11. NGRN设为8。
  12. NSAA = NSAA+max(8, 参数自然对齐大小)。
  13. 如果参数是复合类型，参数copy到内存，NSAA=NSAA+size（参数）。此参数处理完成。
  14. 如果参数小于8字节，参数设置为8字节大小，高位bits值不确定。
  15. 参数copy到内存，NSAA=NSAA+size（参数）。此参数处理完成。

从上面规则，可以得到经验：

1. 处理完参数列表中所有的参数后，调用者一定知道传递参数用了多少栈空间。（NSAA - SP）
2. 浮点数和short vector types通过q寄存器和栈传递，不会通过x寄存器传递。（除非是小复合类型的成员）
3. 寄存器和栈中，参数未填充满的部分的值，不可确定。

## 函数返回结果

函数返回方式取决于返回结果的类型。

1. 如果返回是类型T，如下

```c
If the type, T, of the result of a function is such that
void func(T arg)
would require that arg be passed as a value in a register (or set of registers) according to the rules in §5.4 Parameter Passing, then the result is returned in the same registers as would be used for such an argument.
```

arg值通过寄存器（组）传递，返回的结果也是通过相同的寄存器（组）返回。 2. 调用者申请内存（内存大小足够放入返回结果且是内存对齐的），将内存地址放入x8中传递给子函数，子函数运行时候，可以更新x8指向内存的内容，从而将结果返回。



## 寄存器

### 通用寄存器

`31` 个寄存器 `R0 ~ R30`，每个寄存器可以存取一个 **64** 位大小的数。 
当使用 `x0 - x30`访问时，是一个 64位的数；
当使用 `w0 - w30`访问时，是一个 32 位的数，访问的是寄存器的 **低 `32` 位**，如图：

[![img](http://chevereto.shenyuanluo.com/images/CommonRegister.png)](http://chevereto.shenyuanluo.com/images/CommonRegister.png)

### 向量寄存器

由于浮点数运算的特殊性，ARM 64还有32个浮点数寄存器q0~q31，长度不同称谓也不同，b，h，s，d，q，分别代表byte(8位)，half(16位)，single(32位)，double(64位)，quad(128位)。

q0-q7在函数调用过程中传递参数和返回值；q8-q15 是Callee-saved registers（见术语解释），且是保存前64bits（更大的位数，调用者负责保存），q0-q7, q16-q31不需要保存或者调用者保存。

（也可以说是 **浮点型寄存器**）每个寄存器的大小是 `128` 位的。 分别可以用`Bn Hn Sn Dn Qn`的方式来访问不同的位数；如图:

[![img](http://chevereto.shenyuanluo.com/images/VectorRegister.png)](http://chevereto.shenyuanluo.com/images/VectorRegister.png)

> **注：**word 是 `32` 位，也就是 4 Byte大小。

- **Bn：**一个 Byte的大小，即 `8` 位
- **Hn：**half word，即 `16` 位
- **Sn：**single word，即 `32` 位
- **Dn：**double word，即 `64` 位
- **Qn：**quad word，即`128` 位

### 特殊寄存器

- **sp：** (Stack Pointer)，栈顶寄存器，用于保存栈顶地址；
- **fp(x29)：** (Frame Pointer)为栈基址寄存，用于保存栈底地址；
- **lr(x30)：** (Link Register) ，保存调用跳转指令 *`bl`* 指令的下一条指令的内存地址；
- **zr(x31)：** (Zero Register)，`xzr/wzr`分别代表 64/32 位，其作用就是 0，写进去代表丢弃结果，读出来是 `0`；
- **pc：**保存将要执行的指令的地址（有操作系统决定其值，不能改写）。

### 状态寄存器 CPSR

> **CPSR** (Current Program Status Register)和其他寄存器不一样，其他寄存器是用来存放数据的，都是整个寄存器具有一个含义；而 **CPSR** 寄存器是按位起作用的，即，每一位都有专门的含义，记录特定的信息；如下图
>
> **注：** CPSR 寄存器是 **32** 位的。

[![img](http://chevereto.shenyuanluo.com/images/CPSR.png)](http://chevereto.shenyuanluo.com/images/CPSR.png)

1. `CPSR` 的 **低8位**（包括 `I`、`F`、`T` 和 `M[4：0]`）称为控制位，程序无法修改，除非 CPU 运行于 **特权模式** 下，程序才能修改控制位。
2. `N`、`Z`、`C`、`V` 均为条件码标志位；其内容可被算术或逻辑运算的结果所改变，并且可以决定某条指令是否被执行。
   - **N（Negative）标志：**CPSR 的第 `31` 位是 N，*符号标志位*；记录相关指令执行后其结果是否为负数，**如果为负数，则 `N = 1`；如果是非负数，则 `N = 0`**。
   - **Z(Zero)标志：**`CPSR` 的第 `30` 位是 Z，*0标志位*；记录相关指令执行后，其结果是否为0，**如果结果为0，则 `Z = 1`；如果结果不为0，则 `Z = 0`**。
   - **C(Carry)标志：**CPSR 的第 `29` 位是C，*进位标志位*；
     - 加法运算：当运算结果产生了 **进位** 时（无符号数溢出），`C = 1`，否则 `C = 0` ；
     - 减法运算（包括 `CMP`）： 当运算时产生了 **借位** 时（无符号数溢出），`C = 0`，否则 `C = 1` 。
   - **V(Overflow)标志：**CPSR 的第 28` 位是 V，*溢出标志位*；在进行有符号数运算的时候，如果超过了机器所能标识的范围，称为溢出。

### 条件码列表

| 操作码 |         条件码助记符          |   标志   |        含义        |
| :----: | :---------------------------: | :------: | :----------------: |
|  0000  |              EQ               |   Z=1    |        相等        |
|  0001  |         NE(Not Equal)         |   Z=0    |       不相等       |
|  0010  | CS/HS(Carry Set/High or Same) |   C=1    | 无符号数大于或等于 |
|  0011  |   CC/LO(Carry Clear/LOwer)    |   C=0    |    无符号数小于    |
|  0100  |           MI(MInus)           |   N=1    |        负数        |
|  0101  |           PL(PLus)            |   N=0    |      正数或零      |
|  0110  |       VS(oVerflow set)        |   V=1    |        溢出        |
|  0111  |      VC(oVerflow clear)       |   V=0    |      没有溢出      |
|  1000  |           HI(High)            | C=1,Z=0  |    无符号数大于    |
|  1001  |       LS(Lower or Same)       | C=0,Z=1  | 无符号数小于或等于 |
|  1010  |     GE(Greater or Equal)      |   N=V    | 有符号数大于或等于 |
|  1011  |         LT(Less Than)         |   N!=V   |    有符号数小于    |
|  1100  |       GT(Greater Than)        | Z=0,N=V  |    有符号数大于    |
|  1101  |       LE(Less or Equal)       | Z=1,N!=V | 有符号数小于或等于 |
|  1110  |              AL               |   任何   |  无条件执行(默认)  |
|  1111  |              NV               |   任何   |      从不执行      |

### 指令读取

在 `arm64` 架构中，每个指令读取都是 *64* 位，即 **`8`字节 空间**。

### 函数、汇编、寄存器 ， arm64 约定（一般来说）

|  寄存器   | 特殊用途 |                             作用                             |
| :-------: | :------: | :----------------------------------------------------------: |
|    sp     |          |                    Stack pointer：栈指针                     |
|    x30    |    LR    |  Link Register：在调用函数时候，保存下一条要执行指令的地址   |
|    x29    |    FP    |              Frame Pointer：保存函数栈的基地址               |
| x19...x28 |          |                    Callee-saved registers                    |
|    x18    |          | 平台寄存器，有特定平台解释其用法。如果平台未把其做特殊用途，可当做临时寄存器使用。（iOS平台保留的寄存器，应用不可使用） |
|    x17    | PL(PLus) | The second intra-procedure-call temporary register(can be used by call veneers and PLT code); at other time may be used as a temporary register |
|    x16    |   IP0    | The first intra-procedure-call scratch register(can be used by call veneers and PLT code); at other time may be used as a temporary register |
| x9...x15  |          |               Caller-saved temporary registers               |
|    x8     |          |         在一些情况下，返回值是通过r8返回的，C=1,Z=0          |
|  x0...x7  |          |  用来参数传递给子程序或者从函数返回值，也可以用来存储中间值  |
|   NZCV    |          | 状态寄存器：N（Negative）负数 Z(Zero) 零 C(Carry) 进位 V(Overflow) 溢出 |



- `x0 ~ x7` 分别会存放方法的前 8 个参数；如果参数个数超过了8个，多余的参数会存在栈上，新方法会通过栈来读取。
- 方法的返回值一般都在 x0 上；如果方法返回值是一个较大的数据结构时，结果会存在 x8 执行的地址上。

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

```asm
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

```asm
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

# iOS ABI

在iOS平台上有几个不同点需要加以注意下。

## iOS platform designers

1. x18寄存器为平台保留，程序不可用。

2. wchar_t类型是32bit， long类型是64bit。

3. x29（FP：保存函数栈的基地址）必须总是有意义的。

4. 空结构体，在函数调用的参数中被忽略。

   ![empty-Struct](https://user-gold-cdn.xitu.io/2019/6/27/16b97a75bbbda811)

## iOS与官方arm64不同点

### 参数传递

1. 在arm64标准中，当参数是通过栈传递时（比如参数超过8个），每个参数消耗8个字节的倍数。但是iOS中删除了这个要求。 例如：

```c
void two_stack_args(char w0, char w1, char w2, char w3, char w4, char w5, char w6, char w7, char s0, char s1) {}
```

s0在sp处占用1个字节，s1在sp + 1处占用1个字节。然后填充满足内存对齐（sp必须是16的倍数）。

 2. 在arm64标准中，当传递16字节对齐的参数时，从偶数寄存器xN开始。但是iOS中，没有这个要求。例如：

```c
void large_type(int x0, __int128 x1_x2) {} 
```

在iOS平台，参数x1_x2在x1和x2中传递；arm64标准里，参数应该在x2和x3中传递。 

3. 在arm64标准中，被调用者负责对少于32bits的参数进行0扩展或者标记；在iOS中，调用者负责扩展至32bits。

### 可变参数

iOS ABI和arm64标准完全不同。

1. 可变参数中的匿名参数都是通过栈传递的。（每个可变参数分配8字节倍数栈空间，同时注意sp%16=0），固定参数传递方式跟arm64标准一致。

   **由于参数传递方式不同，可变参数函数，是不能直接用函数指针赋值，然后当做固定参数函数调用。** 

   例如：

```
typedef int __cdecl (*PInvokeFunc) (const char*, int);

int test()
{
    PInvokeFunc fp = (PInvokeFunc)printf;
    fp("Hello World: %d", 10); //不一定打印出Hello World: 10
    return 0;
}
复制代码
```

解决办法是：通过IL2CPP生成包装函数。

2. `C`语言在函数调用之前，小于`int`类型的参数，进行提升。注意，栈空间上未填充的bit的值不确定（`arm64`标准里，寄存器和栈上未填充的bit值，也是不确定的）。
3. `va_list`(可变参数函数里，获取可变参数列表的类型)就是 `char*` 。

### 基本C语言类型

1. long double是double(32bit)精度，而不是quad(64bit)精度。
2. char 和 wchar_t 有符号的。

### 数据类型和内存对齐

| Data type  | Size (in bytes) | Natural alignment (in bytes) |
| ---------- | --------------- | ---------------------------- |
| BOOL, bool | 1               | 1                            |
| char       | 1               | 1                            |
| short      | 2               | 2                            |
| int        | 4               | 4                            |
| long       | 8               | 8                            |
| long long  | 8               | 8                            |
| pointer    | 8               | 8                            |
| size_t     | 8               | 8                            |
| NSInteger  | 8               | 8                            |
| CFIndex    | 8               | 8                            |
| fpos_t     | 8               | 8                            |
| off_t      | 8               | 8                            |




# 汇编指令



### GNU平台无关

> **ARM64汇编为了提高访问效率要求按照16字节进行对齐**

#### 符号定义伪指令

```asm
.global`,`.local`,`.set`,`.equ
```

##### .global

使得符号对连接器可见，变为对整个工程可用的全局变量

```asm
.global symbol
```

`.globl` 可以让一个标签对链接器可见，可以供其他链接对象模块使用。 换句话说 `.global _my_func` 可以在代码中调用到 `my_func`。

##### .local

表示符号对外部不可见，只对本文件可见

```asm
.local symbol
```

##### .set

给一个全局变量或局部变量赋值，和`.equ`的功能一样

```asm
.set symbol expr
.set start, 0x40
.set start, 0x50
mov r1, #start      ;r1里面是0x50
```

##### .equ

和`.set`一样，只是格式不同

```asm
symbol .equ  expr
start  .equ, 0x40
start  .equ, 0x50
mov r1, #start      ;r1里面是0x50
```

#### 数据定义伪指令

```asm
.byte`,`.short`,`.long`,`.quad`,`.float`,`.string`,`.asciz`,`.ascii`,`.rept
```

##### .byte

在存储器中分配**1个字节**，用指定的数据对存储单元进行初始化

```asm
label:  .byte   expr    ;label是程序标号，expr可以是-128~255的数字，也可是字符
a:  .byte   #1  ;等价于C中的char a=1;
```

##### .short

在存储器中分配**2个字节**，用指定的数据对存储单元进行初始化

```asm
a: .short 0x1234
```

##### .word / .long

在存储器中分配**4个字节**，用指定的数据对存储单元进行初始化

```asm
a: .word 0x12345678
```

##### .long

在存储器中分配**个字节**，用指定的数据对存储单元进行初始化

##### .quad

在存储器中分配**8个字节**，用指定的数据对存储单元进行初始化

```asm
a: .quad 0x12345678 ;等价于C中的long a=0x1234567812345678
```

##### .float

在存储器中分配**4个字节**，用指定的**浮点**数据对存储单元进行初始化

```asm
a: .float 1.11
```

##### .space/.skip

用于分配一块连续的存储区域并初始化为指定的值，如果后面的填充值省略不写则在后面填充为0;

```asm
label: .space size,expr     ;expr可以是4字节以内的浮点数 
a:  space 8, 0x1
```

##### .string

定义一个字符串，默认是string8，还有string16，string32，string64

```asm
a: .space "hello world!"
```

##### .rept

重复执行接下来的指令，以.rept开始，以.endr结束

```asm
.rept cnt   ;cnt是重复次数
...
.endr
```

`.rept` 和 `.endr` 是一组循环伪指令。

```asm
func_1:
.rept 5
add x0, x0, x1
.endr
ret
```

上述指令得到内容如下： ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-4.8d465430.png) 生成了 5 个连续的 `add` 指令。

#### 汇编控制伪操作

流程控制伪指令主要yy`.if .else .endif` `.macro .endm .exitm`

##### .if .else .endif

```asm
.if logical-expression
...
.elseif logical-expression2
...
.else
...
.endif
```

##### .macro .endm .exitm

该伪指令可以将一段代码定义为一个整体，称为宏指令，然后就可以在程序中通过宏指令多次调用该段代码，而`.exitm`指令用来退出当前的宏指令，宏指令可以使用一个或多个参数，当宏操作被展开时，这些参数被相应的值替换。
包含在`.macro`和`。endm`之间的指令序列称为宏定义体。在宏定义体的第一行应声明宏的原型，包含宏名所需的参数，然后就可以在汇编程序中通过宏名来调用该指令序列，在源程序被编译时，汇编器将宏调用展开，用宏定义中的指令序列代替程序中的宏调用，并将实际参数的值传递给宏定义中的形式参数

```asm
.macro macroname macargs ...
;code
.endm
```

#### 杂项

```asm
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

##### .text

`.text` 用于声明以下是代码段。

##### .align

`.align` 用于将指令对齐到内存地址，对齐位置为参数的 2 的幂次方。 以 `.align 10` 举例，在没有添加对齐时：

```asm
func_1:
ret
func_2:
ret
```

得到的结果，两个指令构成了连续的地址： ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-1.90fdbc4b.png)

在 `func_1` 和 `func_2` 之间加入 `.align 10`：

```asm
func_1:
ret
.align 10
func_2:
ret
```

得到如下结果，如果 `.align` 只是个普通的指令，那 `func_2` 的 `ret` 对应地址应当为 `0x0000000100008060`，这里却变成了 `0x000000010000840`，`0x000000010000840`刚好是 2^10 的倍数。 ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-2.d1447eed.png)

在本文中，会有一个妙用点，我们在 `func_1` 也加入 `.align 10`： ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-3.77e22018.png) 可以看到 `0x8800 - 0x8400 = 0x400`，0x400 刚好是 2^10，这说明我只要知道 `func_2` 的 `ret` 指令内存地址，就可以得到 `func_1` 的 `ret` 指令地址。

#### 标签

如下代码：

```asm
_my_label:
add x0, x0, x1
```

`_my_label` 会指向 `add x0, x0, x1` 的地址，用于辅助标记命令地址。

### 常用汇编指令

#### nop

什么也不做的指令，愣一下。没什么用途，但可以偏移指令地址。 ![img](https://blog.dianqk.org/assets/img/trampolinehook-study-notes-5.4799e6e3.png) 可以看到 nop 也占了 4 个字节，原本 `0000000100013fe4` 对应的指令应当是 ret。

#### **mov：** 

将某一寄存器的值复制到另一寄存器（只能用于寄存器与寄存器或者寄存器与常量之间传值，不能用于内存地址），如：

  ```asm
mov x1, x0 							; 将寄存器 x0 的值复制到寄存器 x1 中
  ```

#### **add：** 
将某一寄存器的值和另一寄存器的值 *相加* 并将结果保存在另一寄存器中，如：

  ```asm
add x0, x0, #1 					; 将寄存器 x0 的值和常量 1 相加后保存在寄存器 x0 中 
add x0, x1, x2 					; 将寄存器 x1 和 x2 的值相加后保存到寄存器 x0 中 
add x0, x1, [x2] 				; 将寄存器 x1 的值加上寄存器 x2 的值作为地址，再取该内存地址的内容放入寄存器 x0 中
  ```

#### **sub：** 
将某一寄存器的值和另一寄存器的值 *相减* 并将结果保存在另一寄存器中，如：

  ```asm
sub x0, x1, x2 					; 将寄存器 x1 和 x2 的值相减后保存到寄存器 x0 中
  ```

#### **mul：**
将某一寄存器的值和另一个寄存器的值 *相乘* 并将结果保存在另一寄存器中，如：

  ```asm
mul x0, x1, x2  				; 将寄存器 x1 和 x2 的值相乘后结果保存到寄存器 x0 中
  ```

#### **sdiv：**（有符号数，对应 **udiv**: 无符号数）
将某一寄存器的值和另一个寄存器的值 *相除* 并将结果保存在另一寄存器中，如：

  ```asm
sdiv x0, x1, x2 				; 将寄存器 x1 和 x2 的值相除后结果保存到寄存器 x0 中
  ```

#### **and：** 
将某一寄存器的值和另一寄存器的值 *按位与* 并将结果保存到另一寄存器中，如：

  ```asm
and x0, x0, #0xf 				; 将寄存器 x0 的值和常量 0xf 按位与后保存到寄存器 x0 中
  ```

#### **orr：** 
将某一寄存器的值和另一寄存器的值 *按位或* 并将结果保存到另一寄存器中，如：

  ```asm
orr x0, x0, #9 					; 将寄存器 x0 的值和常量 9 按位或后保存到寄存器 x0 中
  ```

#### **eor：** 
将某一寄存器的值和另一寄存器的值 *按位异或* 并将结果保存到另一寄存器中，如：

  ```asm
eor x0, x0, #0xf 				; 将寄存器 x0 的值和常量 0xf 按位异或后保存到寄存器 x0 中
  ```

#### **str：** (store register) 
将寄存器中的值写入到内存中，如：

  ```asm
str w9, [sp, #0x8] 			; 将寄存器 w9 中的值保存到栈内存 [sp + 0x8] 处

str x8, [sp, #-16]! ;将 x8 存到 sp - 16 的位置，并将 sp -= 16
stp x4, x5, [sp, #-16]! ;将 x4 x5 的值存到 sp - 16 的位置，并将 sp -= 16
  ```

这里涉会及到一些寻址的格式，有 3 种方式：

```asm
[x10, #0x10] ;从 x10 + 0x10 的地址取值
[sp, #-16]! ;从 sp - 16 地址取值，取值完后在把 sp 向低地址移 16 字节，即开辟一段新栈空间
[sp], #16 ;从 sp 地址取值，取值完后在把 sp 向高地址偏移 16 字节，即释放一些栈空间
```

| Example             | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `LDR X0, [X1, #8]`  | Load from address X1 + 8                                     |
| `LDR X0, [X1, #8]!` | Pre-index: Update X1 first (to X1 + #8), then load from the new address |
| `LDR X0, [X1], #8`  | Post-index: Load from the unmodified address in X1 first, then update X1 (to X1 + #8) |

#### **strb：** (store register byte) 

将寄存器中的值写入到内存中（只存储一个字节），如：

  ```asm
strb w8, [sp, #7]				; 将寄存器 w8 中的低 1 字节的值保存到栈内存 [sp + 7] 处
  ```

#### **ldr：** (load register) 
将内存中的值读取到寄存器中，如：

  ```asm
ldr x0, [x1] 						; 将寄存器 x1 的值作为地址，取该内存地址的值放入寄存器 x0 中 
ldr w8, [sp, #0x8] 			; 将栈内存 [sp + 0x8] 处的值读取到 w8 寄存器中 
ldr x0, [x1, #4]! 			; 将寄存器 x1 的值加上 4 作为内存地址, 取该内存地址的值放入寄存器 x0 中, 然后将寄存器 x1 的值加上 4 放入寄存器 x1 中 
ldr x0, [x1], #4 				; 将寄存器 x1 的值作为内存地址，取内该存地址的值放入寄存器 x0 中, 再将寄存器 x1 的值加上 4 放入寄存器 x1 中 
ldr x0, [x1, x2] 				; 将寄存器 x1 和寄存器 x2 的值相加作为地址，取该内存地址的值放入寄存器 x0 中
  ```

这里涉会及到一些寻址的格式，有 3 种方式：

```asm
[x10, #0x10] ; 从 x10 + 0x10 的地址取值
[sp, #-16]! ;  从 sp - 16 地址取值，取值完后在把 sp 向低地址移 16 字节，即开辟一段新栈空间
[sp], #16 ;    从 sp 地址取值，取值完后在把 sp 向高地址偏移 16 字节，即释放一些栈空间
```

#### **ldrsb：** (load register byte) 

将内存中的值（只读取一个字节）读取到寄存器中，如：

  ```asm
ldrsb	w8, [sp, #7]			; 将栈内存 [sp + 7] 出的 低 1 字节的值读取到寄存器 w8 中
  ```

#### **stur：**
同 `str` 将寄存器中的值写入到内存中（一般用于 `负` 地址运算中），如：

  ```asm
stur w10, [x29, #-0x4] 	; 将寄存器 w10 中的值保存到栈内存 [x29 - 0x04] 处
  ```

#### **ldur：**
同 `ldr` 将内存中的值读取到寄存器中（一般用于 `负` 地址运算中），如：

  ```asm
ldur w8, [x29, #-0x4]   ; 将栈内存 [x29 - 0x04] 处的值读取到 w8 寄存器中
  ```

#### **stp：** 
入栈指令（`str` 的变种指令，可以同时操作两个寄存器），如：

  ```asm
stp x29, x30, [sp, #0x10] 	; 将 x29, x30 的值存入 sp 偏移 16 个字节的位置
  ```

#### **ldp：** 
出栈指令（`ldr` 的变种指令，可以同时操作两个寄存器），如：

  ```asm
ldp x29, x30, [sp, #0x10] 	; 将 sp 偏移 16 个字节的值取出来，存入寄存器 x29 和寄存器 x30

ldr x8, [sp], #16 ;将 sp 位置的值取出来，存入 x8 中，并将 sp += 16
ldp x4, x5, [sp], #16 ;将 sp 位置的值取出来，存入 x4 x5 中，并将 sp += 16
  ```

#### **scvtf：** (Signed Convert To Float)
带符号 *定点数* 转换为 *浮点数*，如：

  ```asm
scvtf	d1, w0			; 将寄存器 w0 的值(顶点数，转化成 浮点数) 保存到 向量寄存器/浮点寄存器 d1 中
  ```

#### **fcvtzs：**(Float Convert To Zero Signed)
*浮点数* 转化为 *定点数* （舍入为0），如：

  ```asm
fcvtzs w0, s0			; 将向量寄存器 s0 的值(浮点数，转换成 定点数)保存到寄存器 w0 中
  ```

#### **cbz：** 
和 0 比较（Compare），如果结果为零（Zero）就转移（只能跳到后面的指令），如：

  ```asm
cbz  x8, loc_1800b4530	; 将寄存器 x8 的值和 0 比较，如果结果为 “0” 则跳转到 ‘loc_1800b4530’ 标签处开始指令
  ```

#### **cbnz：** 
和非 0 比较（Compare），如果结果非零（Non Zero）就转移（只能跳到后面的指令），如：

  ```asm
cbnz  x9, loc_1800b4530	; 将寄存器 x9 的值和 0 比较，如果结果为 “非 0” 则跳转到 ‘loc_1800b4530’ 标签处开始指令
  ```

#### **cmp：** 
比较指令，相当于 `subs`，影响程序状态寄存器 *CPSR* ;

#### **cset：**
比较指令，满足条件，则并置 `1`，否则置 `0` ，如：

  ```asm
  cmp	 w8, #2    	; 将寄存器 w8 的值和常量 2 进行比较
  cset w8, gt			; 如果是大于(grater than)，则将寄存器 w8 的值设置为 1，否则设置为 0
  ```

#### **csel**

 `csel` 指令，该指令是 ARM 中的三目运算：

```asm
csel   x21, x21, xzr, ne

csel  Wd, Wn, Wm, cond 
# 等价于
Wd = cond ? Wn : Wm
```




#### **LSL：** 逻辑左移

#### **LSR：** 逻辑右移

#### **ASR：** 算术右移

#### **ROR：** 循环右移

#### **adrp：** 
用来定位数据段中的数据用, 因为 *aslr* 会导致代码及数据的地址随机化, 用 *adrp* 来根据 *pc* 做辅助定位

例如我们要找到 counter 变量，本质上是计算当前指令距离 counter 变量的距离，即计算基于 PC 的偏移量，能表示的偏移量的最大长度决定了能够寻址的空间大小，可以想象，如果代码和数据段之间的距离过大，将难以通过一次运算进行寻址。计算 counter 变量地址的过程如下。

1. 使用 adrp 命令计算出 _counter label 基于 PC 的偏移量的高 21 位，并存储在 x8 寄存器中，@PAGE 代表页偏移的高 21 位；

   ```asm
   adrp	x8, _counter@PAGE
   ```

2. 使用 add 命令将余下的 12 位补齐，通过 @PAGEOFF 代表页偏移的低 12 位；

   ```asm
   add	x8, x8, _counter@PAGEOFF
   ```

3. 此时，x8 中即为 counter 变量的实际地址了，通过 ldr 命令将寄存器的值读取到 w0 中，作为函数返回值。

   ```asm
   ldr	w0, [x8]
   ret
   ```

看到这里，相信你会有个很大的疑问，为什么不能一次性的将地址加载到 x8，而要拆分成高 21 位和低 12 位呢，这是因为 ARM64 虽然支持 64 位地址，但指令的长度仅有 32 位，因此难以通过一条指令去编码 64 位地址，所以才拆解成了 adrp + add 的组合，从而支持了正负 32 位地址偏移量范围的寻址。

想深入了解基于 PC 的寻址，可以阅读 [What are @PAGE and @PAGEOFF symbols in IDA?](https://reverseengineering.stackexchange.com/questions/14385/what-are-page-and-pageoff-symbols-in-ida) 中的高票回答

```asm
adrp x0, 1
```

> ADRP 是计算指定的数据地址 到当前PC值的相对偏移（eg：adrp x0, 1）
> 1.将1的值,左移12位 1 0000 0000 0000 == 0x1000
> 2.将PC寄存器的低12位清零 0x1002e6874 ==> 0x1002e6000
> 3.将将1 和 2 的结果相加 给 X0 寄存器!!



#### **b：** （branch）
跳转到某地址（无返回）, 不会改变 *lr (x30)* 寄存器的值；一般是本方法内的跳转，如 `while` 循环，`if else` 等 ，如：

  ```asm
b	LBB0_1; 直接跳转到标签 ‘LLB0_1’ 处开始执行
  ```

函数跳转的指令是`b/br`， 其中`b`指令的操作数是距离当前位置相对距离的偏移地址，`br`指令的操作数则是寄存器，表明跳转到寄存器所指定的地址中去。下面就是函数跳转指令以及其内部实现的等价操作。

```asm
b ZZ1   <==>  PC = ZZ1
```

也就是说跳转指令等价于将指令中的地址赋值给`PC`寄存器。



#### **bl：** 

> `bl`、`blr` 指令在跳转的同时，会将 `lr` 寄存器的值设置为当前 pc+4byte。

跳转到某地址（有返回），先将下一指令地址（即函数返回地址）保存到寄存器 *lr* (x30)中，再进行跳转 ；一般用于不同方法直接的调用 ，如：

  ```asm
bl 0x100cfa754	; 先将下一指令地址（‘0x100cfa754’ 函数调用后的返回地址）保存到寄存器 ‘lr’ 中，然后再调用 ‘0x100cfa754’ 函数
  ```

跳转指令，将 bl 的下一个指令地址保存到 lr 寄存器，然后跳转到指定地址，因为将下一个指令保存到 lr 了，所以跳转完会回来执行下一个指令。用法如下：

```asm
mov x0, x1
bl 0x100221232 ;跳转到 0x100221232，执行完毕回来执行下一条指令
sub x0, x0, 0x10
```

函数调用的指令是`bl/blr `，其中`bl`指令的操作数是距离当前位置相对距离的偏移地址，`blr`指令的操作数则是寄存器，表明调用寄存器所指定的地址。因为bl指令中的操作数部分是函数的相对偏移地址，又因为arm64位系统的一条指令占用4个字节，根据指令的定义bl指令所能跳转的范围是距离当前位置±32MB的范围，所以如果要跳转到更远的地址则需要借助blr指令。 下面就是函数调用指令以及其内部实现的等价操作。

```asm
//如果YY1地址离调用指令的距离是在±32MB内则使用bl指令即可。
bl YY1 <==>  PC = YY1， LR = XX3

//如果YY1地址离调用指令的距离超过±32MB则使用blr指令执行间接调用。
ldr  x16,  YY1
blr  x16
```

也就是说执行一条函数调用指令等价于将指令中的地址赋值给`PC`寄存器，同时把函数的返回地址赋值给`LR`寄存器中去。

#### **blr：** 

> `bl`、`blr` 指令在跳转的同时，会将 `lr` 寄存器的值设置为当前 pc+4byte。

跳转到 `某寄存器` (的值)指向的地址（有返回），先将下一指令地址（即函数返回地址）保存到寄存器 *lr* (x30)中，再进行跳转 ；如：

  ```asm
blr x20	; 先将下一指令地址（‘x20’指向的函数调用后的返回地址）保存到寄存器 ‘lr’ 中，然后再调用 ‘x20’ 指向的函数
  ```

跳转指令，和 bl 类似，但可以使用动态地址，可以跳转到寄存器的值保存的地址，用法如下：

```asm
mov x8, 0x100221232
blr x8
```

#### **br：** 

跳转到某寄存器(的值)指向的地址（无返回）, 不会改变 *lr (x30)* 寄存器的值。

跳转指令，直接跳转到指定地址，跳转完不返回。有些类似在一个函数末尾调用了另外的函数。用法如下：

```asm
br x10 ;跳转到 x10 中保存的地址
```

#### **brk:** 
可以理解为跳转指令特殊的一种。

#### **ret：** 
子程序（函数调用）返回指令，返回地址已默认保存在寄存器 *lr* (x30) 中

函数返回的指令是` ret`， 下面就是函数返回指令以及其内部实现的等价操作。

```asm
 ret  <==>   PC = LR
```

也就是说执行一条`ret`指令等价于将`LR`寄存器中的值赋值给`PC`寄存器。

#### LDXR & STXR 

----

LDXR 即 LDR 的 Exclusive 版本，它的用法与 LDR 完全一致，区别在于它含有 Load-Exclusive 语义，即将读取的内存单元状态置为 Exclusive。

STXR 即 STR 的 Exclusive 版本，由于需要是否 Store 成功，他相比于 STR 多了一个 32 位寄存器的参数用于接收执行结果，用法为：

```asm
STXR  Ws, Xt, [Xn|SP{,#0}]
```

即尝试将 `Xt` 写入 `[Xn|SP{,#0}]`，如果写入成功则将 0 写入 Ws，否则将非 0 写入，它常常和 CBZ 指令搭配，如果写入失败则跳回到 LDXR，重新执行一遍 LDXR & STXR 操作，直至成功

#### LDAXR & STLXR

----

除了 Exclusive 语义外，LDXR & STXR 还有其 Acquire-Release 语义的 LDAXR & STLXR 版本，用于保证执行顺序。

对于单纯的 Atomic Add 操作，前者已经足够；如果涉及到类似于 [上一篇文章](https://juejin.im/post/5d9891abf265da5b926bc2b7) 提到的读写等待操作，则需要通过后者强保证不被乱序执行干扰。

#### CAS & CASA & CASAL

---

ARM 提供了多条指令直接完成 Compare and Swap 操作，其中 CAS 是最基础的版本，它的使用方法如下[4]：

```asm
CAS Xs, Xt, [Xn|SP{,#0}] ; 64-bit, no memory ordering
```

尝试将 `Xt` 与内存中的值进行交换，首先比较 `Xs` 是否等于内存中的 `[Xn|SP{,#0}]`，如果相等则将 `Xt` 写入内存，同时将内存中的值写回到 `Xs`，因此只要在 CAS 之后判断 `Xs` 是否等于 `Xt` 即可知道是否写入成功，如果写入失败则 `Xs` 的值应为原始值，即 `Xs` ≠ `Xt`，如果写入成功则内存中的值已被更新，即 `Xs` = `Xt`。

下面的例子采用 CAS 方式同样实现了原子加一操作：

```asm
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
```

**注意：为了在 iOS 系统上编译包含 CAS 指令的内容，需要给 .s 文件添加一个 Compile Flag: `-march=armv8.1-a`**[5]。

> 同样的，CAS 也有其含有 Acquire-Release 语义的版本，分别是含有 Acquire 语义的 CASA, 含有 Release 语义的 CASL，和同时包含 Acquire-Release 两种语义的 CASAL。

#### **b.ne、b.ge、、**



#### **b.lo、b.le、b.lt**

比较的结果存储在PSR(指令状态寄存器中)，接着通过b.le（less or equal）读取PSR



#### **push**

入栈操作，将相关数据加入栈中，并自动调整栈顶的位置，`sp` 的值会做应该的减法

```asm
push {x7, rl}
```



#### **pop**

与`push`相反，`pop` 是出栈操作

# 参考文章



- [iOS ABI Function Call Guide](https://developer.apple.com/library/archive/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARM64FunctionCallingConventions.html)
- ARM汇编可以参考官方文档[**armasm_user_guide**](https://link.jianshu.com/?t=http%3A%2F%2Finfocenter.arm.com%2Fhelp%2Ftopic%2Fcom.arm.doc.dui0801g%2FDUI0801G_armasm_user_guide.pdf)和[**ABI**](https://link.jianshu.com/?t=http%3A%2F%2Finfocenter.arm.com%2Fhelp%2Ftopic%2Fcom.arm.doc.den0024a%2FDEN0024A_v8_architecture_PG.pdf)
- [iOS开发同学的arm64汇编入门](https://blog.cnbluebox.com/blog/2017/07/24/arm64-start/)
- [iOS逆向之旅（基础篇） — 汇编（一）— 汇编基础](https://juejin.im/post/5bd197f5e51d457a2b7b248b)
- [[C in ASM(ARM64)\]第一章 一些实例](https://zhuanlan.zhihu.com/p/31168191)
- [Using the Stack in AArch64: Implementing Push and Pop](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/using-the-stack-in-aarch64-implementing-push-and-pop)
- [arm指令帮助文档](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0802a/BRK.html)
- Apple提供的[System Call Table](https://www.theiphonewiki.com/wiki/Kernel_Syscalls)
- ARM公司提供的[官方文档](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0802a/STUR_fpsimd.html)
- [iOS调试进阶](https://zhuanlan.zhihu.com/c_142064221)
- [iOS开发同学的arm64汇编入门](https://blog.cnbluebox.com/blog/2017/07/24/arm64-start/)
- [volatile 与内存屏障总结](https://zhuanlan.zhihu.com/p/43526907)