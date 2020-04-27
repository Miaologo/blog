---
title: 深入理解Link Map File
date: 2020-04-26 23:13:17
tags:
---

#### Link Map File初识

我们编写的源码需要经过编译、链接，最终生成一个可执行文件。在编译阶段，每个类会生成对应的.o文件（目标文件）。在链接阶段，会把.o文件和动态库链接在一起。Link Map File就是这样一个记录链接相关信息的纯文本文件，里面记录了可执行文件的路径、CPU架构、目标文件、符号等信息。

#### 为什么要理解Link Map File

理解Link Map File，可以帮助我们：

- 理解链接过程
- 理解Crash时，通过Symbols还原出源码的机制
- 理解内存分段、分区
- 分析可执行文件中哪个类或库占用比较大，进行安装包瘦身

#### Link Map File配置

点击工程，选择Build Setting选项，搜索map，可以看到如下界面。将Write Link Map File设置为Yes后，Build结束后，会在默认路径下生成一个Link Map File文件，该文件是txt格式的。点击Path to Link Map File，可以设置Debug或Release模式下的生成路径。

#### Path & Arch



```bash
# Path: /xxxxxx/Library/Developer/Xcode/DerivedData/xxxxxx.app-hjsjojqpqxstlzceepeqbvxqzcrh/Build/Intermediates.noindex/xxxx/xxxxxx.app/xxxxxx
# Arch: x86_64
```

Path是可执行文件的路径，Arch是架构类型。

#### Object files



```csharp
# Object files:
[  0] linker synthesized
[  1] /Users/shanggaolin/Library/Developer/Xcode/DerivedData/NewMissFresh-hjsjojqpqxstlzceepeqbvxqzcrh/Build/Intermediates.noindex/NewMissFresh.build/Debug-iphonesimulator/NewMissFresh.build/NewMissFresh.app-Simulated.xcent
[  2] /Users/shanggaolin/Library/Developer/Xcode/DerivedData/NewMissFresh-hjsjojqpqxstlzceepeqbvxqzcrh/Build/Intermediates.noindex/NewMissFresh.build/Debug-iphonesimulator/NewMissFresh.build/Objects-normal/x86_64/UIButton+SSEdgeInsets.o
[  3] /Users/shanggaolin/Library/Developer/Xcode/DerivedData/NewMissFresh-hjsjojqpqxstlzceepeqbvxqzcrh/Build/Intermediates.noindex/NewMissFresh.build/Debug-iphonesimulator/NewMissFresh.build/Objects-normal/x86_64/MFRankListTableViewDelegate.o
```

在编译成目标文件后，通过链接器进行链接，最终合成可执行文件。这里展示的信息是链接时用到的文件，包括.o文件和dylib库。第一列的序号是类的编号，通过该编号可以对应到具体的类。

在后面的Symbols部分，我们会用到类的编号。

#### Section区



```objectivec
# Sections:
# Address   Size        Segment Section
0x100002780 0x0129617D  __TEXT  __text
0x1012988FE 0x000015E4  __TEXT  __stubs
0x101299EE4 0x0000207C  __TEXT  __stub_helper
0x10129BF60 0x0002BE10  __TEXT  __const
0x1012C7D70 0x00097A6D  __TEXT  __objc_methname
0x10135F7DD 0x00010CD5  __TEXT  __objc_classname
0x1013704B2 0x00015F47  __TEXT  __objc_methtype
0x101386400 0x000B6CB2  __TEXT  __cstring
0x10143D0B2 0x00007D60  __TEXT  __ustring
0x101444E14 0x00054EA8  __TEXT  __gcc_except_tab
0x101499CBC 0x000002B0  __TEXT  __entitlements
0x101499F6C 0x00014CA8  __TEXT  __unwind_info
0x1014AEC18 0x0003B3E8  __TEXT  __eh_frame

0x1014EA000 0x00000010  __DATA  __nl_symbol_ptr
0x1014EA010 0x00000BD8  __DATA  __got
0x1014EABE8 0x00001D30  __DATA  __la_symbol_ptr
0x1014EC918 0x00000028  __DATA  __mod_init_func
0x1014EC940 0x00073BB0  __DATA  __const
0x1015604F0 0x00062700  __DATA  __cfstring
0x1015C2BF0 0x00004BD0  __DATA  __objc_classlist
0x1015C77C0 0x00000058  __DATA  __objc_nlclslist
0x1015C7818 0x00000640  __DATA  __objc_catlist
0x1015C7E58 0x00000088  __DATA  __objc_nlcatlist
0x1015C7EE0 0x00001018  __DATA  __objc_protolist
0x1015C8EF8 0x00000008  __DATA  __objc_imageinfo
0x1015C8F00 0x001EE0A0  __DATA  __objc_const
0x1017B6FA0 0x00024A08  __DATA  __objc_selrefs
0x1017DB9A8 0x000000B8  __DATA  __objc_protorefs
0x1017DBA60 0x00004AD8  __DATA  __objc_classrefs
0x1017E0538 0x00002F08  __DATA  __objc_superrefs
0x1017E3440 0x00015A38  __DATA  __objc_ivar
0x1017F8E78 0x0002F670  __DATA  __objc_data
0x1018284F0 0x00024D18  __DATA  __data
0x10184D210 0x00006C74  __DATA  __bss
0x101853E90 0x00000E90  __DATA  __common
```

