---
title: iOS基础篇-Mach-O
date: 2020-06-12 19:58:22
tags:
	- iOS
	- Mach-O
---

## 可执行文件

进程是特殊文件在内存中加载得到的结果。那这种文件的格式必须是系统内核可以理解的，系统内核才能正确解析。

不同操作系统的可执行文件格式不同：

|           可执行格式           |               魔数                | 用途                                                         |
| :----------------------------: | :-------------------------------: | :----------------------------------------------------------- |
|           PE32/PE32+           |                MZ                 | Windows的可执行文件                                          |
|              ELF               |              \x7FELF              | Linux和大部分UNIX的可执行文件和库文件                        |
|              脚本              |                #!                 | 主要用于shell脚本，也有一些解释器脚本使用这个格式。这是一种特殊的二进制文件格式，#! 后面指向真正的可执行文件（比如python），而脚本其它内容，都被当做输入传递给这个命令。 |
| 通用二进制格式（胖二进制格式） |         0xcafebabe(小端)          | 包含多种架构支持的Mach-O格式，iOS和OS X支持的格式            |
|             Mach-O             | 0xfeedface(32位) 0xfeedfacf(64位) | iOS和OS x支持的格式                                          |

系统内核将文件读入内存，然后寻找文件的头签名（魔数magic），根据magic就可以判断二进制文件的格式。

mach-o/loader.h 里定义了四个魔数标识。

```
#define MH_MAGIC    0xfeedface
#define MH_CIGAM    NXSwapInt(MH_MAGIC)
#define MH_MAGIC_64 0xfeedfacf
#define MH_CIGAM_64 NXSwapInt(MH_MAGIC_64)
```

以上四个魔数标识是 Mach-O 文件。

其实PE/ELF/Mach-O这三种可执行文件格式都是COFF（Common file format）格式的变种。COFF的主要贡献是目标文件里面引入了“段”的机制，不同的目标文件可以拥有不同数量和不同类型的“段”。

## Mach-O Universal 文件

