---
title: "iOS 学习第九章：网络"
date: 2026-06-15 00:09:00 +0800
category: iOS
category_slug: ios
description: "理解 HTTP、请求封装、NSURLSession、JSON 解析、错误处理和图片加载。"
---

网络层负责让 App 和服务端交换数据。iOS 网络学习的重点不是只会发请求，而是理解请求结构、异步回调、错误边界、数据解析和工程封装。

## 1. HTTP 基础

一次 HTTP 请求通常包含：

- URL：请求地址。
- Method：请求方法，比如 GET、POST。
- Headers：请求头，比如 token、内容类型。
- Body：请求体，POST 常用。
- Status Code：响应状态码。
- Response Body：响应数据。

```text
GET /articles?page=1 HTTP/1.1
Host: example.com
Authorization: Bearer token
```

状态码常见含义：

- 2xx：成功。
- 3xx：重定向。
- 4xx：客户端错误。
- 5xx：服务端错误。

## 2. GET 和 POST

GET 通常用于读取数据，参数经常放在 URL 中。

```text
https://api.example.com/articles?page=1
```

POST 通常用于提交数据，请求体携带 JSON。

```json
{
  "name": "Yaw",
  "age": 20
}
```

实际业务中不能只靠方法名判断安全性，还要看服务端接口约定。

## 3. NSURLSession

`NSURLSession` 是系统网络请求核心类。

```objc
NSURL *url = [NSURL URLWithString:@"https://api.example.com/articles"];
NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithURL:url
                                                          completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    if (error) {
        NSLog(@"%@", error);
        return;
    }

    NSDictionary *json = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
    NSLog(@"%@", json);
}];
[task resume];
```

请求创建后必须调用 `resume`。回调不保证在主线程，更新 UI 要切回主线程。

```objc
dispatch_async(dispatch_get_main_queue(), ^{
    self.titleLabel.text = @"loaded";
});
```

## 4. 请求封装

真实项目不应在每个页面里散写 `NSURLSession`。更常见的是封装 NetworkClient。

```objc
typedef void(^NetworkCompletion)(id _Nullable object, NSError * _Nullable error);

@interface NetworkClient : NSObject
- (void)GET:(NSString *)path completion:(NetworkCompletion)completion;
@end
```

封装层通常负责：

- 拼接 baseURL。
- 设置公共 headers。
- 处理 token。
- 解析 JSON。
- 统一错误模型。
- 切换回主线程。

页面只关心业务结果，不关心底层请求细节。

## 5. JSON 解析

服务端返回的数据通常是 JSON。

```objc
NSError *error = nil;
NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:0 error:&error];
```

解析时要注意：

- 字段可能缺失。
- 类型可能不符合预期。
- 服务端可能返回 `null`，对应 Objective-C 中的 `NSNull`。

```objc
id name = dict[@"name"];
if ([name isKindOfClass:NSString.class]) {
    model.name = name;
}
```

不要直接假设所有字段类型都正确。

## 6. Model 转换

Model 是网络数据进入业务层后的结构。

```objc
@interface Article : NSObject
@property (nonatomic, copy) NSString *title;
@property (nonatomic, copy) NSString *articleId;
@end
```

转换时最好集中处理。

```objc
+ (instancetype)modelWithDictionary:(NSDictionary *)dict {
    Article *article = [[Article alloc] init];
    article.title = [dict[@"title"] isKindOfClass:NSString.class] ? dict[@"title"] : @"";
    article.articleId = [dict[@"id"] isKindOfClass:NSString.class] ? dict[@"id"] : @"";
    return article;
}
```

不要让 UI 直接依赖原始 JSON。

## 7. 错误处理

网络错误至少分三类：

- 连接错误：断网、超时、DNS 失败。
- 协议错误：HTTP 状态码异常。
- 业务错误：服务端返回业务 code。

```objc
NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
if (httpResponse.statusCode < 200 || httpResponse.statusCode >= 300) {
    // Handle HTTP error.
}
```

统一错误模型可以避免页面各自处理错误文案和重试逻辑。

## 8. 取消请求

页面消失后，某些请求应该取消。

```objc
@property (nonatomic, strong) NSURLSessionDataTask *task;

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    [self.task cancel];
}
```

不是所有请求都要取消。提交订单、上传日志、支付状态同步等任务可能需要继续执行。是否取消取决于业务语义。

## 9. 图片加载

图片加载不是简单请求。它涉及缓存、解码、占位图、取消和复用。

列表中加载图片要注意 cell 复用：

```objc
cell.imageView.image = placeholder;
NSString *url = model.imageURL;
[imageLoader load:url completion:^(UIImage *image) {
    cell.imageView.image = image;
}];
```

真实项目中还要校验当前 cell 是否仍然对应同一个 model，避免图片错位。

## 10. 掌握标准

应当能做到：

- 说清楚 HTTP 请求和响应结构。
- 能用 `NSURLSession` 发起基本请求。
- 知道网络回调不一定在主线程。
- 能封装统一网络层。
- 能处理 JSON 类型安全和 `NSNull`。
- 能区分连接错误、HTTP 错误和业务错误。
- 能处理请求取消和图片加载复用问题。

网络层的核心是边界清楚：页面负责展示，网络层负责请求，Model 负责承载业务数据。
