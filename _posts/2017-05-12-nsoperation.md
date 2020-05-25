---
title: NSOperation
tags: [iOS]
date: 2017-05-12
---
前几天整理了一下关于多线程`GCD`的相关内容, 苹果还有一种高级的多线程处理方式是使用`NSOperation`, 它拥有着和`GCD`同样的特点即不需要手动管理线程的声明周期, 而是让开发者把重点关注在自己的方法处理上, 并且`NSOperation`还有着可以手动控制开始, 取消操作等优点。`NSOperation`完全是`Objective-C`对象的形式, 个人感觉`NSOperation`应该是苹果主推的多线程管理框架, = = 但基本上面试问`GCD`的还是多一点。`NSOperation`的[官方文档](https://developer.apple.com/documentation/foundation/nsoperation)

# 任务单元 NSInvocationOperation 与 NSBlockOperation
> Because the NSOperation class is an abstract class, you do not use it directly but instead subclass or use one of the system-defined subclasses (NSInvocationOperation or NSBlockOperation) to perform the actual task. 

苹果的官方原话, 意思是说`NSOperation`是一个抽象类, 不能直接被使用, 你可以使用它的子类`NSInvocationOperation`和`NSBlockOperation`。

基本使用方法:
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
运行结果:
```bash
***** 时间00:47:29 ViewController.m 第30行 *****
<NSThread: 0x2804b5cc0>{number = 1, name = main}

***** 时间00:47:29 ViewController.m 第24行 *****
<NSThread: 0x2804b5cc0>{number = 1, name = main}
```
可以看到, 在直接使用`NSInvocationOperation`或`NSBlockOperation`时, 它们都是运行在主线程之上的。
