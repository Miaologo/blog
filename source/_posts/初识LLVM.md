---
title: 初识LLVM
date: 2020-05-18 16:37:19
tags:
	- iOS
	- LLVM
---

学 LLVM Clang 能做什么？

参考 http://clang.llvm.org/docs/Tooling.html

- `语法树分析`、`语言转换`。libclang、libTooling。

语言转换就是比如把 oc代码 转成 Swift代码，或者把 oc代码 转成 js代码。

- Clang 插件开发

可以用来做代码检查（命名规范、代码规范），安装包瘦身等等各种事情。
编写插件可以参考官方教程
http://clang.llvm.org/docs/ClangPlugins.html
http://clang.llvm.org/docs/ExternalClangExamples.html
http://clang.llvm.org/docs/RAVFrontendAction.html

滴滴的动态化方案 `DynamicCocoa` 就是使用了一个将 OC 源码转 JS 的插件来进行代码的转换. 
鹅厂的动态化方案 `OCS` 是直接在端内写了个编译器.
可以搜索 《DynamicCocoa：滴滴 iOS 动态化方案的诞生与起航》 和 《OCS——史上最疯狂的iOS动态化方案》查看介绍。

- Pass 开发。

可以用来做`代码优化`、`代码混淆`等。
官方 Pass 教程 https://llvm.org/docs/WritingAnLLVMPass.html

- 开发新的编程语言

