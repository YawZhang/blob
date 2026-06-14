---
title: "iOS 学习第三章：内存管理"
date: 2026-06-14 00:02:00 +0800
category: iOS
category_slug: ios
description: "理解 ARC、属性语义、循环引用和 Block 捕获。"
---

ARC 已经帮我们处理大部分 retain 和 release，但它不会替我们设计正确的对象关系。iOS 开发里很多泄漏和崩溃都来自引用关系不清楚。

## 核心能力

- 理解 ARC、引用计数、对象持有和对象释放。
- 掌握 `strong`、`weak`、`copy`、`assign` 的使用场景。
- 理解 Block 捕获变量和循环引用。
- 理解 `dealloc`、Autorelease Pool 和临时对象释放。

## 在 iOS 中为什么重要

页面无法释放、网络回调持有 `self`、delegate 用 `strong`、`NSTimer` 持有 target，这些都是常见问题。内存管理学清楚后，才能写出稳定的页面和组件。

## 掌握标准

- 能说明 delegate 为什么通常使用 `weak`。
- 能说明 `NSString` 属性为什么常用 `copy`。
- 能发现并修复 Block 中的 `self` 循环引用。
- 能用 Memory Graph 判断对象为什么没有释放。

