---
title: 逆向iOS
date: 2020-05-15 23:49:39
tags:
	- iOS
---

## iOS逆向流程

1. 砸壳
2. dump出头文件
3. 分析功能界面
4. hopper || iDA 分析伪代码
5. 写hook
6. 打包动态库
7. 注入动态库到APP
8. APP重签名
9. 安装到手机上

### [MonkeyDev](https://github.com/AloneMonkey/MonkeyDev)


```
MonkeyDev`是一个xcode插件
原有iOSOpenDev的升级，非越狱插件开发集成神器！

- 可以使用Xcode开发CaptainHook Tweak、Logos Tweak 和 Command-line Tool，在越狱机器开发插件，这是原来iOSOpenDev功能的迁移和改进。
- 只需拖入一个砸壳应用，自动集成class-dump、restore-symbol、Reveal、Cycript和注入的动态库并重签名安装到非越狱机器。
- 支持调试自己编写的动态库和第三方App
- 支持通过CocoaPods第三方应用集成SDK以及非越狱插件，简单来说就是通过CocoaPods搭建了一个非越狱插件商店。
```

庆哥的github如是说.
`MonkeyDev`解决了上面说到的50%的步骤, 再外加一个动态调试.



## 参考文章

[不懂汇编，如何逆向iOS](https://cloud.tencent.com/developer/article/1173917)

[MonkeyDev](https://github.com/AloneMonkey/MonkeyDev)

[Cycript](http://www.cycript.org/)

[iOS攻防（六）：使用Cycript一窥运行程序的神秘面纱(入门篇)](https://sevencho.github.io/archives/c12f47b1.html)