[用LLVM开发新语言](https://llvm-tutorial-cn.readthedocs.io/en/latest/index.html)
原文：http://llvm.org/docs/tutorial/index.html
译文：https://llvm-tutorial-cn.readthedocs.io/en/latest/index.html

[Kaileidoscope: LLVM Tutorial Chinese version](https://kaleidoscope-llvm-tutorial-zh-cn.readthedocs.io/zh_CN/latest/index.html)
中文：https://kaleidoscope-llvm-tutorial-zh-cn.readthedocs.io/zh_CN/latest/index.html

还是说说 LLVM 到底是什么吧，LLVM的项目是一个模块化和可重复使用的编译器和工具技术的集合.LLVM 曾经是一个缩写词，现在不是，它就是这个项目的名称。
Clang 是 LLVM 的子项目，是 C，C++ 和 Objective-C 编译器。

再来一个更容易理解的说法，iOS 开发中 Objective-C 是 Clang / LLVM 来编译的。（swift 是 Swift / LLVM）

------

传统的编译器最流行的设计是分三段，分别是前端(Frontend)，优化器(Optimizer)和后端(Backend).

![img](https://images.xiaozhuanlan.com/photo/2018/84b13853e4a6b368bb4f487455629bc5.png)

`前端(Frontend)`负责解析源代码，检查语法错误，并将其翻译为抽象的语法树（Abstract Syntax Tree）.也就是词法分析，语法分析，语义分析和生成中间代码。

`优化器(Optimizer)`负责进行各种转换尝试改进代码的运行时间.release 包比 debug 包体积小运行快，其中的一个原因就是优化器起作用。比如重复计算的消除。

`后端(Backend)`用来生成实际的机器码.

这种设计有很多优点，但实际这一结构却从来没有被完美实现过。GCC 做的比较好，实现了很多前端和后端，支持了很多语言。但是有个缺陷，那就是他们是一个完整的可执行文件，没有把前端和后端分的太开，所以GCC 为了支持一门新的语言，或者支持一种新的平台，就变得比较困难。

LLVM 就解决了上面的问题，因为它被设计为一组库,而不是一个编译器，如下图
![img](https://images.xiaozhuanlan.com/photo/2018/a4c50acc4f4babc085359c307c5734a3.png)

从上图中我们发现LLVM与GCC在三段式架构上并没有本质区别，也是分为前端、优化器和后端。但是，其设计最重要的方面是不同的前端、后端使用同一的中间代码 LLVM Intermediate Representation (LLVM IR) . 也就是图中的 LLVM Optimizer

这种设计的好处就是，如果需要支持一种新的编程语言，那么只需要实现一个新的前端。
如果需要支持一种新的硬件设备，那么只需要实现一个新的后端。优化阶段是一个通用的阶段，它针对的是同一的LLVM IR,不论是支持新的编程语言，还是支持新的硬件设备，都不需要对优化阶段做修改。

Clang 就是基于LLVM架构的C/C++/Objective-C编译器`前端`,如下图
![img](https://images.xiaozhuanlan.com/photo/2018/36c1d33576e23405162e03df2b388845.png)
Clang 主要处理一些和具体机器无关的针对语言的分析操作.编译器的优化器部分和后端部分是LLVM后端，也可以直接叫 LLVM（狭义的LLVM），广义上LLVM 就是整个LLVM架构。

![img](https://images.xiaozhuanlan.com/photo/2018/debbaa315c7647803c6b58875ae668f3.png)

看上面的图，左边是编程语言，最终是机器码，在我们Xcode 中编写的代码首先会经过 clang 这个编译器前端，他会生成中间代码（IR），这个中间代码又经过一系列的优化，这些优化就是 `Pass`，如果咱要编写中间代码优化代码的话，那就是编写`Pass`,最后就是生成机器码.
通过图也可以看出，`Pass` 是 LLVM 系统转化和优化的工作的一个节点，每个节点做一些工作，这些工作加起来就构成了 LLVM 整个系统的优化和转化。

在编译一个源文件时，编译器的处理过程分为几个阶段。
用命令行查看一下OC源文件的编译过程

```
$ clang -ccc-print-phases main.m

0: input, "main.m", objective-c

1: preprocessor, {0}, objective-c-cpp-output

2: compiler, {1}, ir

3: backend, {2}, assembler

4: assembler, {3}, object

5: linker, {4}, image

6: bind-arch, "x86_64", {5}, image
```

一共是7个阶段，
第0个阶段找到源代码，读入文件。
第1个阶段 preprocessor，预处理器，就是把头导入，宏定义给展开，包括 `#define`、 `#include`、 `#import`、 `#indef`、 `#pragma`。
第2阶段就是 compiler，编译器编译成 ir 中间代码。
第3阶段就是交给后端，来生成汇编代码（assembler）。
第4阶段是将汇编代码转换为目标对象文件
第5阶段是链接器,将多个目标对象文件合并为一个可执行文件 (或者一个动态库) 。
最后一阶段 生成可执行文件 :Mach-O

**预处理（Preprocess）**
先看一下预处理阶段干了什么事情:
创建一个`main.m` 文件

```
#import <Foundation/Foundation.h>

#define TEN 10

int main(int argc, char * argv[]) {

    int a = TEN;

    int b = 8;

    NSLog(@"num = %d",a+b);

    return 0;

}
```

执行下面命令

```
$ clang -E main.m
```

我们会发现导入了很多的头文件内容,

```
...

# 181 "/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h" 2 3

# 1 "/System/Library/Frameworks/Foundation.framework/Headers/FoundationLegacySwiftCompatibility.h" 1 3

# 185 "/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h" 2 3

# 13 "main.m" 2



int main(int argc, char * argv[]) {

    int a = 10;

    int b = 8;

    NSLog(@"num = %d",a+b);

    return 0;

}
```

可以看到上面的预处理已经把宏替换了，并且导入了头文件。 main 函数上面的一大堆内容就是 Foundation.h 文件中的内容去替换 `#import ` 这行代码,如果 Foundation.h 中也使用了类似的宏引入，则会按照同样的处理方式用各个宏对应的真正代码进行逐级替代。

这也就是为什么人们主张头文件最好尽量少的去引入其他的类或库，因为引入的东西越多，编译器需要做的处理就越多。

在 Xcode 中，可以通过这样的方式查看任意文件的预处理结果：`Product -> Perform Action -> Preprocess`
当然，这个的结果和咱自己执行的命令看到的结果稍微有点不同，因为他带了很多参数。通过下图的方式查看
![img](https://images.xiaozhuanlan.com/photo/2018/97e5e98ed4b2bffaafd46c6ed3b54bfe.png)

涉及到导入头文件这块的不同， 是因为命令行会加上 `-fmodules`参数.
用咱iOS熟悉的表达方式是 `@import` 优于 `#import` 优于 `#include`。
如果在代码中写了 `#include ` 导入语句，用 Xcode 预编译后，会自动变成 `@import Darwin.C.stdio`. 对 modules 具体描述以及命令可以查看 Clang 8 documentation 的 Modules 部分。
http://clang.llvm.org/docs/Modules.html

**词法分析 (Lexical Analysis)**
在预处理之后，就要进行词法分析了，将预处理过的代码转化成一个个 Token，比如左括号、右括号、等于、字符串等等。

```
clang -fmodules -fsyntax-only -Xclang -dump-tokens main.m
```

对clang 命令参数的简单说明
`-fmodules` ：允许modules的语言特性
`-fsyntax-only` : 防止编译器生成代码,只是语法级别的说明和修改
`-Xclang`: 向 clang 编译器传递参数
`-dump-tokens`: 运行预处理器,拆分内部代码段为各种`token`

下面是一段简单的 Objective-C hello word 程序

```
int main() {

  NSLog(@"hello, %@", @"world");

  return 0;

}
```

执行词法分析命令后输出

```c
$ clang -fmodules -fsyntax-only -Xclang -dump-tokens main.m

int 'int'     [StartOfLine]  Loc=<main.m:34:1>

identifier 'main'     [LeadingSpace] Loc=<main.m:34:5>

l_paren '('        Loc=<main.m:34:9>

r_paren ')'        Loc=<main.m:34:10>

l_brace '{'     [LeadingSpace] Loc=<main.m:34:12>

identifier 'NSLog'     [StartOfLine] [LeadingSpace]   Loc=<main.m:35:5>

l_paren '('        Loc=<main.m:35:10>

at '@'        Loc=<main.m:35:11>

string_literal '"hello, %@"'        Loc=<main.m:35:12>

comma ','        Loc=<main.m:35:23>

at '@'     [LeadingSpace] Loc=<main.m:35:25>

string_literal '"world"'        Loc=<main.m:35:26>

r_paren ')'        Loc=<main.m:35:33>

semi ';'        Loc=<main.m:35:34>

return 'return'     [StartOfLine] [LeadingSpace]   Loc=<main.m:36:5>

numeric_constant '0'     [LeadingSpace] Loc=<main.m:36:12>

semi ';'        Loc=<main.m:36:13>

r_brace '}'     [StartOfLine]  Loc=<main.m:37:1>

eof ''        Loc=<main.m:37:2>
```

可以看到，`int` 是一个Token，`（` 是一个Token,就连`,` `;`都是Token，每个 Token 都包含了对应的源码内容和其在源码中的位置。
这样一来，如果编译过程中遇到什么问题，clang 能够在源码中指出出错的具体位置.

**语法分析 (Semantic Analysis)**
根据当前语言的语法，验证语法是否正确，并将之前生成的 Token 解析成一棵抽象语法树(abstract syntax tree -- AST)

```
clang -fmodules -fsyntax-only -Xclang -ast-dump main.m
```

`-ast-dump` : 构建抽象语法树AST,然后对其进行拆解和调试

例子

```
#import <Foundation/Foundation.h>



@interface World : NSObject

- (void)hello;

@end



@implementation World

- (void)hello {

    NSLog(@"hello, world");

}

@end



int main(int argc, char * argv[]) {

    World* world = [World new];

    [world hello];

}
```

执行语法分析命令后输出

```
$ clang -fmodules -fsyntax-only -Xclang -ast-dump main.m

...



|-ImportDecl 0x7f9f151fa818 <main.m:34:1> col:1 implicit Foundation

|-ObjCInterfaceDecl 0x7f9f151fc1e0 <line:36:1, line:38:2> line:36:12 World

| |-super ObjCInterface 0x7f9f151fa908 'NSObject'

| |-ObjCImplementation 0x7f9f151fc458 'World'

| `-ObjCMethodDecl 0x7f9f151fc3d0 <line:37:1, col:14> col:1 - hello 'void'

|-ObjCImplementationDecl 0x7f9f151fc458 <line:40:1, line:44:1> line:40:17 World

| |-ObjCInterface 0x7f9f151fc1e0 'World'

| `-ObjCMethodDecl 0x7f9f151fc4f0 <line:41:1, line:43:1> line:41:1 - hello 'void'

|   |-ImplicitParamDecl 0x7f9f14824c38 <<invalid sloc>> <invalid sloc> implicit self 'World *'

|   |-ImplicitParamDecl 0x7f9f14824cc0 <<invalid sloc>> <invalid sloc> implicit _cmd 'SEL':'SEL *'

|   `-CompoundStmt 0x7f9f14824e98 <col:15, line:43:1>

|     `-CallExpr 0x7f9f14824e50 <line:42:5, col:26> 'void'

|       |-ImplicitCastExpr 0x7f9f14824e38 <col:5> 'void (*)(id, ...)' <FunctionToPointerDecay>

|       | `-DeclRefExpr 0x7f9f14824d20 <col:5> 'void (id, ...)' Function 0x7f9f151fc588 'NSLog' 'void (id, ...)'

|       `-ImplicitCastExpr 0x7f9f14824e80 <col:11, col:12> 'id':'id' <BitCast>

|         `-ObjCStringLiteral 0x7f9f14824dc0 <col:11, col:12> 'NSString *'

|           `-StringLiteral 0x7f9f14824d88 <col:12> 'char [13]' lvalue "hello, world"

|-FunctionDecl 0x7f9f1582dab8 <line:45:1, line:49:1> line:45:5 main 'int (int, char **)'

| |-ParmVarDecl 0x7f9f1582d858 <col:10, col:14> col:14 argc 'int'

| |-ParmVarDecl 0x7f9f1582d970 <col:20, col:32> col:27 argv 'char **':'char **'

| `-CompoundStmt 0x7f9f14165100 <col:35, line:49:1>

|   |-DeclStmt 0x7f9f1582dc70 <line:47:5, col:31>

|   | `-VarDecl 0x7f9f1582dbd0 <col:5, col:30> col:12 used world 'World *' cinit

|   |   `-ObjCMessageExpr 0x7f9f1582dc40 <col:20, col:30> 'World *' selector=new class='World'

|   `-ObjCMessageExpr 0x7f9f1582dce0 <line:48:5, col:17> 'void' selector=hello

|     `-ImplicitCastExpr 0x7f9f1582dcc8 <col:6> 'World *' <LValueToRValue>

|       `-DeclRefExpr 0x7f9f1582dc88 <col:6> 'World *' lvalue Var 0x7f9f1582dbd0 'world' 'World *'

`-<undeserialized declarations>
```

`main` 函数在 `FunctionDecl` 节点
`ParmVarDecl` 节点对应着 main 函数的参数

在抽象语法树中的每个节点都标注了其对应源码中的位置.
可以查看 Clang 8 documentation 的 Clang AST简介
http://clang.llvm.org/docs/IntroductionToTheClangAST.html

一旦编译器把源码生成了抽象语法树，编译器可以对这棵树做静态分析.如果你把 clang 的代码仓库 clone 到本地，然后进入目录 `lib/StaticAnalyzer/Checkers`，你会看到所有静态检查内容.

**IR 代码生成 (CodeGen)**
完成这些步骤后就可以开始IR中间代码的生成了.
IR 是 Frontend(前端) 的输出，也是 Backerend（后端） 的输入，桥接前后端。

LLVM IR 有三种表示格式(本质上是一样的)

- 文本格式，便于阅读，类似于汇编语言，扩展名 `.ll` 。`$ clang -S -fobjc-arc -emit-llvm main.m`
- bitcode 格式，二进制格式,便于存储，扩展名 `.bc` 。 `$ clang -c -fobjc-arc -emit-llvm main.m`
- 内存格式.

对clang 命令参数的简单说明
`-fobjc-arc` : 为OC对象生成retain和release的调用
`-emit-llvm` :使用LLVM描述汇编和对象文件
`-S` :只运行预处理和编译步骤
`-c` :只运行预处理,编译和汇编步骤

看一个 IR 的简单例子，
main.m 文件

```c
void test(int a, int b){

    int c = a + b -4;

}
```

执行命令 `clang -S -fobjc-arc -emit-llvm main.m -o main.ll`