---
title: "iOS 学习第十五章：系统能力"
date: 2026-06-15 00:15:00 +0800
category: iOS
category_slug: ios
description: "梳理 iOS 常见系统能力，包括相机、相册、定位、推送、WebView、分享、Deep Link 和权限管理。"
---

系统能力是 App 和 iOS 系统交互的部分。相机、相册、定位、推送、分享、WebView、后台任务都属于这个范围。

学习系统能力时要抓住三个问题：怎么申请权限，怎么调用 API，用户拒绝后怎么处理。

## 1. 权限管理

很多系统能力都需要权限。权限不是代码细节，而是产品体验的一部分。

常见权限：

- 相机：`NSCameraUsageDescription`
- 相册：`NSPhotoLibraryUsageDescription`
- 定位：`NSLocationWhenInUseUsageDescription`
- 麦克风：`NSMicrophoneUsageDescription`
- 通知：运行时请求授权

权限文案写在 `Info.plist`：

```xml
<key>NSCameraUsageDescription</key>
<string>需要使用相机拍摄头像</string>
```

权限处理要考虑：

- 首次请求。
- 用户同意。
- 用户拒绝。
- 用户在设置中关闭。
- 功能是否可以降级。

## 2. 相机

简单拍照可以使用 `UIImagePickerController`。

```objc
if ([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
    UIImagePickerController *picker = [[UIImagePickerController alloc] init];
    picker.sourceType = UIImagePickerControllerSourceTypeCamera;
    picker.delegate = self;
    [self presentViewController:picker animated:YES completion:nil];
}
```

接收图片：

```objc
- (void)imagePickerController:(UIImagePickerController *)picker
didFinishPickingMediaWithInfo:(NSDictionary<UIImagePickerControllerInfoKey,id> *)info {
    UIImage *image = info[UIImagePickerControllerOriginalImage];
    self.avatarImageView.image = image;

    [picker dismissViewControllerAnimated:YES completion:nil];
}
```

更复杂的拍摄、扫码、视频录制通常会使用 `AVFoundation`。

## 3. 相册

访问相册前要处理权限。现代项目常使用 Photos 框架。

```objc
[PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
    dispatch_async(dispatch_get_main_queue(), ^{
        if (status == PHAuthorizationStatusAuthorized) {
            NSLog(@"photo authorized");
        } else {
            NSLog(@"photo denied");
        }
    });
}];
```

相册权限从 iOS 14 开始有“有限访问”模式，用户可以只授权部分照片。实际项目需要适配这种状态。

## 4. 定位

定位使用 `CLLocationManager`。

```objc
@interface YWLocationManager () <CLLocationManagerDelegate>

@property (nonatomic, strong) CLLocationManager *manager;

@end

@implementation YWLocationManager

- (void)start {
    self.manager = [[CLLocationManager alloc] init];
    self.manager.delegate = self;
    [self.manager requestWhenInUseAuthorization];
    [self.manager startUpdatingLocation];
}

- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray<CLLocation *> *)locations {
    CLLocation *location = locations.lastObject;
    NSLog(@"lat: %f, lng: %f", location.coordinate.latitude, location.coordinate.longitude);
}

@end
```

定位要注意耗电、精度、前后台权限和用户隐私。

## 5. 推送

推送分为两层：

- 用户授权通知展示。
- APNs 设备 Token 注册。

请求通知权限：

```objc
UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
[center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionSound | UNAuthorizationOptionBadge)
                      completionHandler:^(BOOL granted, NSError *error) {
    NSLog(@"notification granted: %d", granted);
}];
```

推送链路通常涉及 App、APNs、业务服务器三方。客户端负责拿到 device token 并上传给服务器。

## 6. WebView

现代 iOS 使用 `WKWebView`。

```objc
WKWebView *webView = [[WKWebView alloc] initWithFrame:self.view.bounds];
[self.view addSubview:webView];

NSURL *url = [NSURL URLWithString:@"https://example.com"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
[webView loadRequest:request];
```

需要掌握：

- 加载 URL。
- 监听加载进度。
- 处理跳转拦截。
- JS 与 Native 通信。
- Cookie 和登录态。
- 白屏和内存问题。

## 7. 分享

系统分享可以使用 `UIActivityViewController`。

```objc
NSArray *items = @[@"分享内容", [NSURL URLWithString:@"https://yawzhang.github.io/blob/"]];
UIActivityViewController *controller = [[UIActivityViewController alloc] initWithActivityItems:items applicationActivities:nil];
[self presentViewController:controller animated:YES completion:nil];
```

系统分享的优点是简单稳定，缺点是定制能力有限。

## 8. Deep Link

Deep Link 用于从外部打开 App 的某个页面。

常见方式：

- URL Scheme。
- Universal Links。

URL Scheme 示例：

```text
yawzhang://post?id=123
```

App 收到链接后解析路径和参数，再跳转到对应页面。

```objc
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary *)options {
    if ([url.host isEqualToString:@"post"]) {
        NSURLComponents *components = [NSURLComponents componentsWithURL:url resolvingAgainstBaseURL:NO];
        NSLog(@"open post: %@", components.query);
    }
    return YES;
}
```

Universal Links 使用 HTTPS 链接，体验更自然，但配置链路更复杂，需要服务器放置 `apple-app-site-association` 文件。

## 9. 后台任务

iOS 对后台运行限制严格。常见后台能力包括：

- 后台音频。
- 定位更新。
- 后台下载。
- 静默推送。
- BackgroundTasks。

不要假设 App 进入后台后还能一直运行。系统会根据电量、内存、用户行为管理后台时间。

## 10. 权限拒绝后的体验

权限被拒绝后，不应该反复弹窗。更合理的是解释功能受限，并提供去设置页的入口。

```objc
NSURL *url = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
if ([[UIApplication sharedApplication] canOpenURL:url]) {
    [[UIApplication sharedApplication] openURL:url options:@{} completionHandler:nil];
}
```

权限体验要克制。用户还没理解功能价值时就请求权限，拒绝率通常更高。

## 11. 掌握标准

掌握系统能力，需要能做到：

- 能为系统权限配置正确的 `Info.plist` 文案。
- 能处理授权、拒绝、受限等状态。
- 能使用相机和相册完成基础图片选择。
- 能使用定位获取位置并理解耗电和隐私问题。
- 能理解推送的客户端、APNs、服务端链路。
- 能使用 `WKWebView` 加载网页并理解 JS 通信方向。
- 能使用系统分享。
- 能理解 URL Scheme 和 Universal Links 的区别。
- 能设计权限拒绝后的降级体验。
