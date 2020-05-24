---
title: iOS消失发送和转发机制原理
date: 2020-05-18 11:39:01
tags:
	- iOS
---

> [原文链接](http://yulingtianxia.com/blog/2016/06/15/Objective-C-Message-Sending-and-Forwarding/)

## [Call Convention](https://en.wikipedia.org/wiki/Calling_convention#ARM_(A64))

#### ARM (A64)[[edit](https://en.wikipedia.org/w/index.php?title=Calling_convention&action=edit&section=6)]

The 64-bit ARM ([AArch64](https://en.wikipedia.org/wiki/AArch64)) calling convention allocates the 31 general-purpose registers as:[[2\]](https://en.wikipedia.org/wiki/Calling_convention#cite_note-2)

- x30 is the link register (used to return from subroutines)
- x29 is the frame register
- x19 to x29 are callee-saved
- x18 is the 'platform register', used for some operating-system-specific special purpose, or an additional caller-saved register
- x16 and x17 are the Intra-Procedure-call scratch register
- x9 to x15: used to hold local variables (caller saved)
- x8: used to hold indirect return value address
- x0 to x7: used to hold argument values passed to a subroutine, and also hold results returned from a subroutine

The 32nd register, which serves as a stack pointer or as a zero register depending on the context, is referenced either as sp or xzr.

All registers starting with *x* have a corresponding 32-bit register prefixed with *w*. Thus, a 32-bit x0 is called w0.

## iOS 消息发送







## iOS 消息转发

