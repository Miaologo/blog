---
title: Objective-C Direct Methods
date: 2020-05-19 13:16:00
tags:
	- iOS
	- Objective-C
---

# Objective-C Direct Methods

[Objective-C Direct Methods](https://nshipster.com/direct/)

> Google翻译+手动校正

当新特性出现在Objective-C中时，很难让人兴奋。如今，任何此类改进都是为Swift互操作性服务的，而不是对语言本身的投资（请参见可空性*nullability*和轻量级泛型*lightweight*）。

因此，了解到这个最近合并到Clang的补丁是令人惊讶的，它为Objective-C方法添加了一个新的直接派发机制（direct dispatch mechanism）。

这一新语言功能的起源尚不清楚；我们最多只能找到一个Apple内部的Radar编号（2684889），除了它的相对年龄(据我们估计，大约在20世纪初)，它并不能告诉我们更多。幸运的是，该功能具有足够的文档和测试覆盖范围，可以很好地了解其工作原理。 （对实现者Pierre Habouzit，审查经理John McCall和其他LLVM贡献者表示敬意）。

本周在NSHipster上，我们借此机会回顾了Objective-C方法的调度，并试图了解这一新语言功能对未来代码库的潜在影响。

> **Direct Methods**最早可能会出现在Xcode 11.x上，但很可能会在WWDC 2020上宣布。

要了解直接方法的重要性，您需要了解一些有关Objective-C运行时的知识。 但是，在此之前，让我们从oop本身的起源开始我们的讨论：

## Object-Oriented Programming 面向对象编程

艾伦·凯（Alan Kay）在1960年代后期创造了“面向对象编程”一词。在阿黛尔·戈德堡（Adele Goldberg），丹·英加尔斯（Dan Ingalls）和他在Xerox PARC的其他同事的帮助下，凯伊（Kay）通过创建Smalltalk编程语言，在70年代将该思想付诸实践。

在此期间，Xerox PARC的研究人员还开发了Xerox Alto，这将成为苹果Macintosh和所有其他GUI计算机的灵感来源。

在1980年代，布拉德·考克斯（Brad Cox）和汤姆·洛夫（Tom Love）开始开发Objective-C的第一个版本，该语言试图采用Smalltalk的面向对象模式，并在C的坚实基础上实现它。通过90年代的一系列偶然事件，该语言将成为下一个，以及后来的苹果的官方语言。

对于我们这些在iPhone时代开始学习Objective-C的人来说，这门语言常常被看作是苹果公司的又一项专有技术——它是苹果公司“不在这里发明”（nih）文化的无数晦涩难懂的副产品之一。但是，Objective-C不仅是“面向对象的C”，它还是原始的面向对象的语言之一，与其他任何语言一样，都有很强的面向对象凭证。

现在，oop是什么意思？这是个好问题。上世纪90年代的炒作周期使该词几乎毫无意义。但是，就我们今天的目的而言，让我们集中讨论艾伦·凯（Alan Kay）在1998年写的一句话：

> 很抱歉，我很久以前就为该主题创造了“对象”一词，因为它使许多人把注意力集中在较次要的思想上。 其主要思想是“消息传递”[ […]> ——Alan Kay

## Dynamic Dispatch and the Objective-C Runtime 动态派发和Objective-C的运行时

在Objective-C中，程序由一组相互交互的对象组成，这些对象通过传递消息来相互作用，而这些消息又反过来调用方法或函数。这种消息传递行为用方括号语法表示:

```
[someObject aMethod:withAnArgument];
复制代码
```

当Objective-C代码被编译时，消息发送被转换成对一个名为`objc_msgSend`的函数的调用(字面意思是“将消息发送给某个带有参数的对象”)。

```
objc_msgSend(object, @selector(message), withAnArgument);
复制代码
```

- 第一个参数是接收方（实例方法的自身）
- 第二个参数是_cmd：选择器或方法的名称
- 任何方法参数都作为附加的函数参数传递

`objc_msgSend`负责确定响应此消息该调用哪个底层实现，这个过程称为方法派发。

在Objective-C中，每个类(Class)维护一个派发表来解析运行时发送的消息。 派发表中的每个条目都是一个方法(Method)，它将选择器(SEL)键接到对应的实现(IMP)，后者是指向C函数的指针。 当对象收到消息时，它查询其类的派发表。 如果可以找到选择器的实现，则调用关联的函数。 否则，对象将查询其超类的派发表。 这将继续沿继承链向上进行，直到找到匹配项或根类（NSObject）认为选择器无法识别。

> 这还不包括Objective-C如何让你替换方法实现以及在运行时动态创建新类。你所能做的绝对是疯狂的。

如果你认为所有这些间接听起来都需要大量工作…在某种程度上，你是对的!

如果您的代码中有一个热路径(一种频繁调用的昂贵方法)，那么您可以想象避免所有这些间接方法的好处。 为此，一些开发人员已使用C函数作为动态调度的一种方式。

## Direct Dispatch with a C Function 用C函数直接调用

正如我们在`objc_msgSend`中看到的，任何方法调用都可以通过将隐式self作为第一个参数传递来用等价的函数表示。

例如，考虑使用传统的动态派发方法对Objective-C类进行以下声明。

```
@interface MyClass: NSObject
- (void)dynamicMethod;
@end
复制代码
```

如果开发人员希望在MyClass上实现某些功能,但不想完成整个消息发送过程，那么他们可以声明一个静态C函数，该函数以MyClass的一个实例作为参数。

```
static void directFunction(MyClass *__unsafe_unretained object);
复制代码
```

以下是这些方法如何转化为呼叫站点:

```
MyClass *object = [[[MyClass] alloc] init];

// Dynamic Dispatch
[object dynamicMethod];

// Direct Dispatch
directFunction(object);
复制代码
```

## Direct Methods 直接方法

直接方法具有常规方法的外观，但是具有C函数的行为。 当直接方法被调用时，它直接调用它的底层实现，而不是通过`objc_msgSend`。

有了这个新的LLVM补丁，您现在可以注释掉Objective-C方法，从而有选择性地避免参与动态派发。

## objc_direct, @property(direct), and objc_direct_members

要使实例或类方法直接使用，可以使用`objc_direct` Clang属性对其进行标记。 同样，可以通过使用direct property属性声明Object-C属性的方式来使其直接使用。

```
@interface MyClass: NSObject
@property(nonatomic) BOOL dynamicProperty;
@property(nonatomic, direct) BOOL directProperty;

- (void)dynamicMethod;
- (void)directMethod __attribute__((objc_direct));
@end
复制代码
```

> 根据我们的计算，直接的添加使`@property`特性的总数达到16：
>
> - **getter** and **setter**
> - **readwrite** and **readonly**,
> - **atomic** and **nonatomic**
> - **weak**, **strong**, **copy**, **retain**, and **unsafe_unretained**
> - **nullable**, **nonnullable**, and **null_resettable**
> - **class**

当类别或类扩展的@interface使用`objc_direct_members`属性进行注释时，其中包含的所有方法和属性声明都被认为是直接的，除非该类以前声明过。

> 不能使用`objc_direct_members`属性来注释主类接口

```
__attribute__((objc_direct_members))
@interface MyClass ()
@property (nonatomic) BOOL directExtensionProperty;
- (void)directExtensionMethod;
@end
复制代码
```

用`objc_direct_members`注释`@implementation`也有类似的效果，导致未先前声明的成员被认为是直接成员，包括由属性合成产生的任何隐式方法。

```
__attribute__((objc_direct_members))
@implementation MyClass
- (BOOL)directProperty {…}
- (void)dynamicMethod {…}
- (void)directMethod {…}
- (void)directExtensionMethod {…}
- (void)directImplementationMethod {…}
@end
复制代码
```

> 动态方法不能在子类中被直接方法覆盖，而且直接方法根本不能被覆盖。 协议不能声明直接方法要求，而类也不能使用直接方法来实现协议要求。

将这些注释应用到我们之前的例子中，我们可以看到直接方法和动态方法在调用站点上是如何难以区分的:

```
MyClass *object = [[[MyClass] alloc] init];

// Dynamic Dispatch
[object dynamicMethod];

// Direct Dispatch
[object directMethod];
复制代码
```

对于我们中注重性能的开发人员而言，直接方法似乎是一个非常有用的特性。但这里有个转折:

在大多数情况下，使用直接方法可能不会有明显的性能优势。

事实证明，`objc_msgSend`的速度快得惊人。 由于积极的缓存，广泛的底层优化以及现代处理器固有的性能特征，`objc_msgSend`的开销非常低。

iPhone硬件被合理地描述为资源受限的环境的时代已经过去很久了。因此，除非苹果正在准备一个新的嵌入式平台(AR眼镜，有人知道吗?)，否则我们对苹果在2019年实现Objective-C直接方法的最合理的解释是性能以外的原因。

> Mike Ash是互联网上最重要的`objc_msgSend`专家。 多年来，他的文章为Cupertino之外的Objective-C运行时提供了最深刻，最完整的理解。 对于那些好奇的人来说，“剖析ARM64上的`objc_msgSend`”是一个很好的开始。

## Hidden Motives 隐藏动机

当Objective-C方法被标记为直接时，它的实现就隐藏了可见性。 也就是说，直接方法只能在相同的模块中调用(或学究式的将之称为链接单元)。 它甚至不会出现在Objective-C运行时中。

隐藏可见性有两个直接的优势:

- 较小的二进制文件大小
- 没有外部调用

没有外部可见性，也没有从Objective-C运行时动态调用它们的方法，直接方法实际上是私有方法。

> 如果您想参与直接派发，但仍希望使您的API可以从外部访问，则可以将其封装在一个C函数中。
>
> ```
> static inline void performDirectMethod(MyClass *__unsafe_unretained object) {
>     [object directMethod];
> }
> 复制代码
> ```

虽然隐藏的可见性可以被Apple用来防止方法交换（Swizzing）和私有API的使用，但这似乎不是主要的动机。

实施此功能的Pierre认为，此优化的主要好处是减少了代码大小。 据报道，未使用的Objective-C元数据在编译后的二进制代码中多5 - 10%的比例。

您可以想象，从现在开始直到明年的开发者大会，一些工程师可以遍历每个SDK框架，使用`objc_direct`注释私有方法，并使用`objc_direct_members`注释私有类，这是一种逐步瘦身SDK的轻量级方法。

如果这是真的，那么也许我们已经开始怀疑Objective-C的新特性了。当他们不为Swift服务时，他们为Apple服务。尽管Objective-C在编程史和苹果公司的发展史上占有重要的地位，但很难不将它视为历史。