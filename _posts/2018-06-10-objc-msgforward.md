---
title: Objective-C的消息转发
tags: [iOS]
date: 2018-06-10
---
说道消息转发的问题其实早在很久以前就有相关的文章了, 只不过自己没有去尝试过相关内容, 最早在新浪的博客上面看过有人写过关于`NSInvocation`的内容, 博客编写时间是2013年...后来根据面试经历, iOS的底层内容被问的越来越多, 自己也开始了解这些内容, 记录一下消息的转发机制。

# iOS中的消息机制
在讨论消息转发之前, 先回忆一下`Objective-C`的消息机制, 每个对象的方法调用其实都是在给调用对象发送消息, 所有的方法调用最后都走了`objc_msgSend`, 根据目标对象的`isa`指针寻找到该对象所属的类, 在类的方法列表中寻找对应的方法, 如果找不到的话, 就向这个类的父类查找, 一直查到到最上端。

那么问题来了, 如果在寻找方法的过程中, 一直到最后的父类都没有寻找到该方法会出现什么情况呢? 这也就是今天要写的东西的切入点。

# unrecognized selector
我来写了个`button`添加了点击事件, 但是没有实现点击方法
```cpp
_button = [[UIButton alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
_button.backgroundColor = [UIColor redColor];
[_button addTarget:self action:@selector(buttonAction:) forControlEvents:UIControlEventTouchUpInside];
[self.view addSubview:_button];
```
编译器会报一个`Undeclared selector 'buttonAction'`警告, 但是并不影响执行, 这也更说明了`Objective-C`是一门动态语言, 其方法寻找和参数类型确定都是在运行时进行。当我们运行并点击按钮的时候, 如果不做任何处理, 程序就会崩掉:
```cpp
-[ViewController buttonAction:]: unrecognized selector sent to instance 0x10510b090
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[ViewController buttonAction:]: unrecognized selector sent to instance 0x10510b090'
```
其实这是最开始学习的时候会常出现的错误, 当时也只是知道这个是由于方法没有被实现而引起的错误, 并没有深入了解, 直到后来被别人问到: 如何能避免这种情况呢?

![](/img/forwardme.jpg)

那问题又来了如何做到不让他崩溃呢?
![](/img/forwardmsg.jpg)

。。。。。。
![](/img/forwarde.jpg)
为了解决这个问题, 去网上看了一下相关资料, 才发现原来在报`unrecognized selector`之前还是有补救机会的, 在`objc_msgSend`到最后没有找到对应的方法的话则会进行消息转发`_objc_msgForward`这期间需要进行三步:

# 消息转发第一步: + (BOOL)resolveInstanceMethod:(SEL)sel 与 + (BOOL)resolveClassMethod:(SEL)sel
在这两个方法中, 提供一个方法的实现, 即可重启消息发送机制, 消息也会被转发到新的实现上面。
很显然, 第一个是实例方法所对应的解决方案, 第二个是类方法所对应的解决方案, 我们用第一个来测试一下
```cpp
// 直接调用不存在的方法
[self performSelector:@selector(customAction:second:) withObject:@"1" withObject:@"2"];
```
然后在类中添加
```cpp
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSLog(@"%@", NSStringFromSelector(sel));
    // 动态添加一个方法
    class_addMethod(self, sel, class_getMethodImplementation(self, @selector(replaceImpl)), "v@:"); // 无返回值 无参数类型

    return true;
}

- (void)replaceImpl {
    NSLog(@"替换了实现");
}
```
随后发现真的走了`replaceImpl`这个方法!但是这样就相当于抛弃了参数, 我们仍然可以定义个方法把这两个参数接收过来, 比如:
```cpp
- (BOOL)customActionttttt:(NSString *)str second:(NSString *)second {
    NSLog(@"这里也被替换了 %@  %@", str, second);
    return true;
}
```
之后的动态实现方法可以写成这样
```cpp
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    // 如果返回值和参数比较复杂 则获取 Method 然后通过 method_getTypeEncoding 手动获取 types
    Method m = class_getClassMethod(self, @selector(customActionttttt:second:));
    class_addMethod(self, sel, class_getMethodImplementation(self, @selector(customActionttttt:second:)), method_getTypeEncoding(m));

    return true;
}
```
这样就能获取到参数了~

