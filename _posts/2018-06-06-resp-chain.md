---
title: iOS响应者链
tags: [iOS]
date: 2018-06-06
---
点击一个按钮之后发生了什么?以前基本上没有自己验证过这些东西, 其中有的时候需要拦截事件用的是这样的一段代码:
```cpp
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    if (self.userInteractionEnabled == NO || self.hidden == YES || self.alpha <= 0.01) return nil;
    if ([self pointInside:point withEvent:event] == NO) return nil;
    NSInteger count = self.subviews.count;
    for (NSInteger i = count - 1; i >= 0; i--) {
        UIView *childView = self.subviews[i];
        CGPoint childPoint = [self convertPoint:point toView:childView];
        UIView *resultView = [childView hitTest:childPoint withEvent:event];
        if (resultView) {
            return resultView;
        }
    }
    return self;
}
```
但这是怎么得来的呢?来论证一下。

# 视图是否能响应
首先`userInteractionEnabled = NO`和`hidden = YES`这两种情况是肯定不能响应的, 但是`alpha <= 0.01`这个我倒没试过, 因为实际的需求中也不可能存在这种透明度的视图, 自己试过才发现确实是这样, `alpha`必须大于等于0.02的时候才有响应, 

# 视图是否在点击的范围内
也就是`[self pointInside:point withEvent:event]`这句, 如果返回了`YES`肯定就是在范围内的(也可能有欺骗肉眼的情况, 后面会说道), 这也毋庸置疑。

# 事件的传递
```cpp
// Returns the farthest descendant of the receiver in the view hierarchy (including itself) that contains a specified point.
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;
// Returns a Boolean value indicating whether the receiver contains the specified point.
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;
```
这两个方法其中第一个大致说的意思是会返回一个在点击的点上面的 视图层次中最远的视图(= = 应该就是最上层的视图被我们点击的), 第二个就简单了大致是说这个视图是否包含我们点击的点。

其探究事件传递的话, 主要会用到第一个方法, 我们可以重写第一个方法看看他执行了多少步:
```cpp
// 写一个UIView的分类 利用方法搅拌替换方法 则可在方法之前实现打印
+ (void)load {
    Method hitTest = class_getInstanceMethod([self class], @selector(hitTest:withEvent:));
    Method gqHitTest = class_getInstanceMethod([self class], @selector(gq_hitTest:withEvent:));
    method_exchangeImplementations(hitTest, gqHitTest);
}

- (UIView *)gq_hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    NSLog(@"%@", NSStringFromClass([self class]));
    UIView *view = [self gq_hitTest:point withEvent:event];
    NSLog(@"%@", view);
    return view;
}
```
其中打印的`class`非常多 = =, 如果你使用了系统控件比如`UINavigationController`或`UITabbarController`, 他们的部分视图也会在上面, 就不粘贴了, 其传递过程确实是从`UIWindow`开始, 但`UIWindow`也未必是顶层, 有可能是`UITextEffectsWindow`, 一直到我们所点击的`view`, 所以如果是某个视图重写了这个方法, 倒序遍历也是对的。

所以最初的代码可以解释为:
```cpp
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    // 1.判断当前视图能否接收事件
    if (self.userInteractionEnabled == NO || self.hidden == YES || self.alpha <= 0.01) return nil;
    // 2. 判断点击区域在不在当前视图
    if ([self pointInside:point withEvent:event] == NO) return nil;
    // 3.从后往前遍历自己的子视图，将事件传递给子视图
    NSInteger count = self.subviews.count;
    for (NSInteger i = count - 1; i >= 0; i--) {
        UIView *childView = self.subviews[i];
        // 3.视图坐标转换
        CGPoint childPoint = [self convertPoint:point toView:childView];
        UIView *resultView = [childView hitTest:childPoint withEvent:event];
        if (resultView) {
            // 4.找到最终响应的视图
            return resultView;
        }
    }
    // 5.如果没有响应自己
    return self;
}
```
下面来看一下复杂情况下的情况。

# 平行视图重合情况 
我们可以随意切换两个视图的`userInteractionEnabled`的属性值来观察他们最终的响应情况。

```cpp
_label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
_label.backgroundColor = [UIColor redColor];
_label.userInteractionEnabled = true;
[self.view addSubview:_label];
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(labelAction)];
[_label addGestureRecognizer:tap];

_button = [[UIButton alloc] initWithFrame:CGRectMake(120, 120, 100, 100)];
_button.backgroundColor = [UIColor greenColor];
_button.userInteractionEnabled = true;
[_button addTarget:self action:@selector(buttonAciton) forControlEvents:UIControlEventTouchUpInside];
[self.view addSubview:_button];

// 当Label不可交互Button可交互时, hitTest最终返回UIButton Button是可交互的, 不会影响点击事件
// 当Label为可交互Button可交互时, 重合部分会返回UIButton, Button的方法会执行Label的方法不会执行
// 当Label为可交互Button不可交互时, 重合的部分会返回UILabel, Label的方法会被执行
```

