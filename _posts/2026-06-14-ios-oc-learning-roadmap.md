---
title: "iOS Objective-C 核心学习能力"
date: 2026-06-14 00:04:00 +0800
category: iOS
category_slug: ios
description: "按照技术栈整理 OC / UIKit / 工程化所需掌握的关键能力。"
---

这篇文章按技术栈整理 Objective-C iOS 开发需要掌握的关键能力。它不是时间计划，而是一张能力地图。

## 1. [C 语言基础]({{ "/posts/ios-c-foundation/" | relative_url }})

- 指针、结构体、函数、头文件、宏定义。
- 理解编译过程、内存地址和基础数据表示。

## 2. [Objective-C 语言]({{ "/posts/ios-objective-c-language/" | relative_url }})

- 类与对象、属性与方法、继承与多态。
- 协议、分类、扩展、Block、Runtime 基础。
- `id`、`instancetype`、`nil`、`NULL`、`NSNull`。

## 3. [内存管理]({{ "/posts/ios-memory-management/" | relative_url }})

- ARC、`strong`、`weak`、`copy`、`assign`。
- `dealloc`、Autorelease Pool、循环引用、Block 捕获。

## 4. [Foundation]({{ "/posts/ios-foundation-framework/" | relative_url }})

- `NSString`、`NSArray`、`NSDictionary`、`NSNumber`、`NSDate`、`NSData`。
- `NSURL`、`NSError`、`NSNotificationCenter`、KVC / KVO、`NSTimer`、`NSOperation`。

## 5. UIKit

- `UIView`、`UIViewController`、`UIWindow`。
- `UILabel`、`UIButton`、`UIImageView`、`UITextField`、`UIScrollView`。
- `UITableView`、`UICollectionView`、`UINavigationController`、`UITabBarController`。

## 6. [生命周期]({{ "/posts/ios-lifecycle/" | relative_url }})

- App 生命周期、Scene 生命周期、页面生命周期。
- 前后台切换、页面加载与释放、内存警告处理。

## 7. 布局

- Frame、Auto Layout、Safe Area、Stack View。
- 屏幕适配、横竖屏适配、暗色模式基础。

## 8. 事件与交互

- Target-Action、Delegate、手势、Responder Chain。
- 键盘处理、页面跳转、页面传值。

## 9. 网络

- HTTP、GET / POST、Header / Body、状态码。
- JSON 解析、`NSURLSession`、请求封装、错误处理、图片加载。

## 10. 数据存储

- Model 设计、JSON 转 Model。
- `NSUserDefaults`、文件存储、沙盒目录、SQLite、Core Data、Keychain。

## 11. 多线程

- 主线程、GCD、Serial Queue、Concurrent Queue。
- `dispatch_async`、`dispatch_group`、`NSOperationQueue`、线程安全。

## 12. 架构

- MVC、MVVM、Service 层、Manager 层、ViewModel、Router。
- 模块拆分、胖 ViewController 拆分。

## 13. 调试

- Xcode 断点、LLDB、Console、View Debugger。
- Memory Graph、Instruments、Crash Log、内存泄漏分析、卡顿分析。

## 14. 工程化

- Project / Workspace、Target、Scheme、Build Settings。
- Debug / Release、CocoaPods、Swift Package Manager、静态库 / 动态库。
- 打包、TestFlight、App Store Connect。

## 15. 系统能力

- 相机、相册、定位、推送、WebView、分享、支付。
- Deep Link、Universal Links、后台任务、权限管理。

## 16. Swift 混编

- Swift 基础、OC 调 Swift、Swift 调 OC。
- Bridging Header、Nullability、Swift Package、混编工程结构。
