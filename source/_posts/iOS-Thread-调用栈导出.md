---
title: iOS Thread 调用栈导出
date: 2019-07-24 19:11:46
tags:
	- iOS
	- 调用栈
---



### iOS 之 Thread调用栈学习

#### Mach 线程

iOS 是基于 Apple Darwin 内核，由 kernel、XNU 和 Runtime 组成，而 XNU 是 Darwin 的内核，它是“X is not UNIX”的缩写，是一个混合内核，由 Mach 微内核和 BSD 组成。Mach 内核是轻量级的平台，只能完成操作系统最基本的职责，比如：进程和线程、虚拟内存管理、任务调度、进程通信和消息传递机制。其他的工作，例如文件操作和设备访问，都由 BSD 层实现。

![img](https://elliotsomething.github.io/images/thread_study_01.png)

上述是权威著作《OS X Internal: A System Approach》给出的 Mac OS X 中进程子系统组成的概念，与 Mac OS X 类似，iOS 的线程技术也是基于 Mach 线程技术实现的，在 Mach 层中 thread_basic_info 结构体提供了线程的基本信息。

```
struct thread_basic_info {
        time_value_t    user_time;      /* user run time */
        time_value_t    system_time;    /* system run time */
        integer_t       cpu_usage;      /* scaled cpu usage percentage */
        policy_t        policy;         /* scheduling policy in effect */
        integer_t       run_state;      /* run state (see below) */
        integer_t       flags;          /* various flags (see below) */
        integer_t       suspend_count;  /* suspend count for thread */
        integer_t       sleep_time;     /* number of seconds that thread has been sleeping */
};
```

#### 任务（task）

任务（task）是一种容器（container）对象，虚拟内存空间和其他资源都是通过这个容器对象管理的，这些资源包括设备和其他句柄。严格地说，Mach 的任务并不是其他操作系统中所谓的进程，因为 Mach 作为一个微内核的操作系统，并没有提供“进程”的逻辑，而只是提供了最基本的实现。不过在 BSD 的模型中，这两个概念有1：1的简单映射，每一个 BSD 进程（也就是 OS X 进程）都在底层关联了一个 Mach 任务对象。

上面引用的是《OS X and iOS Kernel Programming》对 `Mach task` 的描述，Mach task 可以看作一个机器无关的 `thread` 执行环境的抽象；

一个 task 包含它的线程列表。内核提供了 `task_threads` API 调用获取指定 task 的线程列表，然后可以通过 `thread_info` API 调用来查询指定线程的信息，`thread_info` API 在 `thread_act.h`中定义。

```
kern_return_t task_threads
(
  task_t target_task,
  thread_act_array_t *act_list,
  mach_msg_type_number_t *act_listCnt
);

task_threads 将 target_task 任务中的所有线程保存在 act_list 数组中，数组中包含 act_listCnt 个线程。

kern_return_t thread_info
(
  thread_act_t target_act,
  thread_flavor_t flavor,
  thread_info_t thread_info_out,
  mach_msg_type_number_t *thread_info_outCnt
);
```

通过 act_list 数组可以读取该任务的所有线程，获取线程之后，对于每一个线程，可以用 `thread_get_state` 方法获取它的所有信息，信息填充在 `_STRUCT_MCONTEXT` 类型的参数中。这个方法中有两个参数随着 CPU 架构的不同而改变，因此需要注意不同 CPU 之间的区别（后面会统一讲）。在 `_STRUCT_MCONTEXT` 类型的结构体中，存储了当前线程的 `Stack Pointer` 和最顶部栈帧的 `Frame Pointer`，从而获取到了整个线程的调用栈。

线程的调用栈如下：（该图来自维基百科）![img](https://elliotsomething.github.io/images/thread_study_02.png)

上图表示了一个栈，它分为若干栈帧(frame)，每个栈帧对应一个函数调用；

那么我们先来学习下函数调用栈（汇编厉害的可以直接跳过）；

#### 汇编学习之函数调用栈

程序的执行过程可看作连续的函数调用。当一个函数执行完毕时，程序要回到调用指令的下一条指令(紧接call指令)处继续执行。函数调用过程通常使用堆栈实现，每个用户态进程对应一个调用栈结构(call stack)。编译器使用堆栈传递函数参数、保存返回地址、临时保存寄存器原有值(即函数调用的上下文)以备恢复以及存储本地局部变量。

不同处理器和编译器的堆栈布局、函数调用方法都可能不同，但堆栈的基本概念是一样的。

**理解EIP、ESP、EBP**

在x86处理器中，`EIP(Instruction Pointer)`是指令寄存器，指向处理器下条等待执行的指令地址(代码段内的偏移量)，每次执行完相应汇编指令EIP值就会增加。`ESP(Stack Pointer)`是堆栈指针寄存器，存放执行函数对应栈帧的栈顶地址(也是系统栈的顶部)，且始终指向栈顶；`EBP(Base Pointer`)是栈帧基址指针寄存器，存放执行函数对应栈帧的栈底地址，用于C运行库访问栈中的局部变量和参数。

注意，EIP是个特殊寄存器，不能像访问通用寄存器那样访问它，即找不到可用来寻址EIP并对其进行读写的操作码(OpCode)。EIP可被jmp、call和ret等指令隐含地改变(事实上它一直都在改变)。

不同架构的CPU，寄存器名称被添加不同前缀以指示寄存器的大小。例如x86架构用字母“e(extended)”作名称前缀，指示寄存器大小为32位；x86_64架构用字母“r”作名称前缀，指示各寄存器大小为64位。

**栈帧指针寄存器**

为了访问函数局部变量，必须能定位每个变量。局部变量相对于堆栈指针ESP的位置在进入函数时就已确定，理论上变量可用ESP加偏移量来引用，但ESP会在函数执行期随变量的压栈和出栈而变动。尽管某些情况下编译器能跟踪栈中的变量操作以修正偏移量，但要引入可观的管理开销。而且在有些机器上(如Intel处理器)，用ESP加偏移量来访问一个变量需要多条指令才能实现。

因此，许多编译器使用帧指针寄存器FP(Frame Pointer)记录栈帧基地址。局部变量和函数参数都可通过帧指针引用，因为它们到FP的距离不会受到压栈和出栈操作的影响。有些资料将帧指针称作局部基指针(LB-local base pointer)。

在Intel CPU中，寄存器BP(EBP)用作帧指针。在Motorola CPU中，除A7(堆栈指针SP)外的任何地址寄存器都可用作FP。当堆栈向下(低地址)增长时，以FP地址为基准，函数参数的偏移量是正值，而局部变量的偏移量是负值。

**函数调用回溯约定**

程序寄存器组是唯一能被所有函数共享的资源。虽然某一时刻只有一个函数在执行，但需保证当某个函数调用其他函数时，被调函数不会修改或覆盖主调函数稍后会使用到的寄存器值。因此，IA32采用一套统一的寄存器使用约定，所有函数(包括库函数)调用都必须遵守该约定。

根据惯例，寄存器%eax、%edx和%ecx为主调函数保存寄存器(caller-saved registers)，当函数调用时，若主调函数希望保持这些寄存器的值，则必须在调用前显式地将其保存在栈中；被调函数可以覆盖这些寄存器，而不会破坏主调函数所需的数据。寄存器%ebx、%esi和%edi为被调函数保存寄存器(callee-saved registers)，即被调函数在覆盖这些寄存器的值时，必须先将寄存器原值压入栈中保存起来，并在函数返回前从栈中恢复其原值，因为主调函数可能也在使用这些寄存器。此外，被调函数必须保持寄存器%ebp和%esp，并在函数返回后将其恢复到调用前的值，亦即必须恢复主调函数的栈帧。

**栈帧结构**

函数调用经常是嵌套的，在同一时刻，堆栈中会有多个函数的信息。每个未完成运行的函数占用一个独立的连续区域，称作栈帧(Stack Frame)。栈帧是堆栈的逻辑片段，当调用函数时逻辑栈帧被压入堆栈, 当函数返回时逻辑栈帧被从堆栈中弹出。栈帧存放着函数参数，局部变量及恢复前一栈帧所需要的数据等。

编译器利用栈帧，使得函数参数和函数中局部变量的分配与释放对程序员透明。编译器将控制权移交函数本身之前，插入特定代码将函数参数压入栈帧中，并分配足够的内存空间用于存放函数中的局部变量。使用栈帧的一个好处是使得递归变为可能，因为对函数的每次递归调用，都会分配给该函数一个新的栈帧，这样就巧妙地隔离当前调用与上次调用。

栈帧的边界由栈帧基地址指针EBP和堆栈指针ESP界定(指针存放在相应寄存器中)。EBP指向当前栈帧底部(高地址)，在当前栈帧内位置固定；ESP指向当前栈帧顶部(低地址)，当程序执行时ESP会随着数据的入栈和出栈而移动。因此函数中对大部分数据的访问都基于EBP进行。

为更具描述性，以下称EBP为帧基指针， ESP为栈顶指针，并在引用汇编代码时分别记为%ebp和%esp。

函数调用栈的典型内存布局如下图所示：![img](https://elliotsomething.github.io/images/thread_study_03.jpg)

图中给出主调函数(caller)和被调函数(callee)的栈帧布局，”m(%ebp)”表示以EBP为基地址、偏移量为m字节的内存空间(中的内容)。该图基于两个假设：第一，函数返回值不是结构体或联合体，否则第一个参数将位于”12(%ebp)” 处；第二，每个参数都是4字节大小(栈的粒度为4字节)。在本文后续章节将就参数的传递和大小问题做进一步的探讨。 此外，函数可以没有参数和局部变量，故图中“Argument(参数)”和“Local Variable(局部变量)”不是函数栈帧结构的必需部分。

从图中可以看出，函数调用时入栈顺序为：

实参N~1→主调函数返回地址→主调函数帧基指针EBP→被调函数局部变量1~N

其中，主调函数将参数按照调用约定依次入栈(图中为从右到左)，然后将指令指针EIP入栈以保存主调函数的返回地址(下一条待执行指令的地址)。进入被调函数时，被调函数将主调函数的帧基指针EBP入栈，并将主调函数的栈顶指针ESP值赋给被调函数的EBP(作为被调函数的栈底)，接着改变ESP值来为函数局部变量预留空间。此时被调函数帧基指针指向被调函数的栈底。以该地址为基准，向上(栈底方向)可获取主调函数的返回地址、参数值，向下(栈顶方向)能获取被调函数的局部变量值，而该地址处又存放着上一层主调函数的帧基指针值。本级调用结束后，将EBP指针值赋给ESP，使ESP再次指向被调函数栈底以释放局部变量；再将已压栈的主调函数帧基指针弹出到EBP，并弹出返回地址到EIP。ESP继续上移越过参数，最终回到函数调用前的状态，即恢复原来主调函数的栈帧。如此递归便形成函数调用栈。

EBP指针在当前函数运行过程中(未调用其他函数时)保持不变。在函数调用前，ESP指针指向栈顶地址，也是栈底地址。在函数完成现场保护之类的初始化工作后，ESP会始终指向当前函数栈帧的栈顶，此时，若当前函数又调用另一个函数，则会将此时的EBP视为旧EBP压栈，而与新调用函数有关的内容会从当前ESP所指向位置开始压栈。

以上内容出自[C语言函数调用栈(一)](http://www.cnblogs.com/clover-toeic/p/3755401.html)，总结上面的内容就是Stack Pointer(ESP栈指针)表示当前栈的顶部，Frame Pointer （EBP栈基指针）指向的地址中，存储了上一次 Stack Pointer 的值，也就是返回地址，其中EIP是每个函数的入口；了解了这些之后，通过函数调用栈递归，可以知道该线程的函数栈的所有函数调用；

#### 获取thread的状态信息

获取所有函数调用地址可以通过该方法获得，该方法可以访问目标内存空间；

```
kern_return_t vm_read_overwrite
(
	vm_map_t target_task,
	vm_address_t address,
	vm_size_t size,
	vm_address_t data,
	vm_size_t *outsize
);
```

我们只要通过thread的入口地址，就可以回溯整个函数调用栈，而thread的所有信息都保存在了`_STRUCT_MCONTEXT` 类型的参数中；

该函数的用法如下：

```
typedef struct BSStackFrameEntry{
    const struct BSStackFrameEntry *const previous;
    const uintptr_t return_address;
} BSStackFrameEntry;
kern_return_t bs_mach_copyMem(const void *const src, void *const dst, const size_t numBytes){
    vm_size_t bytesCopied = 0;
    return vm_read_overwrite(mach_task_self(), (vm_address_t)src, (vm_size_t)numBytes, (vm_address_t)dst, &bytesCopied);
}
uintptr_t bs_mach_framePointer(mcontext_t const machineContext){
    return machineContext->__ss.__ebp;
}
int main(){
  //获取线程的state   _STRUCT_MCONTEXT machineContext;
  mach_msg_type_number_t state_count = BS_THREAD_STATE_COUNT;
    kern_return_t kr = thread_get_state(thread, BS_THREAD_STATE, (thread_state_t)&machineContext->__ss, &state_count);
  //找出所有的函数调用地址   BSStackFrameEntry frame = {0};
  const uintptr_t framePtr = bs_mach_framePointer(&machineContext);
  bs_mach_copyMem((void *)framePtr, &frame, sizeof(frame)) != KERN_SUCCESS)
  for(; i < 50; i++) {
        backtraceBuffer[i] = frame.return_address;
        if(backtraceBuffer[i] == 0 ||
           frame.previous == 0 ||
           bs_mach_copyMem(frame.previous, &frame, sizeof(frame)) != KERN_SUCCESS) {
            break;
        }
    }
}
```

#### 函数符号化

由于我们在上面回溯线程调用栈拿到的是一组地址，所以将函数地址进行符号化，这样才能够转化为对我们有用信息；

**首先符号表是什么？**

符号表储存在 Mach-O 文件的 `__LINKEDIT` 段中，涉及其中的符号表`（Symbol Table）`和字符串表`（String Table）`。符号表在 Mach-O目标文件中的地址可以通过`LC_SYMTAB`加载命令指定的 `symoff`找到，对应的符号名称在`stroff`，总共有`nsyms`条符号信息；也就是说，通过`LC_SYMTAB`来找存储在`__LINKEDIT`中的符号地址地址；

那么我们首先要通过`__LINKEDIT`寻址，把该段的地址都找出来，这样就可以通过`LC_SYMTAB`来定位符号信息了，通过`__LINKEDIT`寻找镜像地址可以看我的另一篇博客[Mach-O学习](https://elliotsomething.github.io/2017/06/01/Mach-O学习/);

**补充：** 所有的符号地址全部存储在__LINKEDIT段中（注意是地址），所有的符号和字符串以及字符串对应符号所对应的函数指针（保存为一个结构体nlist，下面会讲到）都存在LC_SYMTAB加载命令区中。

**符号解析**

接下来就是定位符号信息了，首先LC_SYMTAB的数据结构定义如下：

```
struct symtab_command {
    uint32_t    cmd;        /* LC_SYMTAB */
    uint32_t    cmdsize;    /* sizeof(struct symtab_command) */
    uint32_t    symoff;        /* symbol table offset */
    uint32_t    nsyms;        /* number of symbol table entries */
    uint32_t    stroff;        /* string table offset */
    uint32_t    strsize;    /* string table size in bytes */
};
```

这里我们用 MachOView 随意打开一个可执行文件，找到其中的 Symbol Table 项。

![img](https://elliotsomething.github.io/images/thread_study_04.png)

符号表的结构是一个连续的列表，其中的每一项都是一个 struct nlist。

```
// 位于系统库 头文件中 struct nlist {
  union {
  //符号名在字符串表中的偏移量     uint32_t n_strx;
  } n_un;
  uint8_t n_type;
  uint8_t n_sect;
  int16_t n_desc;
  //符号在内存中的地址，类似于函数指针   uint32_t n_value;
};
```

这里重点关注第一项和最后一项，第一项是符号名在字符串表中的偏移量，用于表示函数名，最后一项是符号在内存中的地址，类似于函数指针（这里只说明大概的结构，详细的信息请参考官方Mach O文件格式的文档）。也就是说如果我们知道了符号名和内存地址的对应关系，我们是可以根据这个结构来逆向构造出符号表数据的。我们可以通过.n_un.n_strx来定位字符串表中相应的符号名。

![img](https://elliotsomething.github.io/images/thread_study_09.jpg)

上图是官方的解释了如何找到字符串表对应的符号（字符串）

知道了如何构造符号表，下一步就是收集符号名和内存地址的对应关系了。![img](https://elliotsomething.github.io/images/thread_study_05.png)

符号解析基本思路如下:

- 根据 Frame Pointer 拿到函数调用的地址（address）
- 寻找包含地址 (address) 的目标镜像(image)
- 拿到镜像文件的符号表、字符串表
- 根据 address 、符号表、字符串表的对应关系找到对应的函数名

这里有一点需要注意：由于虚拟地址和内存地址不一样，他们存在一个基于ASLR的偏移量，所以我们需要知道这个偏移才能够知道正确的地址，该偏移slide可以通过dyld_get_image_vmaddr_slide() 来进行获取，函数对应在符号表的地址、slide、frame Pointer address满足下面这个公式。

```
symbol address ＝ frame pointer address ＋ slide
链接时程序的基址 = __LINKEDIT.VM_Address -__LINKEDIT.File_Offset + silde的改变值
```

这里出现了一个 slide，那么slide是啥呢？先看一下ASLR

`ASLR：Address space layout randomization`，将可执行程序随机装载到内存中,这里的随机只是偏移，而不是打乱，具体做法就是通过内核将 Mach-O的段“平移”某个随机系数。slide 正是ASLR引入的偏移

```
// 链接时程序的基址 uintptr_t linkedit_base = (uintptr_t)slide + linkedit_segment->vmaddr - linkedit_segment->fileoff;

// 符号表的地址 = 基址 + 符号表偏移量 nlist_t *symtab = (nlist_t *)(linkedit_base + symtab_cmd->symoff);
// 字符串表的地址 = 基址 + 字符串表偏移量 char *strtab = (char *)(linkedit_base + symtab_cmd->stroff);

// 动态符号表地址 = 基址 + 动态符号表偏移量 uint32_t *indirect_symtab = (uint32_t *)(linkedit_base + dysymtab_cmd->indirectsymoff);
```

ok，我们拿到地址通过nlist数组取得信息之后，把解析之后的有效信息结果放入DL_info的结构体中（或者也可以不用放入，直接用一个`const char*`存储就行）；

```
/* * Structure filled in by dladdr(). */
typedef struct dl_info {
        const char      *dli_fname;     /* Pathname of shared object */
        void            *dli_fbase;     /* Base address of shared object */
        const char      *dli_sname;     /* Name of nearest symbol */
        void            *dli_saddr;     /* Address of nearest symbol */
} Dl_info;
```

该结构的解释如下：

- fname：路径名，例如`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation`
- dli_fbase：刚才讲到的共享对象的的起始地址（Base address of shared object，下面叫做基地址，比如上面的 CoreFoundation)
- dli_saddr ：符号的地址
- dli_sname：符号的名字，即函数信息：`[ViewController tableView:cellForRowAtIndexPath:]`

经过上面的一番折腾之后，Thread的函数调用栈基本就全部了解了，首先通过task任务取得所有线程，然后取得线程的state，通过ESP、EBP、EIP来回溯整个函数调用栈，找出所有的函数地址，通过偏移计算出真实的内存地址，最后进行符号化，该篇的大概内容就是这样了。

**补充：**

由于上层的NSThread和内核线程thread没有直接的API可以知道对应关系，但是幸运的是两者的name是一样的；pthread 也提供了一个方法 pthread_getname_np 来获取线程的名字（实际NSThread就是调用了pthread 提供的接口），所以可以通过给thread的name赋值，来找到对应的NSThread，这样就实现了两者的互相转化；但是需要注意的地方是：主线程设置 name 后无法用 pthread_getname_np 读取到；所以需要在在 load 方法里获取主线程的thread_t ID:

```
static mach_port_t main_thread_id;
+ (void)load {
    main_thread_id = mach_thread_self();
}
```

#### 附录

各平台寄存器以及thread的state的区别，直接用宏定义区别开来；

```
#if defined(__arm64__) #define BS_THREAD_STATE_COUNT ARM_THREAD_STATE64_COUNT #define BS_THREAD_STATE ARM_THREAD_STATE64 #define BS_FRAME_POINTER __fp #define BS_STACK_POINTER __sp #define BS_INSTRUCTION_ADDRESS __pc 
#elif defined(__arm__) #define BS_THREAD_STATE_COUNT ARM_THREAD_STATE_COUNT #define BS_THREAD_STATE ARM_THREAD_STATE #define BS_FRAME_POINTER __r[7] #define BS_STACK_POINTER __sp #define BS_INSTRUCTION_ADDRESS __pc 
#elif defined(__x86_64__) #define BS_THREAD_STATE_COUNT x86_THREAD_STATE64_COUNT #define BS_THREAD_STATE x86_THREAD_STATE64 #define BS_FRAME_POINTER __rbp #define BS_STACK_POINTER __rsp #define BS_INSTRUCTION_ADDRESS __rip 
#elif defined(__i386__) #define BS_THREAD_STATE_COUNT x86_THREAD_STATE32_COUNT #define BS_THREAD_STATE x86_THREAD_STATE32 #define BS_FRAME_POINTER __ebp #define BS_STACK_POINTER __esp #define BS_INSTRUCTION_ADDRESS __eip 
#endif
```

demo代码：

```
bool ksdl_dladdr(const uintptr_t address, Dl_info* const info) {
    // 初始 Dl_info     info->dli_fname = NULL;
    info->dli_fbase = NULL;
    info->dli_sname = NULL;
    info->dli_saddr = NULL;
    // image index     const uint32_t idx = ksdl_imageIndexContainingAddress(address);
    const struct mach_header* header = _dyld_get_image_header(idx);
    // slide     const uintptr_t imageVMAddrSlide = (uintptr_t)_dyld_get_image_vmaddr_slide(idx);
    const uintptr_t addressWithSlide = address - imageVMAddrSlide;
    // 段基址     const uintptr_t segmentBase = ksdl_segmentBaseOfImageIndex(idx) + imageVMAddrSlide;

    // 拿到了 Pathname     info->dli_fname = _dyld_get_image_name(idx);
    // 拿到了基地址     info->dli_fbase = (void*)header;

    // Find symbol tables and get whichever symbol is closest to the address     // 在符号表中查找哪个符号最接近这个指令的地址     // nlist     const STRUCT_NLIST* bestMatch = NULL;
    uintptr_t bestDistance = ULONG_MAX;
    // load commond     uintptr_t cmdPtr = ksdl_firstCmdAfterHeader(header);
    // header->ncmds 代表所有的加载命令，这里进行遍历，查找 LC_SYMTAB     if(cmdPtr == 0) {
		return false;
	}
	for(uint32_t iCmd = 0; iCmd < header->ncmds; iCmd++) {
		const struct load_command* loadCmd = (struct load_command*)cmdPtr;
		if(loadCmd->cmd == LC_SYMTAB) {
			const struct symtab_command* symtabCmd = (struct symtab_command*)cmdPtr;
			// 找到符号表 			const BS_NLIST* symbolTable = (BS_NLIST*)(segmentBase + symtabCmd->symoff);
			// 找到字符串表 			const uintptr_t stringTable = segmentBase + symtabCmd->stroff;
			// 遍历符号表 			for(uint32_t iSym = 0; iSym < symtabCmd->nsyms; iSym++) {
				// If n_value is 0, the symbol refers to an external object. 				if(symbolTable[iSym].n_value != 0) {
					uintptr_t symbolBase = symbolTable[iSym].n_value;
					uintptr_t currentDistance = addressWithSlide - symbolBase;
					// addr >= symbol.value 说明这个指令在这个函数入口中 					// 获取和address的距离找到最接近的一个 					// 离指令地址addr更近的函数入口地址，才是更准确的匹配项 					if((addressWithSlide >= symbolBase) && (currentDistance <= bestDistance)) {
						bestMatch = symbolTable + iSym;
						bestDistance = currentDistance;
					}
				}
			}
			if(bestMatch != NULL) {
				info->dli_saddr = (void*)(bestMatch->n_value + imageVMAddrSlide);
				// 符号的名字 				info->dli_sname = (char*)((intptr_t)stringTable + (intptr_t)bestMatch->n_un.n_strx);
				if(*info->dli_sname == '_') {
					info->dli_sname++;
				}
				// This happens if all symbols have been stripped. 				if(info->dli_saddr == info->dli_fbase && bestMatch->n_type == 3) {
					info->dli_sname = NULL;
				}
				break;
			}
		}
		cmdPtr += loadCmd->cmdsize;
	}

    return true;
}
```

#### 参考

- [获取任意线程调用栈的那些事](https://bestswifter.com/callstack/)
- [iOS 之 Thread调用栈学习]([https://elliotsomething.github.io/2017/06/28/thread%E5%AD%A6%E4%B9%A0/](https://elliotsomething.github.io/2017/06/28/thread学习/))
- [C语言函数调用栈(一)](http://www.cnblogs.com/clover-toeic/p/3755401.html)
- [获取任意线程调用栈的那些事](http://blog.csdn.net/abc649395594/article/details/52350426)
- [iOS中线程Call Stack的捕获和解析（二）](http://blog.csdn.net/jasonblog/article/details/49909209)
- [iOS 符号表恢复 & 逆向支付宝](https://juejin.im/entry/58478862ac502e006b0dfcb5)
- [nlist-Mach-O文件重定向信息数据结构分析](http://turingh.github.io/2016/05/24/nlist-Mach-O文件重定向信息数据结构分析/)
- [趣探 Mach-O：符号解析](http://ios.jobbole.com/89026/)
- [vm_read](http://web.mit.edu/darwin/src/modules/xnu/osfmk/man/vm_read.html)
- [Apple Open Source](https://opensource.apple.com/source/xnu/xnu-201/osfmk/vm/vm_user.c)

------

- 