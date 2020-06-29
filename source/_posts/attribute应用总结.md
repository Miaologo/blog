---
title: attribute应用总结
date: 2020-05-28 00:05:14
tags:
	- iOS
	- GCC	
	- LLVM
---

编译属性 `__attribute__`

## `__attribute__` 介绍

`__attribute__`是一个编译属性，用于向编译器描述特殊的标识、错误检查或高级优化。它是GNU C特色之一，系统中有许多地方使用到。 `__attribute__`可以设置函数属性（Function Attribute ）、变量属性（Variable Attribute ）和类型属性（Type Attribute)等。

## `__attribute__` 格式

```cpp
__attribute__ ((attribute-list)) 
```

## `__attribute__` 常用的编译属性及简单应用

### format

这个属性指定一个函数比如printf，scanf作为参数，这使编译器能够根据代码中提供的参数检查格式字符串。对于追踪难以发现的错误非常有帮助。

**format参数的使用如下：**

```cpp
format (archetype, string-index, first-to-check)
```

第一参数需要传递`archetype`指定是哪种风格,这里是 NSString；`string-index`指定传入函数的第几个参数是格式化字符串；`first-to-check`指定第一个可变参数所在的索引.

C中的使用方法

```cpp
extern int my_printf (void *my_object, const char *my_format, ...) __attribute__((format(printf, 2, 3)));
```

在Objective-C 中通过使用`__NSString__`格式达到同样的效果，就像在`NSString +stringWithFormat:`和`NSLog()`里使用字符串格式一样

```objectivec
FOUNDATION_EXPORT void NSLog(NSString *format, ...) NS_FORMAT_FUNCTION(1,2);
+ (instancetype)stringWithFormat:(NSString *)format, ... NS_FORMAT_FUNCTION(1,2);
```

### `__attribute__((constructor))`

确保此函数在 在`main`函数被调用之前调用，iOS中在`+load`之后`main`之前执行。
 `constructor`和`destructor`会在`ELF`文件中添加两个段-`.ctors`和`.dtors`。当动态库或程序在加载时，会检查是否存在这两个段，如果存在执行对应的代码。

```cpp
__attribute__((constructor))
static void beforeMain(void) {
    NSLog(@"beforeMain");
}
```



```kotlin
__attribute__((constructor(101))) // 里面的数字越小优先级越高，1 ~ 100 为系统保留
```

### `__attribute__((destructor))`

```cpp
__attribute__((destructor))
static void afterMain(void) {
    NSLog(@"afterMain");
}
```

确保此函数在 在`main`函数被调用之后调

### `__attribute__((cleanup))`

用于修饰一个变量，在它的作用域结束时可以自动执行一个指定的方法

