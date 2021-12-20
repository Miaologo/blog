title: swift-源码调试
date: 2021-04-16 14:05:01
tags:

	- swift
	- lldb



## 编译环境

具体使用到的软件

写这个博文的时候，作者本次编译时的环境是这样的，仅供参考。

- macOS Catalina 10.15.7
- Xcode Version 12.4
- Visual Studio Code Version: 1.51.1
- cmake version 3.19.1
- Python 3.8.0
- ninja 1.10.2
- sccache 0.2.13

源代码大约3.5G，根据构建设置不同，构建完成在5G~70G之间。

CMake：CMake是用于C和C ++的跨平台构建系统

Ninja：增量构建，可替代Xcode构建，更快Sccache：编译器缓存工具（可选）

### 环境检查

本次编译，我们会用到 `cmake`, `python3`, `ninja`, `sccache`。可以使用如下终端命令检测是否安装：

```
cmake --version # This should be 3.18.1 or higher for macOS
python3 --version
ninja --version
sccache --version
```

如果以上的工具没有安装，可以使用 `Homebrew` 安装。

```
brew install cmake ninja sccache
```

> 编译过程可以不使用 sccache 

## 编译步骤

我们本次编译的是 `Swift 5.3.1 Release` 版本的源码。

### 下载源码

1、创建 `swift-project` 目录，进入该目录下。

```
mkdir -p swift-project/swift
cd swift-project/swift
```

2、clone 源码。

```
git clone --branch swift-5.3.1-RELEASE https://github.com/apple/swift.git
```

3、更新 Swift 相关的库，不然会导致后面编译失败。

 `utils/update-checkout`是一个脚本，可以帮助你一起使用所有独立的git存储库，而不是手动克隆/更新每个git存储库。

确保在 `swift-project/swift` 目录下，执行如下命令。

```
./swift/utils/update-checkout --tag swift-5.3.1-RELEASE --clone
```

4、开始编译源码。

编译`utils/build-script`是一个高级自动化脚本，用于处理配置（通过CMake），构建（通过Ninja或Xcode），缓存（通过Sccache），运行测试等。

可通过`./swift/utils/build-script -h`了解相关的参数。 

`-r`  、`--release-debuginfo`：构建所有内容的RelWithDebInfo变体（默认是None) 

`--debug-swift-stdlib`：构建Swift标准库和SDK覆盖层的Debug变体 

`-l`、`--lldb`： 构建LLDB

确保在 `swift-project/swift` 目录下，执行如下命令。

```
./swift/utils/build-script -r --debug-swift-stdlib --lldb
```



### 遇到的问题

**出现 `use of undeclared identifier 'task_threads` **

```shell
/Users/******/swift-source/llvm-project/llvm/utils/unittest/googletest/src/gtest-port.cc:121:32: error: use of undeclared identifier 'task_threads'
  const kern_return_t status = task_threads(task, &thread_list, &thread_count);
                               ^
1 error generated.
[43/3647][  1%][6.034s] Building CXX object lib/DebugInfo/PDB/CMakeFiles/LLVMDebugInfoPDB.dir/Native/PDBFile.cpp.o
ninja: build stopped: subcommand failed.
ERROR: command terminated with a non-zero exit status 1, aborting
```

经过本地头文件check 发现 Xcode 引用的 maxOS.sdk 中的task.h 文件为空

```shell
➜ /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs ? # la
total 0
drwxr-xr-x  5 ******  staff   160B Nov 30 20:27 DriverKit20.2.sdk
drwxr-xr-x  7 ******  staff   224B Nov 30 20:27 MacOSX.sdk
lrwxr-xr-x  1 ******  staff    10B Feb  9 15:49 MacOSX11.1.sdk -> MacOSX.sdk
```

可以看到 `MacOSX11.1.sdk` 软连接到 `MacOSX.sdk`, 查看 `MacOSX.sdk` 中 `usr/lib/mach/tas.h` 的文件为空，造成链接失败，找不到 `task_threads` 符号

计划直接从网上找一个相关的 SDK 进行安装，后续发现`/Library/Developer/CommandLineTools` 相关的 SDK 文件，直接从里面 copy 文件 `MacOSX11.0.sdk` 到 `Xcode` 的相关目录

`/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs`

```shell
➜ /Library/Developer/CommandLineTools/SDKs ? # la
total 0
lrwxr-xr-x  1 root  wheel    15B Sep 29  2020 MacOSX.sdk -> MacOSX10.15.sdk
drwxr-xr-x  7 root  wheel   224B Aug 10  2020 MacOSX10.14.sdk
drwxr-xr-x  8 root  wheel   256B Sep 29  2020 MacOSX10.15.sdk
drwxr-xr-x  7 root  wheel   224B Aug 14  2020 MacOSX11.0.sdk
```

 copy 操作后

```shell
➜ /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs ? # la
total 0
drwxr-xr-x  5 ******  staff   160B Nov 30 20:27 DriverKit20.2.sdk
drwxr-xr-x  7 ******  staff   224B Nov 30 20:27 MacOSX.sdk
drwxr-xr-x  7 ******  staff   224B Aug 14  2020 MacOSX11.0.sdk
lrwxr-xr-x  1 ******  staff    10B Feb  9 15:49 MacOSX11.1.sdk -> MacOSX.sdk
```

修改 `MacOSX11.1.sdk ` 的软链接，指向 `MacOSX11.0.sdk `

