---
title: "iOS 学习第一章：C 语言基础"
date: 2026-06-14
category: iOS
category_slug: ios
description: "理解指针、结构体、函数、头文件和宏定义，为 Objective-C 打底。"
---

C 是 Objective-C 的底座。学 iOS 不需要把 C 学成算法竞赛水平，但必须能读懂项目里出现的类型、指针、结构体、函数、头文件和宏。

这一章的目标很明确：能读懂常见 C 代码，知道它在内存里大概发生了什么，并能理解它为什么会出现在 iOS 项目里。

## 1. 基本类型

C 的基本类型用来描述一块内存里保存的值是什么。iOS 开发里最常见的是整数、浮点数、字符和布尔值。

```c
int age = 20;
float progress = 0.75f;
double price = 99.9;
char level = 'A';
bool isFinished = true;
```

需要掌握的是类型会影响三件事：占用多少内存、能表示什么范围、参与运算时如何被解释。

```c
int count = 5;
double result = count / 2.0;
```

这里 `count / 2.0` 会得到 `2.5`，因为 `2.0` 是浮点数。如果写成 `count / 2`，结果是整数除法，得到 `2`。

## 2. 指针

指针保存的是地址。普通变量保存值，指针变量保存另一个变量在内存中的位置。

```c
int score = 90;
int *p = &score;

printf("%d\n", score);
printf("%d\n", *p);
```

`&score` 表示取 `score` 的地址，`*p` 表示通过地址取出里面的值。

指针最重要的意义是：函数可以通过地址修改外部变量。

```c
void increase(int *value) {
    *value = *value + 1;
}

int count = 10;
increase(&count);
printf("%d\n", count);
```

如果函数参数是 `int value`，传进去的是值的拷贝，函数内部修改不会影响外部变量。如果参数是 `int *value`，传进去的是地址，函数就能改到原变量。

## 3. 数组和字符串

数组是一段连续内存，里面保存多个相同类型的值。

```c
int numbers[3] = {10, 20, 30};

printf("%d\n", numbers[0]);
printf("%d\n", numbers[1]);
```

数组名在很多场景下会退化成指针，所以这两种写法经常等价：

```c
printf("%d\n", numbers[0]);
printf("%d\n", *(numbers + 0));
```

C 字符串本质上是以 `\0` 结尾的字符数组。

```c
char name[] = "Yaw";
printf("%s\n", name);
```

需要注意，C 字符串和 Objective-C 的 `NSString` 不是一回事。

```objective-c
char cName[] = "Yaw";
NSString *ocName = @"Yaw";
```

`char[]` 是 C 字符数组，`NSString *` 是 Objective-C 对象指针。

## 4. 结构体

结构体用来把多个相关字段组合成一个值。iOS 里很多基础类型都是结构体，比如 `CGPoint`、`CGSize`、`CGRect`。

```c
struct Student {
    int age;
    double score;
};

struct Student stu;
stu.age = 20;
stu.score = 95.5;
```

实际项目里通常会用 `typedef` 简化写法。

```c
typedef struct {
    double x;
    double y;
} Point;

Point p = {10.0, 20.0};
printf("%f\n", p.x);
```

结构体是值类型。赋值时会复制整份数据。

```c
Point p1 = {10.0, 20.0};
Point p2 = p1;
p2.x = 99.0;

printf("%f\n", p1.x);
printf("%f\n", p2.x);
```

修改 `p2` 不会影响 `p1`。

## 5. 函数

函数用来把一段逻辑封装成可复用的能力。函数声明说明它怎么被调用，函数实现说明它具体做什么。

```c
int add(int a, int b) {
    return a + b;
}

int result = add(3, 5);
```

函数参数默认是值传递。

```c
void change(int value) {
    value = 100;
}

int number = 10;
change(number);
printf("%d\n", number);
```

输出仍然是 `10`，因为 `change` 改的是参数拷贝。

如果需要让函数返回多个结果，可以用指针参数。

```c
void calculate(int a, int b, int *sum, int *diff) {
    *sum = a + b;
    *diff = a - b;
}

int sum = 0;
int diff = 0;
calculate(10, 3, &sum, &diff);
```

这种写法在 C API 和一些底层库里很常见。

## 6. 头文件和实现文件

`.h` 文件通常放声明，告诉外部“这里有什么可以用”。`.c` 或 `.m` 文件放实现，写具体逻辑。

