---
title: MacOS 上 NodeJS 和 NPM 的正确安装方式
date: 2020-04-18 18:05:10
tags:
---



# MacOS 上 NodeJS 和 NPM 的正确安装方式

https://kylehe.me/blog/2018/03/27/how-to-install-nodejs-on-macos.html

不同于一般的安装教程，本文主要介绍一下我自己在 macOS 上安装 NodeJS 和 NPM 的最佳实践。本文只介绍单个版本 NodeJS 的安装，多版本共存和切换的需求可以使用 [tj/n](https://github.com/tj/n) 工具。

> 前置知识点：[PATH 环境变量](http://www.linfo.org/path_env_var.html)

不少同学拿到 Mac 后都会选择从官网下载 pkg 安装包来安装 NodeJS，但这种方式可能导致后续使用 NodeJS 时碰到很多不可预料的权限问题。这里介绍我自己的安装经验是如何避免这类问题的。

首先请使用 [Homebrew](https://brew.sh/) 安装 NodeJS。

如果你还不知道 Homebrew 是什么，那么这里简单介绍一下。Homebrew 是 macOS 上很强大的一个包管理器，类似 Ubuntu 上的 `apt-get`。安装方法很简单，从它的官网上复制一段命令执行一下就安装完成咯。Homebrew 不只可以安装 NodeJS ，很多其他的常用软件也都可以用它来安装呢。安装完之后运行一下 `brew -v` 如果能成功就说明装好啦。

那么继续，假设你已经有了 Homebrew。运行 `brew install node` 就开始自动安装了。安装好之后，运行 `node -v` 如果能看到 NodeJS 的版本号就说明安装成功了。再运行 `npm -v` 看看 NPM 是不是也装上了。如果安装得不对，比如只有 node 没有 npm，那么注意去看安装过程中输出的文本，从里面可以找出错误信息，然后针对性的解决。

我们的安装过程还没有结束，要进行最重要的一步。刚才我们安装的 npm 是在 `/usr/local/` 目录下，默认的全局 node_modules 文件夹也是在这个目录下面，后续如果通过 npm 安装全局的命令时会出现一些权限上的问题。我的建议是修改全局 node_modules 文件夹的位置，把它改到我们自己的用户目录下。首先在 home 目录下新建一个给 npm 用的文件夹，比如 `~/.npm-global` ，然后新建一个 `~/.npmrc` 文件，在里面写上 `prefix=~/.npm-global` 。这时我们再使用 `npm i -g webpack` 安装全局命令时，都会被安装到这个文件夹里了，执行 `ls ~/.npm-global/bin` 就能看到刚刚安装的 webpack 了。

最后，我们需要把 `~/.npm-global/bin` 加入到 **PATH** 变量里面，这样执行 webpack 时就能在 `~/.npm-global/bin` 目录下找到它了。一般地，就是在 `~/.bash_profile` 里增加 `export PATH="$HOME/.npm-global/bin:$PATH"` ，如果你使用的是其他 shell ，修改方法也是类似的。修改之后需要重新加载一下配置，执行 `source ~/.bash_profile` 或者新开个 tab 生效。

另外，也可以通过 `npm i -g npm` 把 npm 也安装到我们的用户目录下，这样以后升级 npm 就只要再次执行这个命令就行了，保证用到的 npm 是新版的。注意，修改 PATH 时要把 `~/.npm-global/bin` 放到 $PATH 的前面，让系统先在这个文件夹里找，才能避免先找到 `/usr/local` 下的 npm。

#### 验证node是否安装成功

1.下发命令`npm -v`、`node -v`，能正确显示版本号即表示node安装成功，如果是通过homebrew安装的，下发命令`brew list`会显示node。

#### 安装cnpm

npm由于源服务器在国外下载node包经常超时，有2种方法：

一、直接修改镜像地址；

二、用封装好的cnpm命令

国内镜像

cnpm镜像地址：[http://registry.cnpmjs.org](https://link.jianshu.com?t=http://registry.cnpmjs.org/)

淘宝镜像地址：[https://registry.npm.taobao.org](https://link.jianshu.com?t=https://registry.npm.taobao.org/)

直接设置镜像有3种方法：

1.npm config set key value 命令，设置指定的镜像地址

npm config set registry https://registry.npm.taobao.org

npm info underscore （这个只是为了检验上面的设置命令是否成功，若成功，会返回[指定包]的信息）

2.npm --registry命令

npm --registry https://registry.npm.taobao.org info underscore （npm info underscore依然是为了检验是否设置成功）

3.修改配置文件~/.npmrc （win系统在C:\Users\用户名.npmrc） 加入下面内容

registry = https://registry.npm.taobao.org

其实1,2，3都是修改npm的配置文件.npmrc

cnpm

如果觉得直接修改比较麻烦的话，就用cnpm命令吧，先用

$ npm install -g cnpm --registry=https://registry.npm.taobao.org

安装cnpm包，然后就可以敲cnpm install [name]命令了，很方便~~

如果网络状况不好，或者觉得npm install慢的可以换成国内的镜像试下~~~

或者你直接通过添加 npm 参数 alias 一个新命令:

alias cnpm="npm --registry=https://registry.npm.taobao.org \

--cache=$HOME/.npm/.cache/cnpm \

--disturl=https://npm.taobao.org/dist \

--userconfig=$HOME/.cnpmrc"

\# Or alias it in .bashrc or .zshrc

$ echo '\n#alias for cnpm\nalias cnpm="npm --registry=https://registry.npm.taobao.org \

--cache=$HOME/.npm/.cache/cnpm \

--disturl=https://npm.taobao.org/dist \

--userconfig=$HOME/.cnpmrc"' >> ~/.zshrc && source ~/.zshrc



cnpm使用国内镜像，通过以下命令安装cnpm，cnpm参考：[淘宝 NPM 镜像](http://npm.taobao.org/)

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 三、删除node

如果之前已经安装过node，再次安装会产生冲突，先删除
 1.如果是通过homebrew安装的，下发命令`brew uninstall node`即可。
 2.如果是通过安装包安装的，下发命令`cnpm -v`，可以查看node的安装位置，手动删除安装目录即可：