# 层叠视图重合情况
## 如果有不可交互的视图上添加了可交互的视图, 则点击事件不能正常响应, 事件传递会到不可交互那一层截止。

```cpp
_label = [[UILabel alloc] initWithFrame:CGRectMake(20, 20, [UIScreen mainScreen].bounds.size.width - 40, 200)];
_label.backgroundColor = [UIColor greenColor];
_label.userInteractionEnabled = true;
[self.view addSubview:_label];

_button = [[UIButton alloc] initWithFrame:CGRectMake(0, 100, [UIScreen mainScreen].bounds.size.width - 40, 200)];
_button.backgroundColor = [UIColor redColor];
[_button addTarget:self action:@selector(buttonAciton) forControlEvents:UIControlEventTouchUpInside];
[_label addSubview:_button];

// 结果: 点击button会发现hitTest的最后一层是UILabel
```
## 这时我们给`UILabel`设置`userInteractionEnabled = YES`响应的情况如何?

```cpp
_label = [[UILabel alloc] initWithFrame:CGRectMake(20, 20, [UIScreen mainScreen].bounds.size.width - 40, 200)];
_label.backgroundColor = [UIColor greenColor];
_label.userInteractionEnabled = YES;
[self.view addSubview:_label];

_button = [[UIButton alloc] initWithFrame:CGRectMake(0, 100, [UIScreen mainScreen].bounds.size.width - 40, 200)];
_button.backgroundColor = [UIColor redColor];
[_button addTarget:self action:@selector(buttonAciton) forControlEvents:UIControlEventTouchUpInside];
[_label addSubview:_button];

// 结果: 点击Label和Button重合的位置hitTest的最后一层是Button也就是说Button是可以被响应的。
// 点击超出Label区域外的Button, 不能响应, hitTest的最后一层返回是UILabel
```
# 放大点击热区
有些时候我们会用到一写比较小的按钮, 比如我们上架应用前都会有一个用户协议, 那这个协议是可以勾选和不勾选的, 往往这种勾选框在实际的ui中都会很小, 点击很影响用户体验, 如何做到放大点击热区呢, 用到的就是`pointInside: withEvent:`这个方法。比如有两个这样的视图:

```cpp
_label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
_label.backgroundColor = [UIColor redColor];
[self.view addSubview:_label];

_button = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 50, 50)];
_button.backgroundColor = [UIColor greenColor];
[_button addTarget:self action:@selector(buttonAciton) forControlEvents:UIControlEventTouchUpInside];
[self.view addSubview:_button];
_button.center = _label.center;
```

现在`_button`的响应范围就是他的大小`50x50`假如我们想把它放大到`100x100`可以这样做:

```cpp
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    if (self.bounds.size.width < 100) {
        CGRect bounds = self.bounds;
        CGFloat widthDelta = MAX(100 - bounds.size.width, 0);
        CGFloat heightDelta = MAX(100 - bounds.size.height, 0);
        bounds = CGRectInset(bounds, -0.5 * widthDelta, -0.5 * heightDelta);
        return CGRectContainsPoint(bounds, point);
    }
    else {
        CGRect bounds = self.bounds;
        return CGRectContainsPoint(bounds, point);
    }
}
```

这样 所有小于`100x100`的按钮的点击热区全部被放大到了`100x100`, 即可实现不在按钮范围内点击也可以响应的情况(也就上面说过的欺骗肉眼的情况)

# 响应过程
可以通过观察一个`view`的`nextResponder`属性来看他是怎么被响应的

```cpp
- (void)buttonAciton {
    UIResponder *resp = self.button.nextResponder;
    UIResponder *nextResp = resp.nextResponder;
    // 也不知道有多少层 有就遍历
    while (nextResp) {
        NSLog(@"resp = %@", [nextResp class]);
        // 手动更改到下一级
        nextResp = nextResp.nextResponder;
    }
}
```
结果也显而易见, 是从最上层开始向下响应, 一直到`AppDelegate`, 其响应过程大致为(中间去掉了一些不常见的`view`)`ViewController -> UINavigationController -> UITabBarController -> UIWindow -> UIApplication ->AppDelegate`
