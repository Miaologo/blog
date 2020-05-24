---
title: 关于iOS依赖注入（DependencyInjection）
date: 2020-05-09 15:16:48
tags:
---

##dependency injection 关于IOS依赖注入那些事

> 原文链接 https://runningyoung.github.io/2015/06/30/2015-08-04-dependency-injection/

　 本文介绍的是另一个屎上最牛叉的ios开发新框架，***最大的特点就是：帮助我们开发出松散耦合(loose coupled)、可维护、可测试的代码和程序。这条原则的做法是大家熟知的面向接口，或者说是面向抽象编程。\*** 众所周知 该编程思想在各大语言中都有体现如 jave C++ PHP以及.net中。当然设计模式的广泛程度远远大于这些，IOS 当然也不例外。 ***本文主要介绍本人在学习dependency injection的时候的学习过程以及对一些学习资料的总结，主要介绍ios中的两大框架[objection](https://github.com/atomicobject/objection)和[Typhoon](http://typhoonframework.org/)。\*** 闲话不多吹下面进入正题。



###什么是dependency injection？　

####简单来说：
　 关于什么是依赖注入，在Stack Overflow上面有一个问题，[如何向一个5岁的小孩解释依赖注入](http://stackoverflow.com/questions/1638919/how-to-explain-dependency-injection-to-a-5-year-old)，其中得分最高的一个答案是：

> “When you go and get things out of the refrigerator for yourself, you can cause problems. You might leave the door open, you might get something Mommy or Daddy doesn’t want you to have. You might even be looking for something we don’t even have or which has expired.
>
> What you should be doing is stating a need, “I need something to drink with lunch,” and then we will make sure you have something when you sit down to eat.”

映射到面向对象程序开发中就是：高层类(5岁小孩)应该依赖底层基础设施(家长)来提供必要的服务。

编写松耦合的代码说起来很简单，但是实际上写着写着就变成了紧耦合。

####更详细点的解释：

依赖倒置解决了高层次模块依赖于低层次模块和其细节的问题。

　　Dependency injection 是一个将行为从依赖中分离的技术，简单地说，它允许开发者定义一个方法函数依赖于外部其他各种交互，而不需要编码如何获得这些外部交互的实例。 这样就在各种组件之间解耦，从而获得干净的代码，相比依赖的硬编码， 一个组件只有在运行时才调用其所需要的其他组件，因此在代码运行时，通过特定的框架或容器，将其所需要的其他依赖组件进行注入，主动推入。

　　依赖注入可以看成是 反转控制 inversion of control 的一个特例。反转的是依赖，而不是其他，JNDI也是一种反转控制，它反转的JNDI名称或资源。参考： [“Inversion of Control Containers and the Dependency Injection pattern”](http://martinfowler.com/articles/injection.html)。

　　依赖注入是最早Spring和piconcontainer等提出，如今已经是一个缺省主流模式，并扩展到前端如Angular.js等等。

　　依赖注入与IOC模式类似工厂模式，是一种解决调用者和被调用者依赖耦合关系的模式，自2004年诞生以来，至今已经成为Java和其他领域的主流模式。它解决了对象之间的依赖关系，使得对象只依赖IOC/DI容器，不再直接相互依赖，实现松耦合，然后在对象创建时，由IOC/DI容器将其依赖的对象注入Inject其体内，故又称依赖注入依赖注射模式，最大程度实现松耦合，特别是Autowiring/Autowired自动配对引入，再结合Java的垃圾回收机制，使得在Java中，对象不再需要开发者自己创建，也需要开发者自己销毁，只需要直接使用即可，大大提升了开发效率。

　　详细解释：依赖注入说白一点，就是容器将某个类依赖的其他类注入到这个类中。 　　

###为什么要用dependency injection
　 依赖注入框架的运用可以帮我们将APP的设计分割成好几个模块，分给不同的开人员，当完成开发之后再进行合并充分解决了团队之间模块化分工的不足.借objic.io上一篇[关于Dependency Injection](http://www.objc.io/issues/15-testing/dependency-injection/)的一句话:

> My initial motivation for exploring DI came from doing test-driven development, because in TDD you constantly wrestle with the question of “How do I write a unit test for this?” But I discovered that DI is actually concerned with a bigger idea: that our code should be composed of modules that we snap together to build an application.
>
> There are many benefits to such an approach. Graham Lee’s article, [“Dependency Injection, iOS and You,”](http://www.bignerdranch.com/blog/dependency-injection-ios/) describes some of them: “to adapt… to new requirements, make bug fixes, add new features, and test components in isolation.”

大体意思:

我在探索DI最初的动机来自做测试驱动的开发，因为在TDD你不断地与问题搏斗“我怎样写单元测试吗？”但我发现，DI实际上是涉及一个更大的想法：即我们的代码应该由我们扣合在一起来构建应用程序模块。

有许多好处，这样的做法。格雷厄姆李的文章，[“依赖注入，iOS和你”](http://www.bignerdranch.com/blog/dependency-injection-ios/)，介绍了其中一些：“适应……新的要求，作出错误修复，隔离增加新的功能以及测试组件。”

用文字说这些概念其实很抽象，下面用几张图片说明下：

通过objection实现依赖注入后，就能更好地实现SRP(Single Responsibility Principle)，代码更简洁，心情更舒畅，生活更美好。拿Pinterest来说，下面的页面就可以划分为3个Section。

[点击查看图片](http://7xsugd.com2.z0.glb.clouddn.com/runningyoungBlog/images/DI01.png)

[![点击查看图片](http://m1.yea.im/1O8.png)](http://m1.yea.im/1O8.png)

各个Section可以由不同的人负责，然后串到一起就行，也能一定程度地避免MVC(Mess View Controller)的出现,对于提高开发成员的效率也会有不少的帮助。

　 **其实用简单的一句话来说就是： 通过DI设计模式，将项目模块化，以提高开发效率。**

###dependency injection试图解决什么问题呢
**我们知道，在IOS基本教程中有一个定律告诉我们：所有的对象都必须创建；或者说：使用对象之前必须创建，但是现在我们可以不必一定遵循这个定律了，我们可以从DI容器中直接获得一个对象然后直接使用，无需事先创建它们。**

　　**这种变革，就如同我们无需考虑对象销毁一样；因为IOS的 ARC 帮助我们实现了对象销毁；现在又无需考虑对象创建，对象的创建和销毁都无需考虑了，这给编程带来的影响是巨大的**

我们可以通过一篇博文了解下原理,该文章是由Limboy大神写的[使用objection来模块化开发iOS项目](http://limboy.me/ios/2014/04/15/use-objection-to-decouple-ios-project.html)

这里提前介绍了一个DI框架就是Objection,详细介绍就去看博文，在此只分析下DI的设计原理:

**类似于Objection的大部分DI框架，主要为我们提供了一个容器,来管理创建销毁对象，我们并不会过多考虑创建对象的方法内容以及其再其他类中的依赖关系，这些都在框架中为我们解决了，我们只需要再需要的时候调用即可，并不需要重复的导入import，避免不同类直接多次重复的import导致的依赖循环问题。**

###那么问题来了？如何学习dependency injection呢
> ios有关DI依赖注入的框架比较好用的有两个：[objection](https://github.com/atomicobject/objection) 和 [Typhoon](https://github.com/appsquickly/Typhoon)

下面就从几个方便来介绍下这两个框架

####[objection](https://github.com/atomicobject/objection) 和 [Typhoon](https://github.com/appsquickly/Typhoon)这两个框架有什么区别呢
其实这两个框架各有优势：

1.objection框架，使用起来比较灵活，用法比较简单。示例代码如下：

属性注册：

```
@class Engine, Brakes;

@interface Car : NSObject
{
  Engine *engine;
  Brakes *brakes;
   BOOL awake;  
}

// Will be filled in by objection
@property(nonatomic, strong) Engine *engine;
// Will be filled in by objection
@property(nonatomic, strong) Brakes *brakes;
@property(nonatomic) BOOL awake;

@implementation Car
objection_requires(@"engine", @"brakes") //属性的依赖注入
@synthesize engine, brakes, awake;
@end
```

方法注入：

```
@implementation Truck
objection_requires(@"engine", @"brakes")
objection_initializer(truckWithMake:model:)//方法的依赖注入
+ (instancetype)truckWithMake:(NSString *) make model: (NSString *)model {
  ...
}
@end
```

2.对比来说Typhoon的使用起来就比较规范，首先需要创建一个 TyphoonAssembly的子类。其需要注入的方法和属性都需要写在这个统一个子类中，当然可以实现不同的子类来完成不同的功能

```
@interface MiddleAgesAssembly : TyphoonAssembly

- (Knight*)basicKnight;

- (Knight*)cavalryMan;

- (id<Quest>)defaultQuest;

@end
```

属性注入：

```
- (Knight *)cavalryMan
{
    return [TyphoonDefinition withClass:[CavalryMan class] 
    configuration:^(TyphoonDefinition *definition) {

    [definition injectProperty:@selector(quest) with:[self defaultQuest]];
    [definition injectProperty:@selector(damselsRescued) with:@(12)];
}];
}
```

方法注入：

```
- (Knight *)knightWithMethodInjection
{
       return [TyphoonDefinition withClass:[Knight class] 
    configuration:^(TyphoonDefinition *definition) {
    [definition injectMethod:@selector(setQuest:andDamselsRescued:) 
        parameters:^(TyphoonMethod *method) {

        [method injectParameterWith:[self defaultQuest]];
        [method injectParameterWith:@321];
    }];
}];
}
```

3.当然还有一些硬性的区别就是Typhoon现在已经支持Swift。

4.两者维护时间都超过2年以上。

Tythoon官方介绍的优势：

```
1）Non-invasive. No macros or XML required. Uses powerful ObjC runtime instrumentation.

2）No magic strings – supports IDE refactoring, code-completion and compile-time checking.

3）Provides full-modularization and encapsulation of configuration details. Let your architecture tell a story.

4）Dependencies declared in any order. (The order that makes sense to humans).

5）Makes it easy to have multiple configurations of the same base-class or protocol.

 6）Supports injection of view controllers and storyboard integration. Supports both initializer and property injection, plus life-cycle management.

7）Powerful memory management features. Provides pre-configured objects, without the memory overhead of singletons.

8）Excellent support for circular dependencies.

9）Lean. Has a very low footprint, so is appropriate for CPU and memory constrained devices.

10）While being feature-packed, Typhoon weighs-in at just 3000 lines of code in total.

 11）Battle-tested — used in all kinds of Appstore-featured apps.
```

大体翻译过来：

```
1)非侵入性。不需要宏或XML。使用强大的ObjC运行时仪器。
2)没有魔法字符串——支持IDE重构,完成和编译时检查。
3)提供full-modularization和封装的配置细节。让你的架构告诉一个故事。
4)依赖关系中声明的任何顺序。(对人类有意义的顺序)。
5)很容易有多个配置相同的基类或协议。
6)支持注射的视图控制器和故事板集成。同时支持初始化器和属性注入,以及生命周期管理。
7)强大的内存管理功能。提供预配置对象,没有单件的内存开销。
8)优秀的支持循环依赖。
9)精益。占用很低,所以适合CPU和内存受限的设备。
10),功能强大,台风重总共只有3000行代码。
11)久经沙场,用于各种Appstore-featured应用。
```

　 　 针对这两个框架网上教程并不多，收集了一些比较有用的资料。最主要的用法还得看官方文档分别在：

[objection](https://github.com/atomicobject/objection) 和 [Typhoon](https://github.com/appsquickly/Typhoon)

###大体学习步骤：

####（一）了解依赖注入的原理

依赖注入各大语言都一样，相关资料：

其中最需要看的是来自

objc.io官网的博文 [Dependency Injection](http://www.objc.io/issues/15-testing/dependency-injection/) 和 Typhoon原创大神(Graham Lee)的文章 [Dependency Injection, iOS and You](https://www.bignerdranch.com/blog/dependency-injection-ios/) 不看后悔一辈子^_^

1.[CSDN专题系列–依赖注入及AOP简述](http://blog.csdn.net/column/details/aopbrief.html)

2.[某知名博主写的文章–依赖注入（Dependency Injection）模式](http://blog.csdn.net/yqj2065/article/details/8510074)

3.[来自博客园的深度理解–深度理解依赖注入](http://kb.cnblogs.com/page/45266/4/)

4.[国人翻译的国外牛人的文章—依赖注入——让iOS代码更简洁](http://blog.csdn.net/linshaolie/article/details/47037941#report)

####（二）进阶内容了解和学习框架

#####首先来看看有关objection的资料
资料比较少

1.Limboy大大大神的–[使用objection来模块化开发iOS项目](http://limboy.me/ios/2014/04/15/use-objection-to-decouple-ios-project.html)

2.[objection的官方github文档](https://github.com/atomicobject/objection)

#####其次Typhoon的教程可能就比较详细了—-[官网](http://typhoonframework.org/)

1.[首先来看官方github文档详细程度已经不需要其他教程了](https://github.com/appsquickly/Typhoon/wiki/Quick-Start)😄😄😄

2.[关于Typhoon的视频教程](http://datab.us/Search/Typhoon%2BFramework%2BPrimer%2BPlayListIDPLhU81D62nv-Yd5jCW9LRjI4_AfI5NwJhe)

3.[国外大神的教程整理 翻译（一）](http://blog.csdn.net/liangliang103377/article/details/47279819)

4.[国外大神的教程整理 翻译（二）](http://blog.csdn.net/liangliang103377/article/details/47279863)

5.[国外大神的教程整理 翻译（三）](http://blog.csdn.net/liangliang103377/article/details/47279899)

## DI 框架和参考文档

https://github.com/Swinject/Swinject

ios有关DI依赖注入的框架比较好用的有两个：[objection](https://github.com/atomicobject/objection) 和 [Typhoon](https://github.com/appsquickly/Typhoon)

[Swift中依赖注入的解耦策略](https://juejin.im/post/5cceaa3e6fb9a032143772fa)

[iOS一次高效的依赖注入](https://juejin.im/post/5b968736f265da0a951ebd34)

[Category 特性在 iOS 组件化中的应用与管控](https://tech.meituan.com/2018/11/08/ios-category-module-communicate.html)



[GitHub - jspahrsummers/libextobjc: A Cocoa library to extend the Objective-C programming language.](https://github.com/jspahrsummers/libextobjc) 里有一个 `EXTConcreteProtocol` 虽然没有直接叫做依赖注入，而是叫做混合协议，但是充分使用了 OC 动态语言的特性，不侵入项目，高度自动化，框架十分轻量，使用非常简单