[FAT 二进制](https://en.wikipedia.org/wiki/Fat_binary)文件，将多种架构的 Mach-O 文件合并而成。它通过 Fat Header 来记录不同架构在文件中的偏移量，Fat Header 占一页的空间。

按分页来存储这些 segement 和 header 会浪费空间，但这有利于虚拟内存的实现。

### 为什么有通用二进制文件

为什么有了Mach-O格式了，苹果还搞通用二进制格式？因为不同CPU平台支持的指令不同，比如arm64和x86，那我们是不是可以把arm64和x86对应的Mach-O格式打包在一起，然后系统根据自己的CPU平台，选择合适的Mach-O。通用二进制格式就是多种架构的Mach-O文件“打包”在一起，所以通用二进制格式，更多被叫做胖二进制格式。

### 通用二进制文件格式

通用二进制格式定义在<mach-o/fat.h>中。

```objective-c
#define FAT_MAGIC	0xcafebabe
#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */

struct fat_header {
	uint32_t	magic;		/* FAT_MAGIC or FAT_MAGIC_64 */
	uint32_t	nfat_arch;	/* number of structs that follow */
};

struct fat_arch {
	cpu_type_t	cputype;	/* cpu specifier (int) */
	cpu_subtype_t	cpusubtype;	/* machine specifier (int) */
	uint32_t	offset;		/* file offset to this object file */
	uint32_t	size;		/* size of this object file */
	uint32_t	align;		/* alignment as a power of 2 */
};
```

通用二进制文件开始是fat_header结构体，magic可以让系统内核读取该文件时候知道是通用二进制文件；nfat_arch表明下面有多少个fat_arch结构体（也可以说这个通用二进制文件包含多少个Mach-O）。

fat_arch结构体是描述Mach-O。cputype和cpusubtype说明Mach-O适用什么平台；offset（偏移）、size（大小）和align（页对齐）可以清楚描述Mach-O二进制位于通用二进制文件哪里。

### 操作通用二进制文件的常用命令

```shell
file 命令查看
$ file bq   
bq: Mach-O universal binary with 2 architectures: [arm_v7:Mach-O executable arm_v7] [arm64]
bq (for architecture armv7):	Mach-O executable arm_v7
bq (for architecture arm64):	Mach-O 64-bit executable arm64

otool 命令查看fat_header信息
$ otool -f -V bq
Fat headers
fat_magic FAT_MAGIC
nfat_arch 2
architecture armv7
    cputype CPU_TYPE_ARM
    cpusubtype CPU_SUBTYPE_ARM_V7
    capabilities 0x0
    offset 16384
    size 74952848
    align 2^14 (16384)
architecture arm64
    cputype CPU_TYPE_ARM64
    cpusubtype CPU_SUBTYPE_ARM64_ALL
    capabilities 0x0
    offset 74973184
    size 84135936
    align 2^14 (16384)
    
    
lipo(脂肪) 可以增、删、提取胖二进制文件中的特定架构（Mach-O）

提取特定Mach-O
lipo bq -extract armv7 -o bq_v7   

删除特定Mach-O
lipo bq -remove armv7 -o bq_v7

瘦身为Mach-O文件格式
lipo bq -thin armv7 -o bq_v7

```

### 通用二进制文件意义

从上面可以知道，尽管通用二进制文件会占用大量的磁盘空间，但是系统会挑选合适的Mach-O来执行，不相关的架构代码不会占用内存空间，且执行效率高了。

挑选合适的Mach-O的函数定义在<mach-o/arch.h>中，NXGetLocalArchInfo()函数获得主机的架构信息，NXFindBestFatArch()函数匹配最合适的Mach-O。

## Mach-O 文件

### Mach-O 术语

Mach-O 是针对不同运行时可执行文件的文件类型。

文件类型：

- Executable： 应用的主要二进制
- Dylib： 动态链接库（又称 DSO 或 DLL）
- Bundle： 不能被链接的 Dylib，只能在运行时使用 `dlopen()` 加载，可当做 macOS 的插件。

Image： executable，dylib 或 bundle
Framework： 包含 Dylib 以及资源文件和头文件的文件夹

### Mach-O 文件是什么

Mach-O文件格式就是COFF（Common file format）格式的变种。而COFF引入了“段”的机制，不同的Mach-O文件可以拥有不同数量和不同类型的“段”。Mach-O目标文件是源代码编译得到的文件，那至少文件里有机器指令、数据吧。其实除了这些之外，还有链接时候需要的一些信息，比如符号表、调试信息、字符串等。然后按照不同的信息，放在不同的“段”（segment）中。机器指令一般放在代码段里，全局变量和局部静态变量一般放在数据段里。

这里简单说下数据分段的好处，比如数据和机器指令分段：

1. 数据和指令可以被映射到两个不同的虚拟内存区域。数据区域是可读写的，指令区域是只读可执行。那就可以方便分别设置这两个区域的操作权限。
2. 两个区域分离，有助于提高缓存的命中率。（提高了程序的局部性）
3. 最主要是，系统运行多个该程序的副本时，它们指令是一样的，那内存只需要保存一份指令部分，可读写的数据区域进程私有。是不是节约了内存，动态链接那篇也是讲这样的方式来节约内存。

### Mach-O 格式

Mach-O文件主要由：Header、Load Commands、Segement三部分组成。而每个 segement 又被划分成一些 Section。

Segment 的名字都是大写的，且空间大小为页的整数。页的大小跟硬件有关，在 Arm64 架构一页是 16KB，其余为 4KB。

section 虽然没有整数倍页大小的限制，但是 section 之间不会有重叠。

几乎所有 Mach-O 都包含这三个段（segment）： `__TEXT`,`__DATA` 和 `__LINKEDIT`：

- `__TEXT` 包含 Mach header，被执行的代码和只读常量（如C 字符串）。只读可执行（r-x）。
- `__DATA` 包含全局变量，静态变量等。可读写（rw-）。
- `__LINKEDIT` 包含了加载程序的『元数据』，比如函数的名称和地址。只读（r–）。

#### Mach Header

文件最开始的Header是mach_header结构体，定义在<mach-o/loader.h>。

```objective-c
//后面默认都讲64位操作系统的，老早就淘汰的古董机iPhone5s就是64位操作系统了。。。
struct mach_header_64 {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
	uint32_t	reserved;	/* reserved */
};
```

1. magic：0xfeedface(32位) 0xfeedfacf(64位)，系统内核用来判断是否是mach-o格式
2. cputype和cpusubtype： 作用同上面胖二进制文件里的
3. filetype：由于可执行文件、目标文件、静态库和动态库等都是mach-o格式，所以需要filetype来说明mach-o文件是属于哪种文件。
4. ncms：加载命令的条数 （加载命令紧跟Header之后）
5. sizeofcmds：加载命令的大小
6. 动态连接器（dyld）的标志，可以先不管
7. reserved：保留字段

其中filetype常取字段有：

```objective-c
#define	MH_OBJECT	0x1	 目标文件	
#define	MH_EXECUTE	0x2	可执行文件	
#define	MH_DYLIB	0x6	 动态库	
#define	MH_DYLINKER	0x7	动态连接器	
#define	MH_DSYM		0xa	存储二进制文件符号信息，用于Debug分析
```

Mach Header 里会有 Mach-O 的 CPU 信息，以及 Load Command 的信息。

可以使用 otool 查看内容：

```shell
otool -v -h main.o
// 注意参数 -h 标示
```

具体的结果：

```
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64   ARM64        ALL  0x00      OBJECT    20       2000 SUBSECTIONS_VIA_SYMBOLS
```



#### Load Command

> 进程是特殊文件在内存中加载得到的结果。那这种文件的格式必须是系统内核可以理解的，系统内核才能正确解析。 

Load Command 包含 Mach-O 里命令类型信息，名称和二进制文件的位置。

使用 otool 命令可以查看详细：

```shell
otool -v -l a.out
// 注意参数 -l 标示
```

Mach-O有不同类型的“段”（Segement），且系统内核（或链接器）需要不同的加载方式来加载对应的段，而加载命令就是指导系统内核如何加载，所以有了不同的加载命令。

** load command**里的 cmd 是以 LC 开头定义的宏，可以参看 loader.h 里的定义，有50多个，主要的是：

- LC_SEGMENT_64(_PAGEZERO)
- LC_SEGMENT_64(_TEXT)
- LC_SEGMENT_64(_DATA)
- LC_SEGMENT_64(_LINKEDIT)
- LC_DYLD_INFO_ONLY
- LC_SYMTAB
- LC_DYSYMTAB
- LC_LOAD_DYLINKER
- LC_UUID
- LC_BUILD_VERSION
- LC_SOURCE_VERSION
- LC_MAIN
- LC_LOAD_DYLIB(libSystem.B.dylib)
- LC_FUNCTION_STARTS
- LC_DATA_IN_CODE

每个 command 的结构都是独立的，前两个字段 cmd 和 cmdsize 是一样的。



**LC_SEGMENT_64**

为了讲清楚Mach-O格式，我仅讲一个最普通且有代表意义的加载命令：段加载命令（LC_SEGMENT_64），其它加载命令，后面篇章用到时候，再具体讲解。

```objective-c
// 定义在<mach-o/loader.h>
struct segment_command_64 { /* for 64-bit architectures */
	uint32_t	cmd;		/* LC_SEGMENT_64 */
	uint32_t	cmdsize;	/* includes sizeof section_64 structs */
	char		segname[16];	/* segment name */
	uint64_t	vmaddr;		/* memory address of this segment */
	uint64_t	vmsize;		/* memory size of this segment */
	uint64_t	fileoff;	/* file offset of this segment */
	uint64_t	filesize;	/* amount to map from the file */
	vm_prot_t	maxprot;	/* maximum VM protection */
	vm_prot_t	initprot;	/* initial VM protection */
	uint32_t	nsects;		/* number of sections in segment */
	uint32_t	flags;		/* flags */
};
```

1. cmd表示加载命令类型，cmdsize表示加载命令大小（还包括了紧跟其后的nsects个section的大小）；需要知道的是，虽然不同加载命令的结构体不同，但是所有结构体的前两个字段都是cmd和cmdsize。这样系统在迭代所有加载命令时候，可以准确找到每个加载命令。
2. segname：加载命令名字
3. 从fileoff（偏移）处，取filesize字节的二进制数据，放到内存的vmaddr处的vmsize字节。（fileoff处到filesize字节的二进制数据，就是“段”）
4. 每一个段的权限相同（或者说，编译时候，编译器把相同权限的数据放在一起，成为段），其权限根据initprot初始化，initprot指定了如何通过读/写/执行位初始化页面的保护级别。段的保护设置可以动态改变，但是不能超过maxprot中指定的值（在iOS中，+x和+w是互斥的）。
5. nsects：段中section数量

#### Section64 Header

```objective-c
struct section_64 { /* for 64-bit architectures */
	char		sectname[16];	/* name of this section */
	char		segname[16];	/* segment this section goes in */
	uint64_t	addr;		/* memory address of this section */
	uint64_t	size;		/* size in bytes of this section */
	uint32_t	offset;		/* file offset of this section */
	uint32_t	align;		/* section alignment (power of 2) */
	uint32_t	reloff;		/* file offset of relocation entries */
	uint32_t	nreloc;		/* number of relocation entries */
	uint32_t	flags;		/* flags (section type and attributes)*/
	uint32_t	reserved1;	/* reserved (for offset or index) */
	uint32_t	reserved2;	/* reserved (for count or sizeof) */
	uint32_t	reserved3;	/* reserved */
};
```

加载命令如果有section，后面会紧跟nsects个section。section的header结构体是一样的。

#### 为什么要同时存在segment和section

真的要讲清这个，需要理解虚拟内存。

其实从链接的角度来看，Mach-O文件是按照section来存储文件的，segment只不过是把多个section打包放在一起而已；但是从Mach-O文件装载到内存的角度来看，Mach-O文件是按照segment（编译时候，编译器把相同权限的数据放在一起，成为segment）来存储的，即使一个segment里的内容小于1页空间的内存，但是还是会占用一页空间的内存，所以segment里不仅有filesize，也有vmsize，而section不需要有vmsize。

这样做，是为了节约内存，减少页面内部碎片。

#### Data

Data 由 Segment 的数据组成，是 Mach-O 占比最多的部分，有代码有数据，比如符号表。Data 共三个 Segment，`__TEXT`、`__DATA`、`__LINKEDIT`。其中`__TEXT` 和 `__DATA` 对应一个或多个 Section，`__LINKEDIT` 没有 Section，需要配合 LC_SYMTAB 来解析 symbol table 和 string table。这些里面是 Mach-O 的主要数据。

主要 Section：

- __nl_symbol_ptr：包含 non-lazy 符号指针，mach-o/loader.h 里有详细说明。服务 dyld_stub_binder 处理的符号。
- **la_symbol_ptr：**stubs 第一个 jump 目标地址。动态库的符号指针地址。
- **got：（Global Offset Table 全局偏移表）二进制文件的全局偏移表 GOT，也包含 S_NON_LAZY_SYMBOL_POINTERS 标记的 non-lazy 符号指针。服务于** TEXT Segment 里的符号。可以将**got 看作一个表，里面每项都是一个地址值。**got 的每项在加载期间都会被 dyld 重写，所以会在 **DATA Segment 中。**got 用来存放 non-lazy 符号最终地址，为 dyld 所用。dylib 外部符号对于全局变量和常量引用地址会指到 __got。
- __lazy_symbol：包含 lazy 符号，首次使用时绑定。
- **stubs：跳转表，重定向到 lazy 和 non-lazy 符号的 section。被标记为 S_SYMBOL_STUBS。**TEXT Segment 里代码和 dylib 外部符号的引用地址对函数符号的引用都指向了 **stubs。其中每项都是 jmp 代码间接寻址，可跳到** la_symbol_ptr Section 中。
- **stub_helper：lazy 动态绑定符号的辅助函数。可跳到** nl_symbol_ptr Section 中。
- __text：机器码，也是实际代码，包含所有功能。
- __cstring：常量。只读 C 字符串。
- __const：初始化过的常量。
- __objc：Objective-C 语言 runtime 的支持。
- __data：初始化过的变量。
- __bss：未初始化的静态变量。
- __unwind_info：生成异常处理信息。
- __eh_frame：DWARF2 unwind 可执行文件代码信息，用于调试。
- string table：以空值终止的字符串序列。
- symbol table：通过 LC_SYMTAB 命令找到 symbol table，其包含所有用到的符号信息。结构体 nlist_64描述了符号的基本信息。nlist_64 结构体中 n_type 字段是一个8位复合字段，其中bit[0:1]表示是外部符号，bit[5:8]表调试符号，bit[4:5]表示私有 external 符号，bit[1:4]是符号类型，有 N_UNDF 未定义、N_ABS 绝对地址、N_SECT 本地符号、N_PBUD 预绑定符号、N_INDR 同名符号几种类型。
- indirect symbol table：每项都是一个 index 值，指向 symbol table 中的项。由 LC_DYSYMTAB 定义，和**nl_symbol_ptr 和** lazy_symbol 一起为 **stubs 和** got 等 Section 服务。



## 参考文章

[Apple 操作系统可执行文件 Mach-O](https://ming1016.github.io/2020/03/29/apple-system-executable-file-macho/)

[iOS程序员的自我修养-MachO文件结构分析](https://juejin.im/post/5d5275b251882505417927b5#heading-9)

