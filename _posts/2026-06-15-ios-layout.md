---
title: "iOS 学习第七章：布局"
date: 2026-06-15 00:07:00 +0800
category: iOS
category_slug: ios
description: "理解 Frame、Auto Layout、Safe Area、Stack View 和屏幕适配，建立稳定的页面布局能力。"
---

布局解决的是一个问题：视图应该出现在什么位置，占据多大空间，并且在不同设备、方向、字体和系统环境下保持稳定。

## 1. Frame 布局

Frame 是最直接的布局方式。它用 `x`、`y`、`width`、`height` 描述视图的位置和尺寸。

```objc
UIView *box = [[UIView alloc] initWithFrame:CGRectMake(20, 100, 200, 80)];
box.backgroundColor = UIColor.systemBlueColor;
[self.view addSubview:box];
```

Frame 适合简单、固定、计算明确的布局。它的问题是设备变化后需要手动重新计算。

```objc
- (void)viewDidLayoutSubviews {
    [super viewDidLayoutSubviews];
    self.button.frame = CGRectMake(20, self.view.bounds.size.height - 80, self.view.bounds.size.width - 40, 48);
}
```

如果使用 Frame，通常要在 `viewDidLayoutSubviews` 或 `layoutSubviews` 中根据当前尺寸计算，而不是只在初始化时写死。

## 2. Bounds 和 Center

`frame` 描述视图在父视图坐标系中的位置和尺寸，`bounds` 描述视图自己的内部坐标系，`center` 描述中心点。

```objc
view.frame = CGRectMake(20, 100, 200, 80);
view.center = CGPointMake(160, 200);
```

普通页面布局主要用 `frame`。涉及滚动、绘制、transform 时，`bounds` 和 `center` 会更重要。

## 3. Auto Layout

Auto Layout 用约束描述视图之间的关系，而不是直接写死坐标。

```objc
UIView *box = [[UIView alloc] init];
box.translatesAutoresizingMaskIntoConstraints = NO;
[self.view addSubview:box];

[NSLayoutConstraint activateConstraints:@[
    [box.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor constant:20],
    [box.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor constant:-20],
    [box.topAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.topAnchor constant:20],
    [box.heightAnchor constraintEqualToConstant:80]
]];
```

`translatesAutoresizingMaskIntoConstraints = NO` 很关键。纯代码 Auto Layout 中，如果忘记关掉它，系统会把 autoresizing mask 转成约束，容易产生冲突。

## 4. Safe Area

Safe Area 表示不被刘海、状态栏、Home Indicator、导航栏等遮挡的安全区域。

```objc
[button.bottomAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.bottomAnchor constant:-20].active = YES;
```

底部按钮、顶部内容、全屏页面都应该考虑 Safe Area。不要简单用屏幕高度减固定值处理所有机型。

## 5. Stack View

`UIStackView` 用于线性排列子视图，可以减少大量约束代码。

```objc
UIStackView *stack = [[UIStackView alloc] initWithArrangedSubviews:@[titleLabel, subtitleLabel, button]];
stack.axis = UILayoutConstraintAxisVertical;
stack.spacing = 12;
stack.alignment = UIStackViewAlignmentFill;
stack.distribution = UIStackViewDistributionFill;
```

Stack View 适合表单、纵向信息块、按钮组。复杂网格和高度动态列表不应强行用 Stack View 堆。

## 6. 内容优先级

Auto Layout 中，两个优先级很重要：

- Content Hugging：不想被拉大。
- Compression Resistance：不想被压缩。

```objc
[titleLabel setContentCompressionResistancePriority:UILayoutPriorityRequired forAxis:UILayoutConstraintAxisHorizontal];
[subtitleLabel setContentHuggingPriority:UILayoutPriorityDefaultLow forAxis:UILayoutConstraintAxisHorizontal];
```

当两个 label 横向排列、空间不足时，优先级决定谁被压缩。复杂页面布局问题很多不是约束数量不够，而是优先级没表达清楚。

## 7. 动态高度

列表 cell 常常需要动态高度。核心是让内容约束从顶部贯穿到底部。

```objc
tableView.estimatedRowHeight = 80;
tableView.rowHeight = UITableViewAutomaticDimension;
```

同时 cell 内部要有完整约束。比如最后一个 label 必须约束到底部，否则系统无法推导 cell 高度。

## 8. 横竖屏和尺寸变化

布局不能只考虑一种屏幕。横竖屏、分屏、动态字体、iPad 多窗口都会改变可用空间。

```objc
- (void)viewWillTransitionToSize:(CGSize)size
       withTransitionCoordinator:(id<UIViewControllerTransitionCoordinator>)coordinator {
    [super viewWillTransitionToSize:size withTransitionCoordinator:coordinator];
}
```

大部分适配应交给 Auto Layout，只有在布局策略真正变化时才写额外逻辑。

## 9. 暗色模式与视觉适配

布局不只是位置，也包括视觉环境。颜色应优先使用动态颜色。

```objc
self.view.backgroundColor = UIColor.systemBackgroundColor;
titleLabel.textColor = UIColor.labelColor;
```

不要把所有颜色写死为黑白。系统动态颜色能自动适配亮色和暗色模式。

## 10. 布局掌握标准

应当能做到：

- 区分 `frame`、`bounds`、`center`。
- 能用 Auto Layout 写出稳定约束。
- 知道 Safe Area 的意义。
- 能使用 Stack View 简化线性布局。
- 能处理动态高度和屏幕适配。
- 能定位常见约束冲突和布局错位问题。

布局的核心不是记 API，而是把页面约束关系表达清楚。
