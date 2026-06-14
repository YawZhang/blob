---
title: "iOS 学习第二章：Objective-C 语言"
date: 2026-06-14 00:01:00 +0800
category: iOS
category_slug: ios
description: "掌握类、对象、协议、分类、Block 和消息发送。"
---

Objective-C 的重点不是语法符号，而是对象、消息、协议和运行时思想。它看起来老派，但非常适合理解 iOS 老项目的组织方式。

## 核心能力

- 掌握类、对象、属性、实例方法和类方法。
- 理解继承、多态、协议、分类和扩展。
- 理解 Block 的声明、传递、回调和捕获。
- 理解 `id`、`instancetype`、`SEL`、`IMP`、`nil`、`NULL`、`NSNull`。

## 在 iOS 中为什么重要

UIKit 和大量业务代码都建立在消息发送、代理协议和对象协作之上。只有理解 OC 的语言模型，才能看懂页面跳转、事件回调、网络封装和模块之间的通信。

## 掌握标准

- 能独立写一个 Model、Manager、ViewController 类。
- 能用 Protocol 设计代理回调。
- 能用 Category 给类扩展能力，并知道它不适合存储状态。
- 能解释 `[object message]` 背后的消息发送含义。

