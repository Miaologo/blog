---
title: Githud Hexo搭建Blog
date: 2016-10-09 18:56:21
tags:
---

# Hexo博客搭建

> [原文链接](http://lihei12345.github.io/2016/07/15/hexo-blog-setup/)

WordPress搭建博客的方式不太适应技术记录与分享，搭建复杂，需要自己的主机进行Host。而一般的技术博客，只需要静态网页就能满足需求。目前比较流行的是Hexo与Jekyll，后者的搭建过于复杂，所以目前Hexo较为流行。目前几家做的比较好的团队技术博客：

- [美团点评技术团队](http://tech.meituan.com/)
- 微信移动客户端开发团队: **WeMobileDev**
- [微信阅读团队技术博客](http://wereadteam.github.io/)
- [腾讯Bugly](http://dev.qq.com/)，微信公众号: **腾讯Bugly**
- [百度Hi技术周报](http://baiduhidevios.github.io/)
- [饿了么用户端技术博客](http://eleme.io/mobilists/)
- [移动开发前线](http://mobilefrontier.github.io/)
- 微信公众号：**QQ空间终端开发团队**

# github.io

首先，创建github.io，用来host我们要生成的静态博客。创建的方式非常简单，参考：https://pages.github.com/

# Hexo安装

Hexo是基于Node.js构建的，使用npm进行安装管理非常便捷：https://pages.github.com/。

整体的安装与使用官方文档讲解的非常清晰，参考文档能快速完成搭建工作。在搭建的过程中，有几点需要注意一下：

# 1. 分类与标签

这点与WP这种动态网站不同，不需要额外创建分类与标签，只需要在post的文档顶部配置 `categories` 与 `tags` 即可，hexo在构建静态网页的时候，会自动进行提取归类。例如：

```
---
title: Modern PHP -- 读书笔记
date: 2016-07-14 18:40:28
tags:
    - PHP
categories:
    - Jason
---
```



对于团队技术博客而言，categories可以设置为团队成员的名称，这样就能比较好归纳。

# 2. 主题

## 设置

我采用的主题与微信阅读团队的一致，Next Mist主题。具体的搭建过程可以参考：

- https://github.com/iissnan/hexo-theme-next
- http://theme-next.iissnan.com/getting-started.html

## 评论设置

采用disqus评论系统，设置过程也非常简单，可以参考：[https://github.com/iissnan/hexo-theme-next/wiki/%E8%AE%BE%E7%BD%AE%E5%A4%9A%E8%AF%B4-DISQUS](https://github.com/iissnan/hexo-theme-next/wiki/设置多说-DISQUS)

# 3.部署

我才用的部署方式git，配置与使用都非常简单，一行代码就能编译然后部署博客静态网页：
https://hexo.io/zh-cn/docs/deployment.html

这里有个比较奇怪的问题，在修改post之后，deploy始终无法更新，总是提示 “nothing to commit…”，解决方式：https://github.com/hexojs/hexo/issues/67。调用下面三行命令清除缓存再部署：

```
1.rm -rf .deploy
2.hexo generater
3.hexo deploy
```