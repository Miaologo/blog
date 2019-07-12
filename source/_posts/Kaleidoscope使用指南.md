---
title: Kaleidoscope使用指南
date: 2019-07-12 14:30:00
tags: 工具
---

# Kaleidoscope使用指南

## 简介

Kaleidoscope 是一个非常强大的比对工具，可以十分方便地比对文本、图片、文件夹等内容。搭配上 SourceTree，能够大大提升 Git 的效率。

Kaleidoscope 可以用来：

- 比对任意文字、图片、文件夹
- Code Review 利器，可以比对 Git 中不同 commit、不同分支上的代码
- 快速解决 Git 合并冲突
- …

SourceTree 和 Kaleidoscope 配合起来可以使得 Code Review 和冲突解决变得十分高效。他们配合起来可以做到，在 SourceTree 中选择任意两个 Commit，按下快捷键就唤起 Kaleidoscope 进行比对。对于重视 Code Review 的团队来说，可以说是一个非常能够提升体验的效率工具。我想只有 Code Review 体验的提高，才能让团队里的成员更有 Code Review 的意愿，才能真正推动 Code Review 的进行。

下面我将在本文中分享一下 SourceTree + Kaleidoscope 的配置。

![img](https://punmy.cn/media/15514477375753/15514507029604.jpg)

## 配置步骤

### 1、安装好 SourceTree 和 [Kaleidoscope](https://xclient.info/s/kaleidoscope.html)

### 2、进入 Kaleidoscope 菜单 > Intergration… 安装命令行工具

如下图所示，遵循指示，把`Kaleidoscope`和`Git`两个 Tab 中的命令行工具都安装好。安装完成后，左侧会出现✅标志。
![img](https://punmy.cn/media/15514477375753/15514479975302.jpg)

### 3、打开 SourceTree > Preference > Diff 配置 External Diff / Merge 选项

Diff 和 Merge 的工具都选择 Custom，然后填入如下配置：

Diff Command： `/usr/local/bin/ksdiff`
Arguments： `--partial-changeset --relative-path "$MERGED" -- "$LOCAL" "$REMOTE"`

Merge Command： `/usr/local/bin/ksdiff`
Arguments： `--merge --output "$MERGED" --base "$BASE" -- "$LOCAL" --snapshot "$REMOTE" --snapshot`

> 使用 Custom 配置是因为 SourceTree 对 Kaleidoscope 的原生支持有 Bug

![img](https://punmy.cn/media/15514477375753/15514483067659.jpg)

### 4、配置 Custom Actions 以便快速唤起对比工具

在 Custom Actions 中增加一个配置，配上你希望唤醒对比工具的快捷键，这里我使用 ⇧+⌘+D。
然后在 Script 中填入：`git`，在 Parameters 中填入：`difftool -y -t sourcetree $SHA`。然后就配置完成了。

![img](https://punmy.cn/media/15514477375753/15514488614668.jpg)

## 使用方法

- **比对任意两个 commit 之间的改动**： 按住⌘，选择两个commit，点击刚刚配置的快捷键，即可唤起 Kaleidoscope
- **查看某个文件的改动**：直接右键单击文件，选择 External Diff（也可以对照上面的方法加个快捷键）
- **解决冲突**：右键单击冲突的文件，使用外部工具解决冲突(如下图)

![img](https://punmy.cn/media/15514477375753/15514499170907.jpg)