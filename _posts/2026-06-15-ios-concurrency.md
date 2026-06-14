---
title: "iOS 学习第十一章：多线程"
date: 2026-06-15 00:11:00 +0800
category: iOS
category_slug: ios
description: "梳理 iOS 多线程、GCD、队列、主线程更新 UI、线程安全和常见异步组织方式。"
---

多线程的目标不是“让代码同时跑起来”这么简单，而是让耗时任务不阻塞界面，同时保证共享数据不会被并发访问破坏。

iOS 开发里最重要的规则：UI 必须在主线程更新，耗时任务应该放到后台线程。

## 1. 主线程

主线程负责处理 UI 绘制、触摸事件、页面跳转和大部分用户交互。

如果在主线程做耗时操作，用户看到的就是卡顿、按钮无响应、滚动掉帧。

```objc
// 错误示例：在主线程执行耗时任务
- (void)viewDidLoad {
    [super viewDidLoad];

    NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://example.com/image.png"]];
    self.imageView.image = [UIImage imageWithData:data];
}
```

这段代码会阻塞页面加载。正确思路是后台加载，主线程更新 UI。

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://example.com/image.png"]];
    UIImage *image = [UIImage imageWithData:data];

    dispatch_async(dispatch_get_main_queue(), ^{
        self.imageView.image = image;
    });
});
```

## 2. GCD

GCD 是 iOS 中最常用的多线程工具。它的核心概念是队列和任务。

- 队列：决定任务怎么执行。
- 任务：需要执行的代码块。

常用队列：

- 主队列：`dispatch_get_main_queue()`。
- 全局并发队列：`dispatch_get_global_queue(...)`。
- 自定义串行队列：一次只执行一个任务。
- 自定义并发队列：可以同时执行多个任务。

```objc
dispatch_queue_t queue = dispatch_queue_create("com.yawzhang.image.decode", DISPATCH_QUEUE_SERIAL);

dispatch_async(queue, ^{
    NSLog(@"decode image");
});
```

## 3. 同步与异步

同步和异步描述的是当前线程是否等待任务完成。

- `dispatch_sync`：提交任务后等待执行完成。
- `dispatch_async`：提交任务后立即返回。

```objc
dispatch_queue_t queue = dispatch_queue_create("com.yawzhang.task", DISPATCH_QUEUE_SERIAL);

dispatch_async(queue, ^{
    NSLog(@"async task");
});

NSLog(@"continue");
```

这里 `continue` 不需要等待 `async task` 完成。

危险写法：

```objc
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"deadlock");
});
```

如果这段代码已经在主线程执行，就会死锁。主线程等待主队列任务完成，而主队列任务又要等主线程空出来。

## 4. 串行队列与并发队列

串行队列一次执行一个任务，适合保护顺序。

```objc
dispatch_queue_t serialQueue = dispatch_queue_create("com.yawzhang.write", DISPATCH_QUEUE_SERIAL);

dispatch_async(serialQueue, ^{
    NSLog(@"write file A");
});

dispatch_async(serialQueue, ^{
    NSLog(@"write file B");
});
```

并发队列可以同时执行多个任务，适合互不依赖的耗时操作。

```objc
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.yawzhang.download", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(concurrentQueue, ^{
    NSLog(@"download image 1");
});

dispatch_async(concurrentQueue, ^{
    NSLog(@"download image 2");
});
```

选择队列时先判断任务之间是否有顺序依赖，有依赖就不要盲目并发。

## 5. dispatch_group

多个异步任务都完成后再执行下一步，可以用 `dispatch_group`。

```objc
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_group_async(group, queue, ^{
    NSLog(@"request user info");
});

dispatch_group_async(group, queue, ^{
    NSLog(@"request order list");
});

dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"reload page");
});
```

典型场景是页面需要多个接口数据，全部完成后统一刷新。

## 6. dispatch_barrier

并发读、独占写可以使用 barrier。

```objc
dispatch_queue_t queue = dispatch_queue_create("com.yawzhang.cache", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(queue, ^{
    NSLog(@"read cache");
});

dispatch_barrier_async(queue, ^{
    NSLog(@"write cache");
});

dispatch_async(queue, ^{
    NSLog(@"read cache again");
});
```

barrier 前面的任务执行完成后，barrier 任务独占执行；它完成后，后面的任务再继续并发。

## 7. NSOperationQueue

`NSOperationQueue` 比 GCD 更适合表达任务对象、依赖关系、取消和并发数量控制。

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
queue.maxConcurrentOperationCount = 2;

NSBlockOperation *download = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"download");
}];

NSBlockOperation *parse = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"parse");
}];

[parse addDependency:download];

[queue addOperation:download];
[queue addOperation:parse];
```

这里 `parse` 会等 `download` 完成后再执行。

## 8. 线程安全

线程安全问题来自多个线程同时读写同一份可变数据。

```objc
@property (nonatomic, strong) NSMutableArray *items;
```

如果多个线程同时修改 `items`，可能出现崩溃或数据错乱。

常见解决方式：

- 把读写都放到同一个串行队列。
- 使用锁保护临界区。
- 尽量使用不可变对象。
- 避免把可变集合暴露给外部直接修改。

串行队列保护示例：

```objc
@interface YWStore ()

@property (nonatomic, strong) NSMutableArray *items;
@property (nonatomic) dispatch_queue_t queue;

@end

@implementation YWStore

- (instancetype)init {
    self = [super init];
    if (self) {
        _items = [NSMutableArray array];
        _queue = dispatch_queue_create("com.yawzhang.store", DISPATCH_QUEUE_SERIAL);
    }
    return self;
}

- (void)addItem:(id)item {
    dispatch_async(self.queue, ^{
        [self.items addObject:item];
    });
}

@end
```

## 9. 异步回调中的 self

异步任务会延长 Block 的生命周期。Block 捕获 `self` 时要注意循环引用和对象释放后的回调。

```objc
__weak typeof(self) weakSelf = self;

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSString *result = @"done";

    dispatch_async(dispatch_get_main_queue(), ^{
        __strong typeof(weakSelf) self = weakSelf;
        if (!self) {
            return;
        }

        self.titleLabel.text = result;
    });
});
```

`weakSelf` 避免 Block 强持有页面，`strongSelf` 保证回调执行期间对象不被释放。

## 10. 掌握标准

掌握多线程，需要能做到：

- 能解释主线程为什么不能做耗时任务。
- 能用 GCD 完成后台任务和主线程 UI 更新。
- 能区分同步、异步、串行、并发。
- 能识别 `dispatch_sync` 主队列死锁。
- 能用 `dispatch_group` 组织多个异步任务。
- 能用串行队列或锁保护共享可变数据。
- 能判断什么时候使用 `NSOperationQueue`。
- 能在异步回调中正确处理 `self`。
