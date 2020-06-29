---
title: iOS-App瘦身法
date: 2020-06-01 10:31:12
tags:
	- iOS
---

- [一、需求](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-%E4%B8%80%E3%80%81%E9%9C%80%E6%B1%82)
- [二、参考资料](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-%E4%BA%8C%E3%80%81%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)
- [三、Clang Plugin 实验记录](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-%E4%B8%89%E3%80%81ClangPlugin%E5%AE%9E%E9%AA%8C%E8%AE%B0%E5%BD%95)
  - [i. 实验步骤](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-i.%E5%AE%9E%E9%AA%8C%E6%AD%A5%E9%AA%A4)
    - [1. 下载 llvm 源码](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-1.%E4%B8%8B%E8%BD%BDllvm%E6%BA%90%E7%A0%81)
    - [2. build llvm](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-2.buildllvm)
    - [3. 下载 XcodeZombieCode 源码，编译得到 dylib](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-3.%E4%B8%8B%E8%BD%BDXcodeZombieCode%E6%BA%90%E7%A0%81%EF%BC%8C%E7%BC%96%E8%AF%91%E5%BE%97%E5%88%B0dylib)
    - [4. hack Xcode](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-4.hackXcode)
    - [5. 配置 Compiler](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-5.%E9%85%8D%E7%BD%AECompiler)
    - [6. Build](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-6.Build)
    - [7. 处理 jsonpart 文件](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-7.%E5%A4%84%E7%90%86jsonpart%E6%96%87%E4%BB%B6)
  - [ii. 存在的问题](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-ii.%E5%AD%98%E5%9C%A8%E7%9A%84%E9%97%AE%E9%A2%98)
