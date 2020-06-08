---
title: 设计模式
tags: [iOS]
date: 2017-08-07
---
在以前别人问到我设计模式的时候, 我可能不假思索的回答MVC, MVVM, 单例, 代理, 通知, 工厂等, 后来被别人问到:请你说一下iOS系统中使用到的设计模式, 越多越好, 至少十种以上。这时我有点答不上来了, 因为平时常用到的设计模式也就那么多, 大学的时候接触过也忘记的差不多了, 最近刚好有时间, 整理一下这方面的内容。


为什么要使用设计模式呢, 个人认为设计模式在编程方面最大的用处就是为了解决耦合问题, 但设计模式并不仅仅应用于编程方面, 生活各个方面都会有用到, 可以参考[RUNOOB](https://www.runoob.com/design-pattern/design-pattern-intro.html)上对设计模式的介绍。设计模式一般分为三个大类, 分别是: 创建型模式（Creational Patterns）、结构型模式（Structural Patterns）、行为型模式（Behavioral Patterns）围绕着这些模式, 一起来看看这些模式在iOS上的实践。

# 工厂模式
工厂模式又分为简单工厂, 工厂方法及抽象工厂, 一个一个来看一下。
## 简单工厂
定义一个接口, 根据参数决定实例化某一个类。
## 工厂方法
定义不同的接口, 根据不同的方法实例化不同的类。
## 抽象工厂
定义不同的接口, 根据接口实现一个工厂类, 再由工厂类去实例化不同的类。

虽然在Cocoa框架中找不到太好的例子来描述, 但我们平时使用的工厂模式还是挺多的, 因为他比较能体现封装的思想, 代码的可扩展性比较好。
例子可以[参考](https://www.jianshu.com/p/6b302c7fe987)这个的打印
```cpp
NSLog(@"%@", [[NSArray alloc] class]); // __NSPlacehodlerArray
NSLog(@"%@", [[NSMutableArray alloc] class]); // __NSPlacehodlerArray
NSLog(@"%@", [[[NSArray alloc] init] class]); // __NSArrayI
NSLog(@"%@", [[[NSMutableArray alloc] init] class]); // __NSArrayM
```
`NSPlacehodlerArray`就相当于工厂, 分别构造出可变和不可变的array对象

# 单例模式
一个类只能有一个实例。
这个模式主要用于哪些需要频繁操作的类上面。在iOS中的代表是
```cpp
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    
});
```
单例模式比较重要的一点是如何保证只能初始化唯一实例, 这就涉及到了系统方法私有化, 要保证实例化时的线程安全等, 这个以后会单独写一篇博客说明。

# 建造者模式
通过一个配置对象来实例化一个类。
比如两个类初始化都需要很多参数, 而这些参数有的是必须的有的是可选的这时候怎么办呢?
```cpp
Config *config = [[Config alloc] init];
cofing.name = ...;
config.type = ...;
config.data = ....;

Foo *foo = [Foo createWithConfig: config];
Foo *secondFoo = [Foo createWithSecondConfig: config];
```
将复杂的参数逻辑封装到类中去处理, 并且能通过不同的实例化方法处理不同的参数。

# 原型模式
利用一个已存在的对象, 快速生成和当前对象一样的实例。iOS中的体现为对象的`copy`

# 适配器模式
使两个不兼容的接口或类能够一起工作, 通常需要产生一个新的类。比如内存卡, 读卡器, 电脑三者之间的关系, 读卡器就相当于适配器, 这种模式常用于版本升级, 类库迁移等情况。

# 桥接模式


