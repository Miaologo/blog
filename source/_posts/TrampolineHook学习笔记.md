---
title: TrampolineHook学习笔记
date: 2020-05-29 14:23:10
tags:
	- iOS
---

# TrampolineHook学习笔记

本文主要是学习一个中心重定向的项目`TrampolineHook`，了解其实现原理和具体细节。

 `TrampolineHook` 是一个让你**不用关注底层架构Calling Convention（因为涉及到汇编），不用关心上下文信息保存、恢复，不用担心引入传统 Swizzle 方案在大型项目中有奇奇怪怪 Crash 问题的中心重定向框架。**

## 基础知识

OC中方法替换普遍是采用`method_exchangeImplementation` 之类的 `runtime` 方法交换 `IMP` 。这种方式不需要考虑调用函数时的上下文保存和恢复的情况。

熟悉汇编的人都知道，编写汇编代码就需要**关注底层架构Calling Convention（因为涉及到汇编），上下文信息保存、恢复**等情况。

简单的场景，

## 常用汇编命令



## 实现原理

### 实现方案



### 具体汇编代码

```asm
#if defined(__arm64__)

.text
.align 14    // 2^14, ARM64 一页大小是16k，
.globl _th_dynamic_page

interceptor:  // data page 开始
.quad 0      // 分配了 8 字节空间，用于保存自定义跳转函数 IMP，（interceptor 函数）

.align 14    // ARM64 一页大小是16k
_th_dynamic_page:   // code page 第二页起始地址

_th_entry:          // 调用 interceptor 和 原 IMP 的代码逻辑

nop
nop
nop
nop
nop

sub x12, lr,   #0x8
sub x12, x12,  #0x4000
mov lr,  x13                 // 恢复 lr， 恢复后的 lr 是返回的地址

ldr x10, [x12]               // x10 保存的是原 imp
// q寄存器入栈保存
stp q0,  q1,   [sp, #-32]!
stp q2,  q3,   [sp, #-32]!
stp q4,  q5,   [sp, #-32]!
stp q6,  q7,   [sp, #-32]!
// lr 和 x 寄存器入栈
stp lr,  x10,  [sp, #-16]!
stp x0,  x1,   [sp, #-16]!
stp x2,  x3,   [sp, #-16]!
stp x4,  x5,   [sp, #-16]!
stp x6,  x7,   [sp, #-16]!
str x8,        [sp, #-16]!

ldr x8,  interceptor     // interceptor 是自定义 IMP 这里是 static void MTMainThreadChecker(id obj, SEL selector)
blr x8                   // 跳转到自定义 IMP 执行 也就是 MTMainThreadChecker
// 恢复lr和x寄存器
ldr x8,        [sp], #16
ldp x6,  x7,   [sp], #16
ldp x4,  x5,   [sp], #16
ldp x2,  x3,   [sp], #16
ldp x0,  x1,   [sp], #16
ldp lr,  x10,  [sp], #16
// 恢复 q 寄存器
ldp q6,  q7,   [sp], #32
ldp q4,  q5,   [sp], #32
ldp q2,  q3,   [sp], #32
ldp q0,  q1,   [sp], #32

br  x10        // x10 保存的是原 imp 执行原 imp

.rept 2032      
// 循环 2032次，重复执行上面的代码逻辑，对应 codePage 的 jumpInstructions[slot] 一个 codePage 可以支持 2032 个 IMP 替换
mov x13, lr     
// 这是替换的新 IMP 地址，此时 lr 也是返回的地址，需要保存下来 lr 存入寄存器 x13 方便 `_th_entry` 使用
bl _th_entry；
.endr

#endif

```

标签 `_th_entry` 一共包含 5 条 nop 指令 ，和 27 条 代码逻辑指令



### 内存页构建



```objective-c
typedef struct {
    IMP originIMP;
} THDynamicData;

#if defined(__arm64__)
#import "THPageDefinition_arm64.h"
#else
#error x86_64 & arm64e to be supported
#endif

static const size_t THNumberOfDataPerPage = (0x4000 - THDynamicPageInstructionCount * sizeof(int32_t)) / sizeof(THDynamicPageEntryGroup);    //THNumberOfDataPerPage 为 2032

typedef struct {
    union {
        struct {
            IMP redirectFunction;
            int32_t nextAvailableIndex;
        };
        int32_t placeholder[THDynamicPageInstructionCount];
    };
    THDynamicData dynamicData[THNumberOfDataPerPage];
} THDataPage;

typedef struct {
    int32_t fixedInstructions[THDynamicPageInstructionCount];
    THDynamicPageEntryGroup jumpInstructions[THNumberOfDataPerPage];
} THCodePage;

typedef struct {
    THDataPage dataPage;
    THCodePage codePage;
} THDynamicPage;
```