第一列是起始位置，第二列是Section占用内存大小，第三列是Segment类型，第四列是Section类型。
 为了理解上面的信息，我们需要先补充一点Mach-O知识。
 Mach-O 文件中的虚拟地址最终会映射到物理地址上。这些地址被分成不同的Segement： __TEXT段、__DATA段 和 __LINKEDIT段。
 （1）__TEXT 包含 Mach header，被执行的代码和只读常量（如C 字符串），只读可执行（r-x）。
 （2）__DATA 包含全局变量，静态变量等，可读写（rw-）。
 （3）__LINKEDIT 包含了加载程序的『元数据』，比如函数的名称和地址，只读（r–）。

Segement划分成了不同的Section，不同的Section存储着不同的信息，下面是一些常用的Section的介绍。



```objectivec
//  __TEXT段中的一些Section
1. __text: 代码节，存放机器编译后的代码
2. __stubs: 用于辅助做动态链接代码（dyld）.
3. __stub_helper:用于辅助做动态链接（dyld）.
4. __objc_methname:objc的方法名称
5. __cstring:代码运行中包含的字符串常量,比如代码中定义`#define kGeTuiPushAESKey        @"DWE2#@e2!"`,那DWE2#@e2!会存在这个区里。
6. __objc_classname:objc类名
7. __objc_methtype:objc方法类型
8. __ustring:
9. __gcc_except_tab:
10. __const:存储const修饰的常量
11. __dof_RACSignal:
12. __dof_RACCompou:
13. __unwind_info:

// __DATA段中的一些Section
1. __got:存储引用符号的实际地址，类似于动态符号表
2. __la_symbol_ptr:lazy symbol pointers。懒加载的函数指针地址。和__stubs和stub_helper配合使用。具体原理暂留。
3. __mod_init_func:模块初始化的方法。
4. __const:存储constant常量的数据。比如使用extern导出的const修饰的常量。
5. __cfstring:使用Core Foundation字符串
6. __objc_classlist:objc类列表,保存类信息，映射了__objc_data的地址
7. __objc_nlclslist:Objective-C 的 +load 函数列表，比 __mod_init_func 更早执行。
8. __objc_catlist: categories
9. __objc_nlcatlist:Objective-C 的categories的 +load函数列表。
10. __objc_protolist:objc协议列表
11. __objc_imageinfo:objc镜像信息
12. __objc_const:objc常量。保存objc_classdata结构体数据。用于映射类相关数据的地址，比如类名，方法名等。
13. __objc_selrefs:引用到的objc方法
14. __objc_protorefs:引用到的objc协议
15. __objc_classrefs:引用到的objc类
16. __objc_superrefs:objc超类引用
17. __objc_ivar:objc ivar指针,存储属性。
18. __objc_data:objc的数据。用于保存类需要的数据。最主要的内容是映射__objc_const地址，用于找到类的相关数据。
19. __data:暂时没理解，从日志看存放了协议和一些固定了地址（已经初始化）的静态量。
20. __bss:存储未初始化的静态量。比如：`static NSThread *_networkRequestThread = nil;`其中这里面的size表示应用运行占用的内存，不是实际的占用空间。所以计算大小的时候应该去掉这部分数据。
21. __common:存储导出的全局的数据。类似于static，但是没有用static修饰。比如KSCrash里面`NSDictionary* g_registerOrders;`, g_registerOrders就存储在__common里面
```

虚拟内存最终会映射到物理内存，通过上面的介绍，我们就可以知道代码和数据在内存中是如何存储的。

#### Symbols



```csharp
<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

# Symbols:
// __text代码区
# Address   Size        File  Name
0x100002780 0x00000450  [  2] -[UIButton(SSEdgeInsets) setImageUpTitleDownWithSpacing:]
0x100002BD0 0x00000070  [  2] _UIEdgeInsetsMake
0x100002C40 0x000004B0  [  2] -[UIButton(SSEdgeInsets) setImageRightTitleLeftWithSpacing:]
0x1000030F0 0x000001B0  [  2] -[UIButton(SSEdgeInsets) setDefaultImageTitleStyleWithSpacing:]
0x1000032A0 0x000006F0  [  2] -[UIButton(SSEdgeInsets) setEdgeInsetsWithType:marginType:margin:]
0x100003990 0x000000B0  [  3] -[MFRankListTableViewDelegate removeAllCellModels]
0x100003A40 0x000002A0  [  3] -[MFRankListTableViewDelegate appendCellModels:]
0x100003CE0 0x00000050  [  3] -[MFRankListTableViewDelegate numberOfSectionsInTableView:]
0x100003D30 0x000000C0  [  3] -[MFRankListTableViewDelegate tableView:numberOfRowsInSection:]

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