```sh
➜ /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs ? # ln -snf MacOSX11.0.sdk MacOSX11.1.sdk
➜ /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs ? # la
total 0
drwxr-xr-x  5 ******  staff   160B Nov 30 20:27 DriverKit20.2.sdk
drwxr-xr-x  8 ******  staff   256B Apr 21 11:19 MacOSX.sdk
drwxr-xr-x  7 ******  staff   224B Aug 14  2020 MacOSX11.0.sdk
lrwxr-xr-x  1 ******  staff    14B Apr 21 11:21 MacOSX11.1.sdk -> MacOSX11.0.sdk
```



错误：

```php
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/cmath:325:9: error: no member named 'isless' in the global namespace
using ::isless;
      ~~^
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/cmath:326:9: error: no member named 'islessequal' in the global namespace
using ::islessequal;
      ~~^
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/cmath:327:9: error: no member named 'islessgreater' in the global namespace
using ::islessgreater;
      ~~^
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/cmath:328:9: error: no member named 'isunordered' in the global namespace
using ::isunordered;
      ~~^
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/cmath:329:9: error: no member named 'isunordered' in the global namespace
using ::isunordered;
/Users/zaizai/improve/swift-source/llvm-project/llvm/include/llvm/Analysis/ScalarEvolution.h:1696:22: warning: '\c' command does not have a valid word argument [-Wdocumentation]
  /// by a call to \c @llvm.experimental.guard in \p BB.
                   ~~^
1 warning and 13 errors generated.
[42/1341][  3%][56.500s] Building CXX ...MakeFiles/swiftIRGen.dir/GenCast.cpp.o
ninja: build stopped: subcommand failed.
ERROR: command terminated with a non-zero exit status 1, aborting
```

解决方案:
 看报错信息应该是和CommandLine有关，直接删除Developer下的CommandLineTools，使用Xcode中的。

```
sudo rm -rf /Library/Developer/CommandLineTools
sudo xcode-select -s /Applications/Xcode.app
```

1. sudo rm -rf /Library/Developer/CommandLineTools
2. sudo xcode-select -s /Applications/Xcode.app



## 调试

### VSCode

把`swift-project`拖进**VSCode**打开，然后在扩展搜索 `CodeLLDB `并安装

#### 安装依赖 CodeLLDB

我们后面会使用 `Visual Studio Code` 来调试 `Swift`。所以需要装个 `CodeLLDB` 插件来方便调试。

打开 `VSCode` 后，`Command + shift + x` 切换到 `Extensions` 后，在搜索框中搜索 `CodeLLDB` 后点击 `install` 即可。

#### 开始调试

`VSCode` open 我们的 `swift-project` 目录后，点击调试的时候会提示需要配置编译 `launch.json`。选择 `lldb` 来创建`launch.json`，具体的配置如下，要注意路径一致。

其中 `program` 配置后面填写对应的 `swift` 路径

```
/build/Ninja-RelWithDebInfoAssert+stdlib-DebugAssert/swift-macosx-x86_64/bin/swift
```



```
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${workspaceFolder}/build/Ninja-RelWithDebInfoAssert+stdlib-DebugAssert/swift-macosx-x86_64/bin/swift",
            "args": [],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

`Command + shift + D` 切换到 `Run` 后，点击左上秒的 `Debug` 按钮即可。

接下来我们直接`Run`起来后会触发断点，`DEBUG CONSOLE` （调试控制台）内容如下：

```jsx
Launching: /Users/***/swift-project/build/Ninja-RelWithDebInfoAssert+stdlib-DebugAssert/swift-macosx-x86_64/bin/swift
Launched process 27705
Stop reason: exec
```

断点继续，之后`TERMINAL`（终端）会输出如下内容：

```swift
***  You are running Swift's integrated REPL,  ***
***  intended for compiler and stdlib          ***
***  development and testing purposes only.    ***
***  The full REPL is built as part of LLDB.   ***
***  Type ':help' for assistance.              ***
(swift) 
```

此时说 swift 工程运行成功，可以随意编写进行代码测试

#### 调试时变量区 `Local` 没有显示内存处理

找到编译后的LLDB文件目录，把bin目录下的文件全部拷贝到CodeLLDB的bin目录下
LLDB目录：

VSCode 的 bin 目录的地址：

```
/Users/***/.vscode/extensions/vadimcn.vscode-lldb-1.6.2/lldb/lib
```

编译的LLDB文件目地址：

 ```
/Users/***/swift-project/build/Ninja-RelWithDebInfoAssert+stdlib-DebugAssert/lldb-macosx-x86_64/bin/
 ```

> （这里最好先将`vscode 的 lldb/bin`目录下的内容备份，防止出现错误）

#### 同时修改`CodeLLDB`的`lib`文件下面的`liblldb.dylib`文件

复制编译好的`/Users/***/swift-project/build/Ninja-RelWithDebInfoAssert+stdlib-DebugAssert/lldb-macosx-x86_64/bin/` 目录中的 `lldb`文件到`vscode 的 lldb/lib`目录下，删除`vscode 的 lldb/lib`目录下 `liblldb.dylib` 文件（注意备份，防止出现意外case），并将新copy 进入的 `lldb` 并重命名为 `liblldb.dylib` 

lldb 会直接使用自身lib目录下的 `liblldb.dylib` 动态库来进行调试，`liblldb.dylib` 本身包含了 `LLDB.framework` 文件。这里替换改名的目的是为了让 `VSCode` 去bin 中找到编译后的`LLDB.framework`。（此文件替换后也不会显示） 



## 参考文档

[Swift源码项目编译](https://blog.csdn.net/nsf724947554/article/details/110784648)

[Swift源码编译](https://www.jianshu.com/p/270268b0b9d3)

