---
title: "iOS 学习第八章：事件与交互"
date: 2026-06-15 00:08:00 +0800
category: iOS
category_slug: ios
description: "掌握 Target-Action、Delegate、手势、Responder Chain、键盘处理和页面传值。"
---

事件与交互描述用户操作如何进入 App，以及页面如何响应这些操作。点击、滑动、输入、返回、传值、键盘弹出，本质上都是事件流。

## 1. Target-Action

按钮点击最常见的是 Target-Action。

```objc
[button addTarget:self
           action:@selector(handleLoginButtonTap:)
 forControlEvents:UIControlEventTouchUpInside];

- (void)handleLoginButtonTap:(UIButton *)sender {
    NSLog(@"tap");
}
```

Target 是接收事件的对象，Action 是被调用的方法，Control Event 是触发时机。它适合一对一、明确的控件事件。

## 2. Delegate

Delegate 用来把对象内部发生的事件交给外部处理。它适合一对一回调。

```objc
@interface LoginView : UIView
@property (nonatomic, weak) id<LoginViewDelegate> delegate;
@end

@protocol LoginViewDelegate <NSObject>
- (void)loginViewDidTapSubmit:(LoginView *)view;
@end
```

Delegate 通常用 `weak`，避免 view 和 controller 互相强持有。

## 3. Block 回调

Block 适合轻量回调，尤其是异步结果。

```objc
self.onSubmit = ^{
    NSLog(@"submit");
};
```

使用 Block 要注意循环引用：

```objc
__weak typeof(self) weakSelf = self;
self.onSubmit = ^{
    __strong typeof(weakSelf) self = weakSelf;
    [self submit];
};
```

Block 适合局部事件，不适合把复杂业务链路都塞进闭包。

## 4. 手势识别

`UIGestureRecognizer` 用来处理点击、滑动、长按、缩放等手势。

```objc
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleTap:)];
[self.view addGestureRecognizer:tap];

- (void)handleTap:(UITapGestureRecognizer *)gesture {
    CGPoint point = [gesture locationInView:self.view];
    NSLog(@"%@", NSStringFromCGPoint(point));
}
```

常见手势包括：

- `UITapGestureRecognizer`
- `UIPanGestureRecognizer`
- `UILongPressGestureRecognizer`
- `UIPinchGestureRecognizer`
- `UISwipeGestureRecognizer`

手势冲突是常见问题，比如 scroll view 内部手势和自定义手势同时存在时，需要通过 delegate 协调。

## 5. Responder Chain

Responder Chain 是事件传递链。`UIView`、`UIViewController`、`UIWindow`、`UIApplication` 都可以是 responder。

当一个 view 不处理事件时，事件会沿响应链向上传递。

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [super touchesBegan:touches withEvent:event];
}
```

日常开发不常直接重写触摸方法，但理解响应链有助于处理键盘、菜单、手势冲突和事件无法响应的问题。

## 6. 键盘处理

输入页面必须处理键盘遮挡。

```objc
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(handleKeyboardWillShow:)
                                             name:UIKeyboardWillShowNotification
                                           object:nil];

- (void)handleKeyboardWillShow:(NSNotification *)notification {
    CGRect frame = [notification.userInfo[UIKeyboardFrameEndUserInfoKey] CGRectValue];
    NSTimeInterval duration = [notification.userInfo[UIKeyboardAnimationDurationUserInfoKey] doubleValue];
}
```

键盘处理重点：

- 获取键盘最终 frame。
- 跟随键盘动画时间调整布局。
- 页面消失时移除监听。
- 滚动容器使用 `contentInset` 更自然。

## 7. 页面跳转

页面跳转常见方式有 push 和 present。

```objc
DetailViewController *detail = [[DetailViewController alloc] init];
[self.navigationController pushViewController:detail animated:YES];
```

```objc
LoginViewController *login = [[LoginViewController alloc] init];
[self presentViewController:login animated:YES completion:nil];
```

push 表示进入导航栈下一层；present 表示模态展示。两者生命周期表现不同，不应混用。

## 8. 页面传值

正向传值通常通过属性或初始化方法。

```objc
DetailViewController *detail = [[DetailViewController alloc] init];
detail.articleId = article.articleId;
[self.navigationController pushViewController:detail animated:YES];
```

反向传值可以用 delegate、block 或通知。

```objc
detail.onUpdate = ^(NSString *value) {
    NSLog(@"%@", value);
};
```

简单页面可以直接传值；复杂业务更适合通过状态管理、ViewModel 或路由参数统一处理。

## 9. 交互设计边界

交互代码容易膨胀。判断逻辑放在哪里，可以看职责：

- 控件事件：View 或 Controller 接收。
- 页面跳转：Controller 或 Router。
- 数据请求：Service。
- 状态转换：ViewModel。
- 埋点：Tracker。

Controller 可以协调，但不应承载所有细节。

## 10. 掌握标准

应当能做到：

- 理解 Target-Action、Delegate、Block 的适用边界。
- 能处理常见手势和手势冲突。
- 理解 Responder Chain 的基本方向。
- 能处理键盘遮挡和输入页面布局变化。
- 能完成页面跳转和页面传值。
- 能把交互逻辑拆到合适的模块。

事件与交互是页面“活起来”的部分。它的难点不是 API，而是事件流是否清楚、职责是否边界明确。