// __objc_methname方法名区
0x1012C7D70 0x0000000F  [  2] literal string: imageForState:
0x1012C7D7F 0x00000005  [  2] literal string: size
0x1012C7D84 0x00000014  [  2] literal string: setTitleEdgeInsets:
0x1012C7D98 0x0000000F  [  2] literal string: titleForState:
0x1012C7DA7 0x00000007  [  2] literal string: length
0x1012C7DAE 0x0000000B  [  2] literal string: titleLabel
0x1012C7DB9 0x00000005  [  2] literal string: font
0x1012C7DBE 0x00000025  [  2] literal string: dictionaryWithObjects:forKeys:count:
0x1012C7DE3 0x00000014  [  2] literal string: sizeWithAttributes:
0x1012C7DF7 0x00000014  [  2] literal string: setImageEdgeInsets:
0x1012C7E0B 0x00000019  [  2] literal string: attributedTitleForState:
0x1012C7E24 0x00000006  [  2] literal string: frame
0x1012C7E2A 0x00000020  [  2] literal string: setImageUpTitleDownWithSpacing:
0x1012C7E4A 0x00000023  [  2] literal string: setImageRightTitleLeftWithSpacing:
0x1012C7E6D 0x00000026  [  2] literal string: setDefaultImageTitleStyleWithSpacing:
0x1012C7E93 0x00000029  [  2] literal string: setEdgeInsetsWithType:marginType:margin:
0x1012C7EBC 0x0000000E  [  3] literal string: setIsLoading:

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

// __objc_classlist类列表区
0x1015C2BF0 0x00000008  [  3] anon
0x1015C2BF8 0x00000008  [  4] anon
0x1015C2C00 0x00000008  [  5] anon
0x1015C2C08 0x00000008  [  6] anon
0x1015C2C10 0x00000008  [  7] anon
```

##### 1、__text代码区

通过地址0x100002780，可以知道它位于__TEXT段的__text区，这段区域存储着代码，通过符号表，根据地址可以对应出源代码，如`-[UIButton(SSEdgeInsets) setImageUpTitleDownWithSpacing:]`。通过第三列的类编号，可以知道该代码属于`UIButton+SSEdgeInsets`分类。

##### 2、__objc_methname方法名区

通过地址0x1012C7D70，可以知道它位于__TEXT段的__objc_methname区，这段区域存储着方法名，通过符号表，根据地址可以对应出具体的方法名，如`imageForState`。
 由上面的信息，可以看出方法名越长，最终占用的内存也越大。

##### 3、__objc_classlist类列表区

__objc_classlist区的size值都是8，区域里存储的值都是一个指针，指向了类的虚拟地址（__objc_data区中地址）。
 objc_class的数据结构为：



```cpp
typedef struct objc_class {
        unsigned long long isa;
        unsigned long long wuperclass;
        unsigned long long cache;
        unsigned long long vtable;
        unsigned long long data;
        unsigned long long reserved1;
        unsigned long long reserved2;
        unsigned long long reserved3;
}objc_class;
```

上面的data字段保存了_objc_const区对应的数据地址（objc_classdata地址）。
 objc_classdata数据结构为:



```cpp
typedef struct objc_classdata {
    long long flags;
    long long instanceStart;
    long long instanceSize;
    long long reserved;
    unsigned long long ivarlayout;
    unsigned long long name;
    unsigned long long baseMethod;
    unsigned long long baseProtocol;
    unsigned long long ivars;
    unsigned long long weakIvarLayout;
    unsigned long long baseProperties;
}
```

objc_classdata结构体中保存了类名，方法名，协议名，ivar指针和属性对应的地址。通过地址，可以找到对应的段和分区，进而找到对应类的各种信息。

#### 查找未使用的类和方法

__objc_classrefs是引用到的类，_objc_classname是所有类名，通过分析两者之间的差别，就可以知道哪些类没有用到。
 同理，分析__objc_selrefs和_objc_methname的差别，也可知道哪些方法没用到。
 由于OC是动态语言，中间可能会出现判断失误的情况。

#### 分析大文件

在Symbols部分，我们可以把类编号相同的size加起来，算出每个类或库占用的大小。在Object files部分根据类的编号可以查出对应的类。分析的结果对App安装包瘦身会有一些帮助。github上有很多实现了这样功能的开源工具，实现原理很简单，如有需要可自行查找。