但是这个方法看起来不是很好用, 还需要动态的去添加方法, 往往在我们开发项目中也不是很好处理这种方法, 不过还有别的步骤, 接下来看第二步:

# 消息转发第二步: - (id)forwardingTargetForSelector:(SEL)aSelector
如果没有实现第一步的两个方法的话则会走第二步的这个方法, 很明显, 这个方法让我们返回一个对象实例供他去调用, 如果对象实例中有这个`SEL`则会走该对象的`SEL`的实现, 比如这样
```cpp
// 调用一个不存在的方法
[self performSelector:@selector(targetTestAction)];

// 提供一个TestButton的对象 该对象中有 targetTestAction 的实现
- (id)forwardingTargetForSelector:(SEL)aSelector {
    return [[TestButton alloc] init];
}
```
```cpp
TestButton.h
@interface TestButton : UIButton
- (void)targetTestAction;
@end

TestButton.m
@implementation TestButton
- (void)targetTestAction{
    NSLog(@"我是在TestButton内部实现的呦");
}
@end
```
这种应该是最好理解的转发机制了, 但缺点和第一条一样, 还是需要动态去管理方法, 只不过把他放到了一个对象中

# 消息转发第三步: - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector 与 - (void)forwardInvocation:(NSInvocation *)anInvocation
这一步分成两个部分
```cpp
// 获取方法签名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    Method m = class_getInstanceMethod([self class], @selector(customActionttttt:second:));
    return [NSMethodSignature signatureWithObjCTypes:method_getTypeEncoding(m)];
}

// 获取到 NSInvocation 对象 并实现
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    // 这里获取到的仍然是为实现的SEL
    NSLog(@"%@", NSStringFromSelector(anInvocation.selector));
    [anInvocation setTarget:self];
    [anInvocation setSelector:@selector(customActionttttt:second:)];
    // 手动设置参数
    NSString * arg1 = @"arg1";
    NSString * arg2 = @"arg2";
    // 第一个参数是2 前面的两个参数是target和selector
    [anInvocation setArgument:&arg1 atIndex:2];
    [anInvocation setArgument:&arg2 atIndex:3];
    [anInvocation retainArguments];
    [anInvocation invoke];
}

- (BOOL)customActionttttt:(NSString *)str second:(NSString *)second {
    NSLog(@"这里也被替换了 %@  %@", str, second); // arg1 arg2
    return true;
}
```
首先尝试获取方法签名, 如果方法签名获取成功则返回一个`NSInvocation`对象, 开发者自行实现这个对象即可。

至此所有的解决方法步骤都执行完毕, 如果这些步骤都没有实现的话, 则会执行`- (void)doesNotRecognizeSelector:(SEL)aSelector`但执行到这里也就没救了, 程序一样会崩溃

所以综合看来, 想要自行处理消息转发还是在第三步最合适, 著名的热修复框架`JSPatch`就是利用了这一步骤达到方法覆盖的目的的, `_objc_msgForward`这个方法的类型是一个`IMP`, 我们可以自己手动写一个尝试一下:
```cpp
IMP imp = _objc_msgForward;
Method m = class_getInstanceMethod([self class], @selector(customActionttttt:second:));
class_replaceMethod([self class], @selector(replaceImpl), imp, method_getTypeEncoding(m));
[self performSelector:@selector(replaceImpl)];

//- (void)replaceImpl {
//    NSLog(@"替换了实现");
//}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    NSLog(@"%@", NSStringFromSelector(anInvocation.selector));
    [anInvocation setTarget:self];
    [anInvocation setSelector:@selector(customActionttttt:second:)];
    // 手动设置参数
    NSString * arg1 = @"arg1";
    NSString * arg2 = @"arg2";
    // 第一个参数是2 前面的两个参数是target和selector
    [anInvocation setArgument:&arg1 atIndex:2];
    [anInvocation setArgument:&arg2 atIndex:3];
    [anInvocation retainArguments];
    [anInvocation invoke];
}

- (BOOL)customActionttttt:(NSString *)str second:(NSString *)second {
    NSLog(@"这里也被替换了 %@  %@", str, second); // arg1 arg2
    return true;
}
```
发现同样是可以执行的