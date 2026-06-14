---
title: "iOS 学习第十二章：架构"
date: 2026-06-15 00:12:00 +0800
category: iOS
category_slug: ios
description: "梳理 iOS 架构中的 MVC、MVVM、Service、Router、模块拆分和 ViewController 瘦身。"
---

架构不是为了把项目写得复杂，而是为了让业务增长后仍然能修改、能测试、能定位问题。

iOS 项目最容易失控的地方是 ViewController。页面逻辑、网络请求、数据转换、跳转、埋点、权限判断全部放进去，短期很快，长期很难维护。

## 1. MVC

MVC 是 iOS 最基础的结构：

- Model：数据和业务状态。
- View：界面展示。
- Controller：协调 Model 和 View。

```objc
@interface YWUser : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;

@end
```

```objc
@interface YWProfileView : UIView

- (void)renderWithUser:(YWUser *)user;

@end
```

```objc
@interface YWProfileViewController : UIViewController

@property (nonatomic, strong) YWUser *user;
@property (nonatomic, strong) YWProfileView *profileView;

@end
```

MVC 本身没有问题，问题通常出在 Controller 承担过多职责。

## 2. ViewController 的合理职责

ViewController 适合做：

- 页面生命周期管理。
- View 创建和布局。
- 用户事件转发。
- 页面状态协调。
- 导航跳转入口。

ViewController 不适合做：

- 大量网络请求细节。
- 复杂数据清洗。
- 跨页面业务流程。
- 本地缓存实现。
- 大量格式化逻辑。

一个判断标准：如果某段代码离开页面也有意义，它就不应该强依赖 ViewController。

## 3. Service 层

Service 用来封装业务能力，通常负责网络请求、数据转换和业务动作。

```objc
typedef void(^YWUserCompletion)(YWUser *user, NSError *error);

@interface YWUserService : NSObject

- (void)fetchUserWithId:(NSString *)userId completion:(YWUserCompletion)completion;

@end
```

```objc
@implementation YWUserService

- (void)fetchUserWithId:(NSString *)userId completion:(YWUserCompletion)completion {
    NSURL *url = [NSURL URLWithString:[NSString stringWithFormat:@"https://example.com/users/%@", userId]];

    NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithURL:url
                                                             completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        if (error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                completion(nil, error);
            });
            return;
        }

        NSDictionary *json = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
        YWUser *user = [[YWUser alloc] initWithDictionary:json];

        dispatch_async(dispatch_get_main_queue(), ^{
            completion(user, nil);
        });
    }];

    [task resume];
}

@end
```

这样 ViewController 只关心“我要用户数据”，不关心接口细节。

## 4. ViewModel

ViewModel 负责把业务数据转换成界面能直接使用的数据。

```objc
@interface YWUserViewModel : NSObject

@property (nonatomic, copy, readonly) NSString *displayName;
@property (nonatomic, copy, readonly) NSString *ageText;

- (instancetype)initWithUser:(YWUser *)user;

@end

@implementation YWUserViewModel

- (instancetype)initWithUser:(YWUser *)user {
    self = [super init];
    if (self) {
        _displayName = user.name.length > 0 ? user.name : @"未命名";
        _ageText = [NSString stringWithFormat:@"%ld 岁", (long)user.age];
    }
    return self;
}

@end
```

ViewModel 的价值是减少 ViewController 和 View 里的格式化逻辑。

## 5. MVVM

MVVM 可以理解为：

- Model：原始数据。
- View：展示和事件。
- ViewModel：页面展示状态和交互逻辑。
- ViewController：连接 View 和 ViewModel。

Objective-C 项目中不一定要追求复杂绑定。简单 MVVM 也可以只做输入输出方法。

```objc
@interface YWProfileViewModel : NSObject

@property (nonatomic, copy, readonly) NSString *title;

- (void)loadWithCompletion:(void(^)(NSError *error))completion;

@end
```

重点不是命名为 MVVM，而是把页面展示状态从 ViewController 中拆出来。

## 6. Router

Router 用来集中处理页面跳转。

```objc
@interface YWRouter : NSObject

+ (void)openProfileFrom:(UIViewController *)source userId:(NSString *)userId;

@end

@implementation YWRouter

+ (void)openProfileFrom:(UIViewController *)source userId:(NSString *)userId {
    YWProfileViewController *controller = [[YWProfileViewController alloc] initWithUserId:userId];
    [source.navigationController pushViewController:controller animated:YES];
}

@end
```

当项目页面多、跳转复杂时，Router 能减少页面之间的直接依赖。

## 7. Manager

Manager 常用于封装跨业务的通用能力，例如登录态、定位、图片缓存、播放状态。

```objc
@interface YWSessionManager : NSObject

@property (nonatomic, copy, readonly) NSString *token;

+ (instancetype)sharedManager;
- (BOOL)isLoggedIn;
- (void)logout;

@end
```

Manager 不应该变成万能类。一个 Manager 应该有清晰边界，只管理一个方向的能力。

## 8. 模块拆分

模块拆分的目标是降低依赖。

常见拆分方式：

- 按业务：登录、首页、订单、个人中心。
- 按能力：网络、存储、图片、日志、埋点。
- 按层次：基础库、业务基础层、业务模块、App 壳。

判断模块边界时可以问：

- 这个模块能否独立编译。
- 这个模块是否依赖了太多上层业务。
- 这个模块对外暴露的接口是否稳定。
- 修改这个模块会影响多少页面。

## 9. 胖 ViewController 拆分

拆分 ViewController 可以从几个方向开始：

- View 相关代码放到自定义 View。
- 网络请求放到 Service。
- 展示格式化放到 ViewModel。
- 跳转放到 Router。
- 复杂状态判断放到独立对象。
- 数据源和代理拆成单独类。

UITableView 数据源拆分示例：

```objc
@interface YWArticleDataSource : NSObject <UITableViewDataSource>

@property (nonatomic, copy) NSArray<YWArticle *> *articles;

@end

@implementation YWArticleDataSource

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return self.articles.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell" forIndexPath:indexPath];
    cell.textLabel.text = self.articles[indexPath.row].title;
    return cell;
}

@end
```

这能让页面控制器少关心列表细节。

## 10. 掌握标准

掌握架构，需要能做到：

- 能解释 MVC 中各层职责。
- 能识别 ViewController 过重的原因。
- 能把网络请求、格式化、跳转从页面中拆出去。
- 能设计简单 Service 和 ViewModel。
- 能理解 Router 解决的是页面依赖问题。
- 能判断 Manager 是否职责过大。
- 能按业务或能力拆分模块。
- 能让代码结构服务于长期维护，而不是只追求命名模式。
