---
title: 浅谈Hook
date: 2020-06-21 09:57:18
tags:
	- Hook
	- ASM
---

iOS日常开发中，经常会遇到各种Hook，或者类似的功能logic，如track event，patch等

下面就简单的针对常见的几种 method hook 进行了一些探讨

## iOS 原装功能 

class_addMethod

class_replaceMethod

method_exchangeImplementations

```objective-c
  Method originMethod = class_getInstanceMethod(cls, originSelector);
  Method targetMethod = class_getInstanceMethod(cls, targetSelector);
  BOOL didAddMethod = class_addMethod(cls, originSelector, method_getImplementation(targetMethod), method_getTypeEncoding(targetMethod));
  if (didAddMethod) {
      class_replaceMethod(cls, targetSelector, method_getImplementation(originMethod), method_getTypeEncoding(originMethod));
  } else {
      method_exchangeImplementations(originMethod, targetMethod);
  }
```



## 渐进式





## 完备式





## 参考文章

http://blog.cnbang.net/tech/3219/