```c
// MathTool.h
int add(int a, int b);
```

```c
// MathTool.c
#include "MathTool.h"

int add(int a, int b) {
    return a + b;
}
```

使用时引入头文件。

```c
#include "MathTool.h"

int result = add(1, 2);
```

在 Objective-C 中也类似，`.h` 暴露接口，`.m` 写实现。

```objective-c
// User.h
@interface User : NSObject
- (void)login;
@end
```

```objective-c
// User.m
#import "User.h"

@implementation User
- (void)login {
    NSLog(@"login");
}
@end
```

## 7. 宏定义

宏是在编译前做文本替换。它可以定义常量、条件开关，也可以定义简单代码片段。

```c
#define MAX_COUNT 100
#define SCREEN_WIDTH 375
```

使用时：

```c
int count = MAX_COUNT;
```

条件编译常用于区分环境。

```c
#if DEBUG
printf("debug mode\n");
#else
printf("release mode\n");
#endif
```

宏没有类型检查，复杂宏容易制造隐藏问题。能用常量、函数或枚举表达时，不要优先写复杂宏。

```c
#define SQUARE(x) x * x

int value = SQUARE(1 + 2);
```

这个结果不是 `9`，因为会被替换成：

```c
int value = 1 + 2 * 1 + 2;
```

更安全的写法是：

```c
#define SQUARE(x) ((x) * (x))
```

## 8. 枚举

枚举用来表示一组有限状态。iOS 代码里非常常见，比如页面状态、网络状态、业务类型。

```c
typedef enum {
    NetworkStatusUnknown,
    NetworkStatusReachable,
    NetworkStatusNotReachable
} NetworkStatus;

NetworkStatus status = NetworkStatusReachable;
```

Objective-C 项目里更常见的是 `NS_ENUM`。

```objective-c
typedef NS_ENUM(NSInteger, LoginState) {
    LoginStateLoggedOut,
    LoginStateLoggingIn,
    LoginStateLoggedIn
};
```

它比普通 C 枚举更适合 Objective-C，类型信息更清楚，也更利于 Swift 混编。

## 9. 编译和链接

C 代码不是直接运行的。大致过程是：预处理、编译、汇编、链接。

- 预处理：处理 `#include`、`#define`、条件编译。
- 编译：把源代码变成更底层的中间结果。
- 汇编：生成目标文件。
- 链接：把多个目标文件和库组合成最终可执行程序。

这解释了为什么头文件不能乱写实现，也解释了为什么重复定义函数或全局变量会报错。

```c
// Bad.h
int count = 0;
```

如果这个头文件被多个文件引入，可能产生重复定义。更合理的方式是头文件声明，源文件定义。

```c
// Good.h
extern int count;
```

```c
// Good.c
int count = 0;
```

## 10. 和 iOS 的关系

iOS 开发里，C 基础主要服务于三类场景。

第一类是理解系统结构体。

```objective-c
CGRect frame = CGRectMake(0, 0, 100, 50);
view.frame = frame;
```

`CGRect` 是结构体，不是对象。

第二类是理解对象指针。

```objective-c
NSString *name = @"Yaw";
UIView *view = [[UIView alloc] init];
```

这里的 `*` 表示变量保存的是对象地址。Objective-C 对象通常通过指针访问。

第三类是读懂底层 C API。

```objective-c
CFStringRef cfName = CFSTR("Yaw");
NSString *name = (__bridge NSString *)cfName;
```

Core Foundation 中大量类型和函数来自 C 风格 API。理解指针、类型转换和内存语义，会让你读这类代码轻松很多。

## 掌握标准

学完这一章，应当能做到：

- 看到 `int *p = &value`，能说清楚 `p` 保存地址，`*p` 访问值。
- 看到函数参数是指针，能判断它是否可能修改外部变量。
- 看到 `char[]`、`NSString *`，能区分 C 字符串和 Objective-C 字符串。
- 看到结构体，知道它是值类型，赋值会复制数据。
- 看到 `.h` 和 `.m`，知道声明和实现的边界。
- 看到宏和条件编译，知道它发生在编译前。
- 看到 `CGRect`、`CGPoint`、`NS_ENUM`，知道它们和 C 基础的关系。

C 语言这一章不追求写复杂程序，重点是为 Objective-C 打底。只要能读懂这些基础形态，后面学习对象、内存管理、UIKit 和工程化都会顺很多。