- [四、Clang Tool 实验记录](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-%E5%9B%9B%E3%80%81ClangTool%E5%AE%9E%E9%AA%8C%E8%AE%B0%E5%BD%95)
  - [i. 实验步骤](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-i.%E5%AE%9E%E9%AA%8C%E6%AD%A5%E9%AA%A4.1)
    - [1. 下载 llvm 源码](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-1.%E4%B8%8B%E8%BD%BDllvm%E6%BA%90%E7%A0%81.1)
    - [2. build llvm](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-2.buildllvm.1)
    - [3. 下载 XcodeZombieCode 源码，编译得到 tool](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-3.%E4%B8%8B%E8%BD%BDXcodeZombieCode%E6%BA%90%E7%A0%81%EF%BC%8C%E7%BC%96%E8%AF%91%E5%BE%97%E5%88%B0tool)
    - [4. 安装 xcpretty](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-4.%E5%AE%89%E8%A3%85xcpretty)
    - [5. 编译项目，生成 compile_commands.json](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-5.%E7%BC%96%E8%AF%91%E9%A1%B9%E7%9B%AE%EF%BC%8C%E7%94%9F%E6%88%90compile_commands.json)
    - [6. 使用 XcodeZombieCodeAnalyzer](https://wiki.sankuai.com/pages/viewpage.action?pageId=910275215#V5.9%E5%AE%89%E8%A3%85%E5%8C%85%E7%98%A6%E8%BA%AB-6.%E4%BD%BF%E7%94%A8XcodeZombieCodeAnalyzer)

# 一、需求

[iOS-C-技术需求-安装包瘦身](https://wiki.sankuai.com/pages/viewpage.action?pageId=836391354)

# 二、参考资料

1. [滴滴出行iOS端瘦身实践](http://ppt.geekbang.org/slide/show/858)
2. [基于clang插件的一种iOS包大小瘦身方案](https://mp.weixin.qq.com/s?__biz=MzA3ODg4MDk0Ng==&mid=2651112856&idx=1&sn=b2c74c62a10b4c9a4e7538d1ad7eb739&chksm=844c6c45b33be553ee0ee89097b474c1360eb6ea9fe6a96bbc83dcd43484a272e3ad33f72bbd&mpshare=1&scene=1&srcid=1116DZiPA36sqSdCuNxPwV60&key=a57cdfb69cf49b8c23fe3f1900b9dbe4d1056bc23e998afb73ba0c337446cbf46d153212f7114d0d62fa876eedb5a25edd9fa8f4a26d49c2b30da2b9fa2f94ac109e55c9c057764813e418978a3b0d96&ascene=0&uin=NTgxOTMxOTM1&devicetype=iMac+MacBookPro11%2C2+OSX+OSX+10.12.3+build%2816D32%29&version=12020110&nettype=WIFI&fontScale=100&pass_ticket=QhDr1jKjs8D1GVZGsMimV9ODnJhNlrp%2BtinUJHNA5obnu2LJ8Y%2FSgJFcNCXqo4A4)
3. [CLANG技术分享系列一:编写你的第一个CLANG插件](http://kangwang1988.github.io/tech/2016/10/31/write-your-first-clang-plugin.html)
4. [CLANG技术分享系列二:代码风格检查(A CLANG PLUGIN APPROACH)](http://kangwang1988.github.io/tech/2016/10/31/check-code-style-using-clang-plugin.html)
5. [CLANG技术分享系列三:API有效性检查](http://kangwang1988.github.io/tech/2016/11/01/validate-ios-api-using-clang-plugin.html)
6. [CLANG技术分享系列四:IOS APP无用代码/重复代码分析](http://kangwang1988.github.io/tech/2016/11/01/find-unused-duplicate-code-of-your-app-using-clang-plugin.html)
7. [计算机体系-编译体系漫游](http://blog.mrriddler.com/2017/02/10/%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%BD%93%E7%B3%BB-%E7%BC%96%E8%AF%91%E4%BD%93%E7%B3%BB%E6%BC%AB%E6%B8%B8/)
8. [http://blog.mrriddler.com/2017/02/24/Clang插件-Sherlock/](http://blog.mrriddler.com/2017/02/24/Clang%E6%8F%92%E4%BB%B6-Sherlock/)
9. <https://jonasdevlieghere.com/understanding-the-clang-ast/>
10. [GMTC 上分享滴滴出行 iOS 端瘦身实践的 Slides](http://www.jianshu.com/p/6422a6861562)
11. [深入剖析 iOS 编译 Clang / LLVM 直播 PPT](http://www.jianshu.com/p/1d8d1c079e4e)
12. [https://github.com/ming1016/study/wiki/深入剖析-iOS-编译-Clang---LLVM](https://github.com/ming1016/study/wiki/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90-iOS-%E7%BC%96%E8%AF%91-Clang---LLVM)  这篇讲得太好了
13. <http://railsware.com/blog/2014/02/28/creation-and-using-clang-plugin-with-xcode/>
14. <http://clang.llvm.org/docs/ExternalClangExamples.html>
15. [使用Xcode开发iOS语法检查的Clang插件](http://www.jianshu.com/p/581ef614a1c5)
16. [http://www.njiang.cn/2017/03/03/原创-关于如何用Xcode调试开发clang插件/](http://www.njiang.cn/2017/03/03/%E5%8E%9F%E5%88%9B-%E5%85%B3%E4%BA%8E%E5%A6%82%E4%BD%95%E7%94%A8Xcode%E8%B0%83%E8%AF%95%E5%BC%80%E5%8F%91clang%E6%8F%92%E4%BB%B6/)
17. [http://www.njiang.cn/2017/03/01/原创-关于clang插件的实现原理及实践/](http://www.njiang.cn/2017/03/01/%E5%8E%9F%E5%88%9B-%E5%85%B3%E4%BA%8Eclang%E6%8F%92%E4%BB%B6%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E8%B7%B5/)
18. <https://www.zybuluo.com/pockry/note/566013>
19. <http://clang.llvm.org/docs/index.html>
20. <http://clang.llvm.org/docs/LibTooling.html>
21. <http://clang.llvm.org/docs/HowToSetupToolingForLLVM.html>
22. <http://clang.llvm.org/docs/JSONCompilationDatabase.html>
23. <http://clang.llvm.org/docs/ClangTools.html>
24. <https://kevinaboos.wordpress.com/2013/07/23/clang-tutorial-part-ii-libtooling-example/>
25. <http://jszhujun2010.farbox.com/post/llvm&clang/clang-tutorial-di-er-bu-fen>
26. [https://neyoufan.github.io/2016/12/29/ios/关于LLVM这些东西你必须要知道%20/](https://neyoufan.github.io/2016/12/29/ios/%E5%85%B3%E4%BA%8ELLVM%E8%BF%99%E4%BA%9B%E4%B8%9C%E8%A5%BF%E4%BD%A0%E5%BF%85%E9%A1%BB%E8%A6%81%E7%9F%A5%E9%81%93%20/)
27. <http://clang.llvm.org/docs/LibASTMatchersTutorial.html>

Code：

1. <https://github.com/kangwang1988/XcodeZombieCode>
2. <https://github.com/kangwang1988/XcodeCodingStyle>
3. <https://github.com/kangwang1988/XcodeValidAPI>
4. <https://github.com/ming1016/SMCheckProject>
5. <https://github.com/facebook/facebook-clang-plugins>
6. <https://github.com/farseer002/clangCheckingTool>

# 三、Clang Plugin 实验记录

## i. 实验步骤

### 1. 下载 llvm 源码

cd /opt
sudo mkdir llvm
sudo chown `whoami` llvm
cd llvm
export LLVM_HOME=`pwd`

git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/llvm.git llvm
git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/clang.git llvm/tools/clang
git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/clang-tools-extra.git llvm/tools/clang/tools/extra
git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/compiler-rt.git llvm/projects/compiler-rt

### 2. build llvm

\1. 首先需要安装 cmake：

brew install cmake

\2. cmake & make

mkdir llvm_build
cd llvm_build
cmake ../llvm -DCMAKE_BUILD_TYPE:STRING=Release
make -j`sysctl -n hw.logicalcpu`

### 3. 下载 XcodeZombieCode 源码，编译得到 dylib

git clone [git@github.com](mailto:git@github.com):kangwang1988/XcodeZombieCode.git

或者

git clone <ssh://git@git.sankuai.com/~xuhong04/xcodezombiecode.git>（在前者基础上做了二次开发）

\1. 选择 target 为 ClangZombieCodePlugin，直接编译即可，得到 /opt/llvm/clangplugin/libClangZombieCodePlugin.dylib

\2. 选择 target 为 XcodeZombieCodeAnalyzer，编译得到 XcodeZombieCodeAnalyzer

### 4. hack Xcode

下载 [XcodeHacking.zip](https://raw.githubusercontent.com/kangwang1988/kangwang1988.github.io/master/others/XcodeHacking.zip)，里面是两个文件 HackedClang.xcplugin 和 HackedBuildSystem.xcspec

\1. 将 HackedClang.xcplugin/Contents/Resources/HackedClang.xcspec 中的 NO = ( "-Wno-receiver-is-weak" ) 更改为NO = ()，这是为了避免出现一些不必要的警告

\2. 将 HackedClang.xcplugin 拷贝到 /Applications/Xcode.app/Contents/PlugIns/Xcode3Core.ideplugin/Contents/SharedSupport/Developer/Library/Xcode/Plug-ins/HackedClang.xcplugin

\3. 将 HackedBuildSystem.xcspec 拷贝到 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Library/Xcode/Specifications/HackedBuildSystem.xcspec

### 5. 配置 Compiler

\1. 打开项目，在 Target -> Build Settings -> Build Options 中 Compiler for C/C++/Objective-C 一项中选择 Clang LLVM Trunk，使用前面编译得到的 Clang

\2. 在 OTHER_CFLAGS 中加入 dylib 的路径，以及 plugin 的参数

-Xclang -load -Xclang /opt/llvm/clangplugin/libClangZombieCodePlugin.dylib -Xclang -add-plugin -Xclang ClangZombieCodePlugin -Xclang -plugin-arg-ClangZombieCodePlugin -Xclang $SRCROOT/

### 6. Build

\1. 在项目里新建一个 Analyzer 目录（这是个坑）

\2. build，就能在 Analyzer 目录看到一堆的 jsonpart 文件（TODO：这步有 bug，会报错中断多次，初步判断是 plugin 代码缺陷，需要修复）

### 7. 处理 jsonpart 文件

使用第 3 步得到的 XcodeZombieCodeAnalyzer 处理第 6 步得到的那一堆 jsonpart 文件（TODO：这步奇慢无比，需要查一下为什么这么慢）

./XcodeZombieCodeAnalyzer Analyzer/ WMAppDelegate （第二个参数是 AppDelegate 的名字）

就可以得到 unusedClsMethod.json 等文件。

## ii. 存在的问题

虽然 Clang Plugin 的方式可以满足需求，但是依然存在一些问题

1. 使用稍微麻烦，需要 hack xcode、配置 compiler；还要分两步进行，编译中输出 log，然后分析 log 才能得到最后的结果
2. 无法调试，通过打 log 的方式来 debug 效率太低了

所以，下面将利用 libtooling 做一个可以独立运行的 Clang Tool，方便使用。

# 四、Clang Tool 实验记录

## i. 实验步骤

### 1. 下载 llvm 源码

cd /opt
sudo mkdir llvm
sudo chown `whoami` llvm
cd llvm
export LLVM_HOME=`pwd`

git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/llvm.git llvm
git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/clang.git llvm/tools/clang
git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/clang-tools-extra.git llvm/tools/clang/tools/extra
git clone -b release_39 [git@github.com](mailto:git@github.com):llvm-mirror/compiler-rt.git llvm/projects/compiler-rt

### 2. build llvm

\1. 安装 cmake：

brew install cmake

\2. cmake & make

mkdir llvm_build_rtti

cd llvm_build_rtti

cmake ../llvm -DCMAKE_BUILD_TYPE:STRING=Release -DLLVM_ENABLE_RTTI=ON -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

make -j`sysctl -n hw.logicalcpu`（注：-j 是为了利用多核加速编译）

注：-DLLVM_ENABLE_RTTI=ON 是为了打开 RTTI，否则 clang tools 在 link 时会报错

### 3. 下载 XcodeZombieCode 源码，编译得到 tool

git clone -b libTooling [git@github.com](mailto:git@github.com):kangwang1988/XcodeZombieCode.git

或者

git clone <ssh://git@git.sankuai.com/~xuhong04/xcodezombiecode.git>（在前者基础上做了二次开发）

 

\1. 选择 target 为 XcodeZombieCodeAnalyzer，

​      如果是从 [git@github.com](mailto:git@github.com):kangwang1988/XcodeZombieCode.git 下载的代码，则需要做一些修改

- General - Signing 中选择自己的 Team
- Build Setting - Linking - Other Linker Flags 中删除 -lLLVMDemangle
- Build Setting - Architectures - Base SDK 选择 macOS 10.12
- Build Setting - Deployment - macOS Deployment Target 选择 macOS 10.12
- Build Setting - Search Paths - Always Search User Paths 改为 YES
- Build Setting - Search Paths - Header Search Paths 改为 /opt/llvm/llvm/include /opt/llvm/llvm/tools/clang/include /opt/llvm/llvm_build_rtti/include /opt/llvm/llvm_build_rtti/tools/clang/include
- Build Setting - Search Paths - Library Search Paths 改为 /opt/llvm/llvm_build_rtti/lib
- Build Setting - Apple LLVM 8.1 Language - C++ - C++ Language Dialect 改为 GNU++11
- Build Setting - Build Locations - Per-configuration Build Products Path 改为 /opt/llvm/llvm_build_rtti/bin，二进制文件必须放在这个目录下，在运行的时候才能找到一些 builtin 的文件，这步是深坑

\2. Build，得到 /opt/llvm/llvm_build_rtti/bin/XcodeZombieCodeAnalyzer 二进制文件

### 4. 安装 xcpretty

xcpretty 的作用是生成 compile_commands.json 文件，这个 json 文件里记录了如何编译每个源文件，即记录了编译的命令和参数。

\1. 修改项目中的 Gemfile，添加 

source '<http://ruby.sankuai.com/>' do
gem 'xcpretty', '~> 0.2.8'
end

\2. bundle install

### 5. 编译项目，生成 compile_commands.json

使用 xcodebuild 编译项目，并导出 xcodebuild.log

使用 xcpretty 处理 xcodebuild.log，生成 compile_commands.json

xcodebuild CLANG_ENABLE_MODULE_DEBUGGING=NO -workspace waimai.xcworkspace -scheme waimai -configuration Debug -sdk iphonesimulator clean build \

| tee xcodebuild.log \

| xcpretty --report json-compilation-database --output ./compile_commands.json

### 6. 使用 XcodeZombieCodeAnalyzer

/opt/llvm/llvm_build_rtti/bin/XcodeZombieCodeAnalyzer /Users/Allen/Desktop/XcodeZombieCode/waimai_ios/waimai.xcodeproj /Users/Allen/Desktop/XcodeZombieCode WMAppDelegate

则在 Analyzer 目录下得到存放结果的 json 文件

# 参考文章



[iOS 瘦身之道](https://juejin.im/post/5cdd27d4f265da036902bda5)

