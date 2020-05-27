---
title: NSOperation
tags: [iOS]
date: 2017-05-12
---
前几天整理了一下关于多线程`GCD`的相关内容, 苹果还有一种高级的多线程处理方式是使用`NSOperation`, 它拥有着和`GCD`同样的特点即不需要手动管理线程的声明周期, 而是让开发者把重点关注在自己的方法处理上, 并且`NSOperation`还有着可以手动控制开始, 取消操作等优点。`NSOperation`完全是`Objective-C`对象的形式, 个人感觉`NSOperation`应该是苹果主推的多线程管理框架, = = 但基本上面试问`GCD`的还是多一点。`NSOperation`的[官方文档](https://developer.apple.com/documentation/foundation/nsoperation)

# NSOperation

> Because the NSOperation class is an abstract class, you do not use it directly but instead subclass or use one of the system-defined subclasses (NSInvocationOperation or NSBlockOperation) to perform the actual task. 

苹果的官方原话, 意思是说`NSOperation`是一个抽象类, 不能直接被使用, 你可以使用它的子类`NSInvocationOperation`和`NSBlockOperation`。

## 基本使用
```cpp
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(operationAction) object:nil];
[operation start];

- (void)operationAction {
    NSLog(@"%@", [NSThread currentThread]);
}

NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"%@", [NSThread currentThread]);
}];
[blockOperation start];
```
一种是`target action`的形式, 一种是`block`回调的形式, 接下来运行一下:
```bash
***** 时间00:47:29 ViewController.m 第30行 *****
<NSThread: 0x2804b5cc0>{number = 1, name = main}

***** 时间00:47:29 ViewController.m 第24行 *****
<NSThread: 0x2804b5cc0>{number = 1, name = main}
```
可以看到, 在直接使用`NSInvocationOperation`或`NSBlockOperation`时, 它们都是运行在主线程之上的。

## NSBlockOperation
但是`NSBlockOperation`稍微有点特殊, 他可以追加任务:
```cpp
NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"%@", [NSThread currentThread]);
}];
[blockOperation addExecutionBlock:^{
    NSLog(@"%@", [NSThread currentThread]);
}];
[blockOperation addExecutionBlock:^{
    NSLog(@"%@", [NSThread currentThread]);
}];
[blockOperation addExecutionBlock:^{
    NSLog(@"%@", [NSThread currentThread]);
}];
[blockOperation start];

// Log
***** 时间15:01:34 ViewController.m 第79行 *****
<NSThread: 0x280ba1100>{number = 6, name = (null)}

***** 时间15:01:34 ViewController.m 第76行 *****
<NSThread: 0x280bce600>{number = 1, name = main}

***** 时间15:01:34 ViewController.m 第82行 *****
<NSThread: 0x280ba1100>{number = 6, name = (null)}

***** 时间15:01:34 ViewController.m 第85行 *****
<NSThread: 0x280bce600>{number = 1, name = main}
```
所以`NSBlockOperation`中的任务不一定是运行在主线程之上的。

# 自定义NSOperation子类
既然`NSOperation`是一个抽象类, 我们不妨自己实现一个, 先看一下它的方法和属性:
```cpp
- (void)start;
- (void)main;
- (void)cancel;
- (void)addDependency:(NSOperation *)op;
- (void)removeDependency:(NSOperation *)op;
- (void)waitUntilFinished API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));

@property (readonly, getter=isCancelled) BOOL cancelled;
@property (readonly, getter=isExecuting) BOOL executing;
@property (readonly, getter=isFinished) BOOL finished;
@property (readonly, getter=isAsynchronous) BOOL asynchronous API_AVAILABLE(macos(10.8), ios(7.0), watchos(2.0), tvos(9.0));
@property (readonly, getter=isReady) BOOL ready;
@property (readonly, copy) NSArray<NSOperation *> *dependencies;
@property NSOperationQueuePriority queuePriority;
@property (nullable, copy) void (^completionBlock)(void) API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
@property NSQualityOfService qualityOfService API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
```
属性方面大多数是任务的状态, 如果需要观察状态我们可以用`kvo`的方式监听一下。方法包含开始取消还有一个`main`, 根据`NSInvocationOperation`与`NSBlockOperation`的使用方法来看, 这个`main`方法很可能是我们需要实现功能的方法, 而`NSInvocationOperation`与`NSBlockOperation`是以回调的形式把`main`的实现留给我们。可以去[官方文档](https://developer.apple.com/documentation/foundation/nsoperation/1407732-main?language=objc)看一下:

> Performs the receiver’s non-concurrent task.

> The default implementation of this method does nothing. You should override this method to perform the desired task. In your implementation, do not invoke super. This method will automatically execute within an autorelease pool provided by NSOperation, so you do not need to create your own autorelease pool block in your implementation.
If you are implementing a concurrent operation, you are not required to override this method but may do so if you plan to call it from your custom start method.

确实是这样的, `main`方法默认不做任何事情, 我们需要重写`main`方法来执行我们自己需要执行的任务, 并且是并发类型, 释放也不需要我们关心。接来下就可以实现一些自定义的方法:

## 自定义任务实现
```cpp
@interface GQOperation : NSOperation
@property (nonatomic, strong) dispatch_block_t taskBlock;
@end

@implementation GQOperation

- (instancetype)init {
    self = [super init];
    if (self) {
        [self addObserver:self forKeyPath:@"cancelled" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
        [self addObserver:self forKeyPath:@"executing" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
        [self addObserver:self forKeyPath:@"finished" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
    }
    return self;
}

- (void)main {
    if (_taskBlock != nil) {
        _taskBlock();
    }
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"keyPath = %@, change = %@", keyPath, change);
}

- (void)dealloc {
    NSLog(@"Operation 释放了");
    [self removeObserver:self forKeyPath:@"cancelled"];
    [self removeObserver:self forKeyPath:@"executing"];
    [self removeObserver:self forKeyPath:@"finished"];
}
@end


// 调用方式
NSLog(@"start %@", [NSThread currentThread]);
GQOperation *operation = [[GQOperation alloc] init];
operation.name = @"com.greg.gqoperation";
__weak typeof(operation) weakOperation = operation;
operation.taskBlock = ^{
    __strong typeof(weakOperation) strongOperation = weakOperation;
    [strongOperation cancel];
    if (!strongOperation.isCancelled) {
        NSLog(@"task %@", [NSThread currentThread]);
        [NSThread sleepForTimeInterval:2];
    }
};
operation.completionBlock = ^{
    NSLog(@"operation 执行结束 %@", [NSThread currentThread]);
};
[operation start];
NSLog(@"end %@", [NSThread currentThread]);
```
打印结果:
```bash
***** 时间11:41:04 ViewController.m 第22行 *****
start <NSThread: 0x280ab2ec0>{number = 1, name = main}

***** 时间11:41:04 GQOperation.m 第33行 *****
keyPath = executing, change = {
    kind = 1;
    new = 1;
    old = 0;
}

***** 时间11:41:04 GQOperation.m 第33行 *****
keyPath = cancelled, change = {
    kind = 1;
    new = 1;
    old = 0;
}

***** 时间11:41:04 GQOperation.m 第33行 *****
keyPath = executing, change = {
    kind = 1;
    new = 0;
    old = 1;
}

***** 时间11:41:04 GQOperation.m 第33行 *****
keyPath = finished, change = {
    kind = 1;
    new = 1;
    old = 0;
}

***** 时间11:41:04 ViewController.m 第36行 *****
operation 执行结束 <NSThread: 0x280ae7700>{number = 5, name = (null)}

***** 时间11:41:04 ViewController.m 第39行 *****
end <NSThread: 0x280ab2ec0>{number = 1, name = main}

***** 时间11:41:04 GQOperation.m 第37行 *****
Operation 释放了
```
这样就可以观察到任务的执行状态及所在线程情况, 还是运行在主线程之上的。

## 依赖
在对剩余的属性及方法进行一波分析:
```cpp
- (void)addDependency:(NSOperation *)op;
- (void)removeDependency:(NSOperation *)op;
- (void)waitUntilFinished API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));

@property (readonly, getter=isAsynchronous) BOOL asynchronous API_AVAILABLE(macos(10.8), ios(7.0), watchos(2.0), tvos(9.0));
@property (readonly, getter=isReady) BOOL ready;
@property (readonly, copy) NSArray<NSOperation *> *dependencies;
@property NSOperationQueuePriority queuePriority;
@property NSQualityOfService qualityOfService API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
```
先看这两个方法`addDependency`, `removeDependency`字面意思是添加依赖和移除依赖, 参数是另一个`NSoperation`我们可以尝试写一下:
```cpp
GQOperation *firstOperation = [[GQOperation alloc] init];
firstOperation.taskBlock = ^{
    NSLog(@"我是第一个operation");
};

GQOperation *secondOperation = [[GQOperation alloc] init];
secondOperation.taskBlock = ^{
    NSLog(@"我是第二个operation");
    [NSThread sleepForTimeInterval:2];
};

GQOperation *thirdOperation = [[GQOperation alloc] init];
thirdOperation.taskBlock = ^{
    NSLog(@"我是第三个operation");
    [NSThread sleepForTimeInterval:3];
};
[firstOperation addDependency:secondOperation];
[secondOperation addDependency:thirdOperation];
[firstOperation start];
```
然后程序崩溃了:
```bash
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** -[GQOperation start]: receiver is not yet ready to execute'
*** 
```
这里大概是说有一个Operation没有准备好运行, 刚巧上面有一个`ready`的属性可以判断试试:
```cpp
if (secondOperation.isReady) {
    [firstOperation start];
}
```
果然, 程序不会在崩溃了, 如果我们将Operation的倒序执行的话:
```cpp
[firstOperation addDependency:secondOperation];
[secondOperation addDependency:thirdOperation];
NSLog(@"second Ready? = %d", secondOperation.isReady);
[thirdOperation start];
NSLog(@"second Ready? = %d", secondOperation.isReady);
[secondOperation start];
NSLog(@"second Ready? = %d", secondOperation.isReady);
if (secondOperation.isReady) {
    [firstOperation start];
}
```
结果:
```cpp
***** 时间14:45:28 ViewController.m 第66行 *****
second Ready? = 0

***** 时间14:45:28 ViewController.m 第61行 *****
我是第三个operation

***** 时间14:45:31 ViewController.m 第68行 *****
second Ready? = 1

***** 时间14:45:31 ViewController.m 第55行 *****
我是第二个operation

***** 时间14:45:33 ViewController.m 第70行 *****
second Ready? = 1

***** 时间14:45:33 ViewController.m 第50行 *****
我是第一个operation
```
可以看到, 这些Operation之间是有相互依赖关系的, 只有依赖的Operation执行完毕之后才能进行另一个Operation, 往往我的业务中也有很多这样的特点, 比如网络请求的依赖等。

还剩下一些优先级参数和同步异步参数会放到`NSOperationQueue`中讲到。

# NSOperationQueue
由于之前的操作都是~~在主线程上进行~~(`NSBlockOperation`有例外, 他的add方法不一定是在主线程, 有可能是其他线程, 由于我只专注了自定义Operation疏忽了对系统的两个对象的使用), 感觉和多线程没有什么关系, 如果要实现多线程编程, 还需要借助另外一个对象`NSOperationQueue`。

## 基础操作
首先我们来创建一个`NSOperationQueue`, 添加两个Operation:
```cpp
NSLog(@"主线程开始");
GQOperation *firstOperation = [[GQOperation alloc] init];
firstOperation.taskBlock = ^{
    NSLog(@"我是自定义Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:1];
    NSLog(@"我是自定义Operation 延迟输出  %@", [NSThread currentThread]);
};

NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:firstOperation];
[queue addOperationWithBlock:^{
    NSLog(@"我是 Block Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:2];
    NSLog(@"我是 Block Operation 延迟输出  %@", [NSThread currentThread]);
}];
NSLog(@"主线程结束");
```
打印结果:
```bash
***** 时间16:03:51 ViewController.m 第75行 *****
主线程开始

***** 时间16:03:51 ViewController.m 第90行 *****
主线程结束

***** 时间16:03:51 ViewController.m 第86行 *****
我是 Block Operation  <NSThread: 0x282617480>{number = 7, name = (null)}

***** 时间16:03:51 ViewController.m 第78行 *****
我是自定义Operation  <NSThread: 0x282623600>{number = 6, name = (null)}

***** 时间16:03:52 ViewController.m 第80行 *****
我是自定义Operation 延迟输出  <NSThread: 0x282623600>{number = 6, name = (null)}

***** 时间16:03:53 ViewController.m 第88行 *****
我是 Block Operation 延迟输出  <NSThread: 0x282617480>{number = 7, name = (null)}
```
可以看到, `NSOperationQueue`默认是一个异步并发的队列, 并且加入到队列中的Operation不用执行`start`方法, 队列会帮我们自动执行。类似于`GCD`的调度组。
## suspended
可以使用`suspended`使队列中的任务暂停:
```cpp
NSLog(@"主线程开始");
GQOperation *firstOperation = [[GQOperation alloc] init];
firstOperation.taskBlock = ^{
    NSLog(@"我是自定义Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:1];
    NSLog(@"我是自定义Operation 延迟输出  %@", [NSThread currentThread]);
};

NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:firstOperation];

queue.suspended = true;

[queue addOperationWithBlock:^{
    NSLog(@"我是 Block Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:2];
    NSLog(@"我是 Block Operation 延迟输出  %@", [NSThread currentThread]);
}];
NSLog(@"主线程结束");

dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    queue.suspended = false;
});
```
打印结果:
```cpp
***** 时间16:46:28 ViewController.m 第75行 *****
主线程开始

***** 时间16:46:28 ViewController.m 第93行 *****
主线程结束

***** 时间16:46:28 ViewController.m 第78行 *****
我是自定义Operation  <NSThread: 0x28247ff40>{number = 6, name = (null)}

***** 时间16:46:29 ViewController.m 第80行 *****
我是自定义Operation 延迟输出  <NSThread: 0x28247ff40>{number = 6, name = (null)}

***** 时间16:46:33 ViewController.m 第89行 *****
我是 Block Operation  <NSThread: 0x28247ff40>{number = 6, name = (null)}

***** 时间16:46:35 ViewController.m 第91行 *****
我是 Block Operation 延迟输出  <NSThread: 0x28247ff40>{number = 6, name = (null)}
```
这样也说明了任务被添加之后就开始了。
## maxConcurrentOperationCount
`maxConcurrentOperationCount`这个属性可以控制最大的并发任务数量, 如果将这个值设置为1则可实现串行:
```cpp
 NSLog(@"主线程开始");
GQOperation *firstOperation = [[GQOperation alloc] init];
firstOperation.taskBlock = ^{
    NSLog(@"我是自定义Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:1];
    NSLog(@"我是自定义Operation 延迟输出  %@", [NSThread currentThread]);
};
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
queue.maxConcurrentOperationCount = 1;
[queue addOperation:firstOperation];
[queue addOperationWithBlock:^{
    NSLog(@"我是 Block Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:2];
    NSLog(@"我是 Block Operation 延迟输出  %@", [NSThread currentThread]);
}];
NSLog(@"主线程结束");
```
可以看到结果是异步的顺序输出:
```cpp
***** 时间17:10:11 ViewController.m 第75行 *****
主线程开始

***** 时间17:10:11 ViewController.m 第91行 *****
主线程结束

***** 时间17:10:11 ViewController.m 第78行 *****
我是自定义Operation  <NSThread: 0x280675b00>{number = 5, name = (null)}

***** 时间17:10:12 ViewController.m 第80行 *****
我是自定义Operation 延迟输出  <NSThread: 0x280675b00>{number = 5, name = (null)}

***** 时间17:10:12 ViewController.m 第87行 *****
我是 Block Operation  <NSThread: 0x280675b00>{number = 5, name = (null)}

***** 时间17:10:14 ViewController.m 第89行 *****
我是 Block Operation 延迟输出  <NSThread: 0x280675b00>{number = 5, name = (null)}
```
## waitUntilFinished
之前说还有一部分`NSOperation`的属性和方法没有分析, 其实他们在队列对象中也有类似的方法, 比如`waitUntilFinished`, 他主要是起到一个阻塞当前线程的效果, 等待到完成之后才会执行其他任务:
```cpp

@property (readonly, getter=isAsynchronous) BOOL asynchronous API_AVAILABLE(macos(10.8), ios(7.0), watchos(2.0), tvos(9.0));
@property (readonly, copy) NSArray<NSOperation *> *dependencies;
@property NSOperationQueuePriority queuePriority;
@property NSQualityOfService qualityOfService API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
```
```cpp
NSLog(@"主线程开始");
GQOperation *firstOperation = [[GQOperation alloc] init];
firstOperation.taskBlock = ^{
    NSLog(@"我是自定义Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:1];
    NSLog(@"我是自定义Operation 延迟输出  %@", [NSThread currentThread]);
};

NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 注释部分和下面这一句是同样的效果
[queue addOperations:@[firstOperation] waitUntilFinished:true];
// [queue addOperation:firstOperation];
// [firstOperation waitUntilFinished];

[queue addOperationWithBlock:^{
    NSLog(@"我是 Block Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:2];
    NSLog(@"我是 Block Operation 延迟输出  %@", [NSThread currentThread]);
}];
NSLog(@"主线程结束");
```
这个得到结果是这样的
```bash
***** 时间17:31:39 ViewController.m 第75行 *****
主线程开始

***** 时间17:31:39 ViewController.m 第78行 *****
我是自定义Operation  <NSThread: 0x28167d940>{number = 6, name = (null)}

***** 时间17:31:40 ViewController.m 第80行 *****
我是自定义Operation 延迟输出  <NSThread: 0x28167d940>{number = 6, name = (null)}

***** 时间17:31:40 ViewController.m 第92行 *****
主线程结束

***** 时间17:31:40 ViewController.m 第88行 *****
我是 Block Operation  <NSThread: 0x28167d940>{number = 6, name = (null)}

***** 时间17:31:42 ViewController.m 第90行 *****
我是 Block Operation 延迟输出  <NSThread: 0x28167d940>{number = 6, name = (null)}
```
当然, 如果我们不需要异步并发的操作, 可以直接使用`NSOperation`对象, 然后调用这些方法。
## queuePriority
`queuePriority`, 默认创建的任务都是Normal级别, 可以验证一下:
```cpp
NSLog(@"主线程开始");
GQOperation *firstOperation = [[GQOperation alloc] init];
// 观察任务的第一条输出 如果指定了高的优先级 总是会先出现自定义Operation的第一条输出 即使自定义的Operation是后添加进去的 如果指定了低的优先级 总是会先出现Block的第一条输出
// firstOperation.queuePriority = NSOperationQueuePriorityHigh;
firstOperation.queuePriority = NSOperationQueuePriorityLow;
firstOperation.taskBlock = ^{
    NSLog(@"我是自定义Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:1];
    NSLog(@"我是自定义Operation 延迟输出  %@", [NSThread currentThread]);
};

NSOperationQueue *queue = [[NSOperationQueue alloc] init];

[queue addOperationWithBlock:^{
    NSLog(@"我是 Block Operation  %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:2];
    NSLog(@"我是 Block Operation 延迟输出  %@", [NSThread currentThread]);
}];
[queue addOperation:firstOperation];
NSLog(@"主线程结束");
```