关于这个**Sunny**在[黑魔法__attribute__((cleanup))](http://blog.sunnyxx.com/2014/09/15/objc-attribute-cleanup/)中讲的很好很细，建议看看。

**iOS中的应用**

既然`__attribute__((cleanup(...)))`可以用来修饰变量，所以也可以用来修饰block

```cpp
// void(^block)(void)的指针是void(^*block)(void)
static void blockCleanUp(__strong void(^*block)(void)) {
    (*block)();
}
```

这里不得不提万能的Reactive Cocoa中神奇的`@onExit`方法，其实正是上面的写法，简单定义个宏：

```csharp
#define onExit\
    __strong void(^block)(void) __attribute__((cleanup(blockCleanUp), unused)) = ^
```

这样的写法可以将成对出现的代码写在一起，比如说一个lock，用了onExit之后，代码更集中了：

```objectivec
NSRecursiveLock *aLock = [[NSRecursiveLock alloc] init];
[aLock lock]; onExit { [aLock unlock]; };
```

当我看到这段代码的时候第一个想到就是Swift中defer关键字

```csharp
lock.lock(); defer { lock.unlock() }
```

### used

`used`的作用是告诉编译器，我声明的这个符号是需要保留的。被`used`修饰以后，意味着即使函数没有被引用，在`Release`下也不会被优化。如果不加这个修饰，那么`Release`环境链接器会去掉没有被引用的段。[gun的官方文档](https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#Common-Variable-Attributes)

> This attribute, attached to a variable with static storage, means that the variable must be emitted even if it appears that the variable is not referenced.

> When applied to a static data member of a C++ class template, the attribute also means that the member is instantiated if the class itself is instantiated.

iOS中的运用，[BeeHive](https://github.com/alibaba/BeeHive)中的一段代码。

```cpp
#define BeeHiveDATA(sectname) __attribute((used, section("__DATA,"#sectname" ")))
```

### nonnull

这个属性指定函数的的某些参数不能是空指针

```cpp
extern void *
my_memcpy (void *dest, const void *src, size_t len)
  __attribute__((nonnull (1, 2)));
```

**iOS中的应用**

```objectivec
- (int)addNum1:(int *)num1 num2:(int *)num2  __attribute__((nonnull (1,2))){//1,2表示第一个和第二个参数不能为空
    return  *num1 + *num2;
}

- (NSString *)getHost:(NSURL *)url __attribute__((nonnull (1))){//第一个参数不能为空
    return url.host;
}
```

### objc_runtime_name

用于 `@interface` 或 `@protocol`，将类或协议的名字在编译时指定成另一个

```bash
__attribute__((objc_runtime_name("<#OtherClassName#>")))
```

**iOS中的应用**

```objectivec
 __attribute__((objc_runtime_name("OtherTest")))
 @interface Test : NSObject
 @end
 
 NSLog(@"%@", NSStringFromClass([Test class])); // "OtherTest"
```

这个属性可以用来做代码混淆

### noreturn

几个标注库函数，例如abort exit，没有返回值。GCC能够自动识别这种情况。`noreturn`属性指定像这样的任何不需要返回值的函数。当遇到类似函数还未运行到return语句就需要退出来的情况，该属性可以避免出现错误信息。

**iOS中的运用**

AFNetworking库为它的网络请求显示入口函数使用了该属性。这个在生成一个专用的线程时使用，保证分离的线程能在应用的整个生命周期继续执行

```objectivec
+ (void) __attribute__((noreturn)) networkRequestThreadEntryPoint:(id)__unused object {
    do {
        @autoreleasepool {
            [[NSRunLoop currentRunLoop] run];
        }
    } while (YES);
}
```

### noinline & always_inline

内联函数:内联函数从源代码层看，有函数的结构，而在编译后，却不具备函数的性质。内联函数不是在调用时发生控制转移，而是在编译时将函数体嵌入在每一个调用处。编译时，类似宏替换，使用函数体替换调用处的函数名。一般在代码中用inline修饰，但是能否形成内联函数，需要看编译器对该函数定义的具体处理

- `noinline` 不内联
- `always_inline` 总是内联
- 这两个都是用在函数上

内联的本质是用代码块直接替换掉函数调用处,好处是:快代码的执行，减少系统开销.适用场景:

- 这个函数更小
- 这个函数不被经常调用

```cpp
void test(int a) __attribute__((always_inline));
```

这个在Swift有类似用法

```swift
extension NSLock {
    
    @inline(__always)
    func executeWithLock(_ block: () -> Void) {
        lock()
        
        block()
        
        unlock()
    }
}
```

### warn_unused_result

当函数或者方法的返回值很重要时，要求调用者必须检查或者使用返回值，否则编译器会发出警告提示。

```java
- (BOOL)availiable __attribute__((warn_unused_result))
{
   return 10;
}
```

在Swift中应该是几乎所有方法都是`warn_unused_result`，可以通过`@discardableResult`去掉警告提示。

## Clang特有的

就像GCC的许多特性一样，Clang支持`__attribute__`，而且添加了一些自己的小扩展。为了检查一个特殊属性的可用性，你可以使用`__has_attribute`指令。

### availability

Clang引入了可用性属性，这个属性可以在声明中描述跟系统版本有关的生命周期。例如：

官方例子

```objectivec
- (CGSize)sizeWithFont:(UIFont *)font NS_DEPRECATED_IOS(2_0, 7_0, "Use -sizeWithAttributes:") __TVOS_PROHIBITED;

//来看一下 后边的宏
 #define NS_DEPRECATED_IOS(_iosIntro, _iosDep, ...) CF_DEPRECATED_IOS(_iosIntro, _iosDep, __VA_ARGS__)

define CF_DEPRECATED_IOS(_iosIntro, _iosDep, ...) __attribute__((availability(ios,introduced=_iosIntro,deprecated=_iosDep,message="" __VA_ARGS__)))

//宏展开以后如下
__attribute__((availability(ios,introduced=2_0,deprecated=7_0,message=""__VA_ARGS__)));
//ios即是iOS平台
//introduced 从哪个版本开始使用
//deprecated 从哪个版本开始弃用
//message    警告的消息
```

- `introduced`: 声明被引入的第一个版本信息。
- `deprecated`: 第一次不建议使用的版本，意味着使用者应该移除这个方法的使用。
- `obsoleted`: 第一次被废弃的版本，意味着已经被移除，不能够使用了。
- `unavailable`: 意味着这个平台不支持使用。
- `message`: 当Clang发出一些关于废弃或不建议使用的警告时的文本。用于引导使用者不要使用改接口了。

支持的平台有：

- ios: 苹果的iOS操作系统。最小部署目标平台版本是通过`-mios-version-min=*version*或-miphoneos-version-min=*version*`命令行指定的。
- macosx: 苹果的OS X操作系统。最小部署目标平台版本是通过`-mmacosx-version-min=*version*`命令行指定的。

```objectivec
//如果经常用,建议定义成类似系统的宏
- (void)oldMethod:(NSString *)string __attribute__((availability(ios,introduced=2_0,deprecated=7_0,message="用 -newMethod: 这个方法替代 "))){
    NSLog(@"我是旧方法,不要调我");
}

- (void)newMethod:(NSString *)string{
    NSLog(@"我是新方法");
}
```

在swift中也有类似的用法

```kotlin
@available(iOS 6.0, *)
    public var minimumScaleFactor: CGFloat // default is 0.0
```

### unavailable

告诉编译器该方法不可用，如果强行调用编译器会提示错误。比如某个类在构造的时候不想直接通过`init`来初始化，只能通过特定的初始化方法()比如单例，就可以将`init`方法标记为`unavailable`。

```cpp
//系统的宏,可以直接拿来用
 #define UNAVAILABLE_ATTRIBUTE __attribute__((unavailable))

 #define NS_UNAVAILABLE UNAVAILABLE_ATTRIBUTE
```



```objectivec
@interface Person : NSObject

@property(nonatomic,copy) NSString *name;

@property(nonatomic,assign) NSUInteger age;

- (instancetype)init NS_UNAVAILABLE;

- (instancetype)initWithName:(NSString *)name age:(NSUInteger)age;

@end
```

实际上unavailable后面可以跟参数,显示一些信息,如:



```cpp
//系统的
 #define NS_AUTOMATED_REFCOUNT_UNAVAILABLE __attribute__((unavailable("not available in automatic reference counting mode")))
```

### overloadable

Clang在C中提供对C++标准函数重载的支持。函数重载在C中是通过overloadable属性引入的。例如：你可以重载tgsin函数，写出sin函数在入参不同时的不同版本。用于c语言函数,可以定义若干个函数名相同，但参数不同的方法，调用时编译器会自动根据参数选择函数原型。

```cpp
__attribute__((overloadable)) void print(NSString *string){
    NSLog(@"%@",string);
}

__attribute__((overloadable)) void print(int num){
    NSLog(@"%d",num);
}
```

## __attribute__ 在iOS开发中的复杂应用

说了这么多重点来了，那么这些属性在iOS上有哪些奇妙的运用呢？有些比较简单的运用在介绍属性的时候就说了，这里主要讲一些比较复杂的运用。

### Swift没有+load方法的替代带方案

```objectivec
static void __attribute__ ((constructor)) Initer() {
    Class class = NSClassFromString(@"AnnotationDemo.MyInitThingy");
    SEL selector = NSSelectorFromString(@"appWillLaunch:");

    NSNotificationCenter *center = [NSNotificationCenter defaultCenter];

    [center addObserver:class
               selector:selector
                   name:UIApplicationDidFinishLaunchingNotification
                 object:nil];
}
```



```swift
class MyInitThingy: NSObject {
    @objc static func appWillLaunch(_: Notification) {
        print("App Will Launch")
    }
}
```

### BeeHive模块注册

模块注册有三种方式：Annotation方式注册、读取本地plist方式注册、Load方法注册。

**首先把数据放在可执行文件的自定义数据段**

```objectivec
// 通过BeeHiveMod宏进行Annotation标记

#ifndef BeehiveModSectName

#define BeehiveModSectName "BeehiveMods"

#endif

#ifndef BeehiveServiceSectName

#define BeehiveServiceSectName "BeehiveServices"

#endif


#define BeeHiveDATA(sectname) __attribute((used, section("__DATA,"#sectname" ")))


// 这里我们就把数据存在data数据段里面的"BeehiveMods"段中
#define BeeHiveMod(name) \
class BeeHive; char * k##name##_mod BeeHiveDATA(BeehiveMods) = ""#name"";


#define BeeHiveService(servicename,impl) \
class BeeHive; char * k##servicename##_service BeeHiveDATA(BeehiveServices) = "{ \""#servicename"\" : \""#impl"\"}";

@interface BHAnnotation : NSObject

@end
```

**从Mach-O section中读取数据**

```objectivec
SArray<NSString *>* BHReadConfiguration(char *sectionName,const struct mach_header *mhp);
static void dyld_callback(const struct mach_header *mhp, intptr_t vmaddr_slide)
{
    NSArray *mods = BHReadConfiguration(BeehiveModSectName, mhp);
    for (NSString *modName in mods) {
        Class cls;
        if (modName) {
            cls = NSClassFromString(modName);
            
            if (cls) {
                [[BHModuleManager sharedManager] registerDynamicModule:cls];
            }
        }
    }
    
    //register services
    NSArray<NSString *> *services = BHReadConfiguration(BeehiveServiceSectName,mhp);
    for (NSString *map in services) {
        NSData *jsonData =  [map dataUsingEncoding:NSUTF8StringEncoding];
        NSError *error = nil;
        id json = [NSJSONSerialization JSONObjectWithData:jsonData options:0 error:&error];
        if (!error) {
            if ([json isKindOfClass:[NSDictionary class]] && [json allKeys].count) {
                
                NSString *protocol = [json allKeys][0];
                NSString *clsName  = [json allValues][0];
                
                if (protocol && clsName) {
                    [[BHServiceManager sharedManager] registerService:NSProtocolFromString(protocol) implClass:NSClassFromString(clsName)];
                }
                
            }
        }
    }
    
}
__attribute__((constructor))
void initProphet() {
    _dyld_register_func_for_add_image(dyld_callback);
}

NSArray<NSString *>* BHReadConfiguration(char *sectionName,const struct mach_header *mhp)
{
    NSMutableArray *configs = [NSMutableArray array];
    unsigned long size = 0;
#ifndef __LP64__
    uintptr_t *memory = (uintptr_t*)getsectiondata(mhp, SEG_DATA, sectionName, &size);
#else
    const struct mach_header_64 *mhp64 = (const struct mach_header_64 *)mhp;
    uintptr_t *memory = (uintptr_t*)getsectiondata(mhp64, SEG_DATA, sectionName, &size);
#endif
    
    unsigned long counter = size/sizeof(void*);
    for(int idx = 0; idx < counter; ++idx){
        char *string = (char*)memory[idx];
        NSString *str = [NSString stringWithUTF8String:string];
        if(!str)continue;
        
        BHLog(@"config = %@", str);
        if(str) [configs addObject:str];
    }
    
    return configs;

    
}

@implementation BHAnnotation

@end
```

`__attribute__((constructor))`就是保证在`main`之前读取所有注册信息。

**使用**

```kotlin
@BeeHiveMod(ShopModule)
@interface ShopModule() <BHModuleProtocol>

@end
@implementation ShopModule
```

### 延迟 premain code

把+load等main函数之前的代码移植到了main函数之后。是[探一种延迟 premain code 的方法](https://everettjf.github.io/2017/03/06/a-method-of-delay-premain-code/)这篇文章提出的，我还没有尝试过。

原理是把函数地址放到QWLoadable段中，然后主程序在启动时获取QWLoadable的内容，并逐个调用。

库的地址 [LoadableMacro](https://github.com/everettjf/Yolo/tree/master/LoadableMacro)

作者测试下来，100个函数地址的读取，在iPhone5的设备上读取不到1ms。新增了这不到1ms的耗时（这1ms也是可审计的），带来了所有启动阶段行为的可审计，以及最重要的Patch能力。

## 补充

### `__attribute__((__naked__))`

`__attribute__((__naked__))`，这个作用是为了让当前的函数生成的汇编代码，不会存在额外的 prologue/epilogue 中的压栈消栈操作


## msgSend observe

这个来自于[质量监控-卡顿检测](https://juejin.im/post/5bb09795f265da0ac84946e0) 这篇文章
 OC方法的调用最终转换成msgSend的调用执行，通过在函数前后插入自定义的函数调用，维护一个函数栈结构可以获取每一个OC方法的调用耗时，以此进行性能分析与优化：

```objectivec
#define save() \
__asm volatile ( \
    "stp x8, x9, [sp, #-16]!\n" \
    "stp x6, x7, [sp, #-16]!\n" \
    "stp x4, x5, [sp, #-16]!\n" \
    "stp x2, x3, [sp, #-16]!\n" \
    "stp x0, x1, [sp, #-16]!\n");

#define resume() \
__asm volatile ( \
    "ldp x0, x1, [sp], #16\n" \
    "ldp x2, x3, [sp], #16\n" \
    "ldp x4, x5, [sp], #16\n" \
    "ldp x6, x7, [sp], #16\n" \
    "ldp x8, x9, [sp], #16\n" );
    
#define call(b, value) \
    __asm volatile ("stp x8, x9, [sp, #-16]!\n"); \
    __asm volatile ("mov x12, %0\n" :: "r"(value)); \
    __asm volatile ("ldp x8, x9, [sp], #16\n"); \
    __asm volatile (#b " x12\n");


__attribute__((__naked__)) static void hook_Objc_msgSend() {

    save()
    __asm volatile ("mov x2, lr\n");
    __asm volatile ("mov x3, x4\n");
    
    call(blr, &push_msgSend)
    resume()
    call(blr, orig_objc_msgSend)
    
    save()
    call(blr, &pop_msgSend)
    
    __asm volatile ("mov lr, x0\n");
    resume()
    __asm volatile ("ret\n");
}
```

[everettjf](https://everettjf.github.io/) 同样封装了一个库[FishhookObjcMsgSend](https://github.com/everettjf/Yolo/tree/master/FishhookObjcMsgSend)

## 参考文章

[探索 facebook iOS 客户端 - section FBInjectable](https://everettjf.github.io/2016/08/20/facebook-explore-section-fbinjectable/)   
[探一种延迟 premain code 的方法](https://everettjf.github.io/2017/03/06/a-method-of-delay-premain-code/) 
[__attribute__](https://nshipster.com/__attribute__/)   
[Specifying Attributes of Variables](https://gcc.gnu.org/onlinedocs/gcc-3.2/gcc/Variable-Attributes.html)    
[gnu-c-attributes](http://www.unixwiz.net/techtips/gnu-c-attributes.html)  
[BeeHive —— 一个优雅但还在完善中的解耦框架](https://halfrost.com/beehive/)    
[Declaring Attributes of Functions](https://gcc.gnu.org/onlinedocs/gcc-4.0.0/gcc/Function-Attributes.html)  
[__attribute__ 总结](https://www.jianshu.com/p/29eb7b5c8b2d)   
[OC中的 __attribute__](https://www.jianshu.com/p/529dc0501bd3)  
[Clang 拾遗之objc_designated_initializer](https://yq.aliyun.com/articles/5847)   
[Macro](https://github.com/sl-sl-sl-sl-sl/Macro)   
[Clang Attributes 黑魔法小记](https://blog.sunnyxx.com/2016/05/14/clang-attributes/)   
[黑魔法__attribute__((cleanup))](http://blog.sunnyxx.com/2014/09/15/objc-attribute-cleanup/)   
[质量监控-卡顿检测](https://juejin.im/post/5bb09795f265da0ac84946e0)
