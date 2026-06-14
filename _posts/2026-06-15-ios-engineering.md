---
title: "iOS 学习第十四章：工程化"
date: 2026-06-15 00:14:00 +0800
category: iOS
category_slug: ios
description: "梳理 iOS 工程化基础，包括 Target、Scheme、Build Settings、依赖管理、打包和发布流程。"
---

工程化解决的是项目如何被组织、构建、依赖、调试、打包和发布。它决定了一个 App 能不能稳定地从开发走到上线。

个人学习阶段也应该理解工程化，因为很多问题不是代码逻辑错误，而是配置、依赖、证书、构建环境错误。

## 1. Project 与 Workspace

`Project` 是 Xcode 工程本身，包含源码、资源、Target、Build Settings 等。

`Workspace` 可以包含多个 Project。使用 CocoaPods 后，通常会打开 `.xcworkspace`，因为它同时包含 App 工程和 Pods 工程。

常见错误是装了 CocoaPods 却继续打开 `.xcodeproj`，导致依赖找不到。

## 2. Target

Target 表示一个构建产物。

常见 Target：

- App Target：主 App。
- Unit Test Target：单元测试。
- UI Test Target：界面测试。
- Extension Target：通知扩展、分享扩展等。
- Framework Target：库。

Target 会决定：

- 编译哪些文件。
- 使用哪些资源。
- 生成什么产物。
- 使用哪些 Build Settings。
- 绑定哪些证书和 Bundle Identifier。

## 3. Scheme

Scheme 描述一次运行或打包使用哪个 Target、哪个配置、哪些测试。

常见用途：

- Debug Scheme：开发调试。
- Release Scheme：发布构建。
- Staging Scheme：测试环境。

如果项目区分开发、测试、生产环境，通常会通过 Scheme 或 Build Configuration 管理。

## 4. Build Configuration

常见配置：

- Debug：调试版本，日志多，优化少。
- Release：发布版本，优化多，调试信息少。

可以用宏区分环境：

```objc
#ifdef DEBUG
static NSString * const YWBaseURL = @"https://api-dev.example.com";
#else
static NSString * const YWBaseURL = @"https://api.example.com";
#endif
```

注意：环境配置要集中管理，不要让接口地址散落在各个文件里。

## 5. Build Settings

Build Settings 控制编译和链接行为。

常见配置：

- `PRODUCT_BUNDLE_IDENTIFIER`：Bundle ID。
- `IPHONEOS_DEPLOYMENT_TARGET`：最低系统版本。
- `HEADER_SEARCH_PATHS`：头文件搜索路径。
- `LIBRARY_SEARCH_PATHS`：库搜索路径。
- `OTHER_LDFLAGS`：链接参数。
- `SWIFT_VERSION`：Swift 版本。

很多“本地能跑、别人不能跑”的问题都和 Build Settings 或本地路径有关。

## 6. Info.plist

`Info.plist` 描述 App 的基础信息和系统能力声明。

常见内容：

- Bundle Display Name。
- Bundle Identifier。
- URL Schemes。
- 权限说明文案。
- 支持的方向。
- App Transport Security 配置。

权限文案示例：

```xml
<key>NSCameraUsageDescription</key>
<string>需要使用相机拍摄头像</string>
```

如果使用相机但没有配置权限说明，App 会崩溃。

## 7. 资源管理

资源包括图片、颜色、字体、Storyboard、Xib、JSON 文件等。

常见注意点：

- 图片放进 Assets，命名清晰。
- 不再使用的资源及时清理。
- 字体文件需要同时加入工程和 Info.plist。
- 大文件不要随意打进 App 包。
- 多语言文案不要写死在代码里。

资源问题常常不会编译报错，而是在运行时才表现为图片为空、字体不生效、文件找不到。

## 8. 依赖管理

常见依赖管理方式：

- CocoaPods。
- Swift Package Manager。
- 手动引入 Framework。
- Git Submodule。

CocoaPods 常用命令：

```bash
pod install
pod update
```

`pod install` 按照 `Podfile.lock` 安装，适合日常协作。`pod update` 会尝试更新版本，影响更大。

SPM 集成在 Xcode 里，适合 Swift 包和现代库管理。

## 9. 静态库与动态库

静态库会在链接时合入最终可执行文件。动态库在运行时加载。

简单理解：

- 静态库：打包进 App，启动时少一层动态加载。
- 动态库：模块独立，但会带来加载和签名管理问题。

iOS 项目中使用三方库时，不一定要一开始深入二进制细节，但要知道链接错误通常和库、符号、架构有关。

## 10. 证书与签名

iOS App 必须经过签名才能安装到真机或发布。

核心概念：

- Certificate：证明开发者身份。
- App ID：标识 App。
- Provisioning Profile：把证书、设备、App ID 绑定在一起。
- Entitlements：声明 App 能使用哪些系统能力。

常见问题：

- Bundle ID 不匹配。
- Profile 过期。
- 设备没有加入开发 Profile。
- Capability 开了但 Profile 未更新。

## 11. 打包与发布

发布链路通常是：

1. 选择 Release 配置。
2. Archive。
3. 上传 App Store Connect。
4. TestFlight 测试。
5. 提交审核。
6. 发布。

发布前要检查：

- 版本号和构建号。
- dSYM 是否上传。
- 隐私权限文案。
- 图标和启动图。
- 第三方 SDK 合规。
- 崩溃和核心流程测试。

## 12. 掌握标准

掌握工程化，需要能做到：

- 能区分 Project、Workspace、Target、Scheme。
- 能理解 Debug 和 Release 的区别。
- 能定位常见 Build Settings 问题。
- 能配置基础权限和 Info.plist。
- 能正确使用 CocoaPods 或 SPM。
- 能理解静态库、动态库和链接错误的大致关系。
- 能解释证书、Profile、Bundle ID 的关系。
- 能完成基本打包、上传和 TestFlight 流程。
