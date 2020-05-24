---
title: xcode工程代码清理
date: 2020-05-09 11:06:39
tags:
---



# 自动检测项目中不用的代码

## FUI

为了给APP提速，需要定期清理不用的类
 fui（Find Unused Imports）是开源项目能很好的分析出不再使用的类，准确率非常高，唯一的问题是它处理不了动态库和静态库里提供的类，也处理不了C++的类模板。

使用方法是在Terminal中cd到项目所在的目录，然后执行fui find，然后等上那么几分钟（需要好几分钟甚至需要更长的时间），就可以得到一个列表了。
 由于这个工具还不是100%靠谱，可根据这个列表，在Xcode中手动检查并删除不再用到的类。

[fui的github链接](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fdblock%2Ffui)
 使用

```kotlin
//安装 fui 工具 在终端中执行命令
sudo gem install fui -n /usr/local/bin

fui usage: https://github.com/dblock/fui

到工程目录下，执行 fui find 命令，可以找出所有的没有用到的class文件
```

## Pecker

> 链接文档 https://www.jianshu.com/p/c190e1d72afd

先放上项目的地址Pecker，觉得不错的不妨点点Star。

背景
最近在折腾编译相关的，然后就想能不能写一个检测项目中不用代码的工具，毕竟这也是比较常见的需求，但这并不容易。想了两天并没有太好的思路，因为Swift的语法是很复杂的，包括Protocol和范型，如果自己Parse源代码，然后查找哪些地方使用到它，这绝对是个大工程，想想都可怕。

正好最近看了看sourcekit-lsp，突然就来了思路，下面我会详细的讲一讲。

sourcekit-lsp
SourceKit-LSP is an implementation of the Language Server Protocol (LSP) for Swift and C-based languages. It provides features like code-completion and jump-to-definition to editors that support LSP. SourceKit-LSP is built on top of sourcekitd and clangd for high-fidelity language support, and provides a powerful source code index as well as cross-language support. SourceKit-LSP supports projects that use the Swift Package Manager.

sourcekit-lsp基于Swift和C语言的 Language Server Protocol (LSP) 实现，它提供了代码自动补全和定义跳转。

按照官方的定义，“The Language Server Protocol (LSP) defines the protocol used between an editor or IDE and a language server that provides language features like auto complete, go to definition, find all references etc.（语言服务器协议是一种被用于编辑器或集成开发环境 与 支持比如自动补全，定义跳转，查找所有引用等语言特性的语言服务器之间的一种协议）”。

这样如果你想让某个IDE支持Swift，就只需要集成sourcekit-lsp即可。比如下面这个Xcode提供的功能Jump to Definition或者Find Call Hierarchy等就是依赖这个原理，你个可以通过sourcekit-lsp让其他IDE实现这个功能。



## SwiftSyntax 图片资源清理

> 链接文档 https://www.jianshu.com/p/b129e80f3383

基于**SwiftSyntax**写一个命令行工具检测**Xcode**项目中不用的图片资源

其实已经有一个不错的用Swift写的命令行工具检测不用的图片资源了，就是喵神的[FengNiao](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fonevcat%2FFengNiao)，至于为什么要再写一个呢，主要是为了学习[SwiftSyntax](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapple%2Fswift-syntax)，前段时间我写了篇文章简单的介绍了SwiftSyntax，文章在这里[SwiftSyntax详解](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5dac6d3ef265da5b741514b0)，对SwiftSyntax不熟悉的可以先看看。

项目在这里[UnusedResources](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fwoshiccm%2FUnusedResources)，下面来简单介绍一下原理。



