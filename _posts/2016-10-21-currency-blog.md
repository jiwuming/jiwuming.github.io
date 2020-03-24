---
title: 货币计算
tags: iOS
date: 2016-10-21
---
由于最近接触的项目是 P2P 项目, 所以涉及到很多价格显示, 偶尔也会有一些前端的小数计算. 这时候如果用 float 或者 double 直接计算的话, 会导致浮点数不精确的问题. 解决办法就是用 NSDecimalNumber 这个类提供的方法去进行计算, 记录一下使用过程<!-- more -->

首先, 使用一个类先看初始化方法:

```mm
- (instancetype)initWithMantissa:(unsigned long long)mantissa exponent:(short)exponent isNegative:(BOOL)flag;
- (instancetype)initWithDecimal:(NSDecimal)dcm NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithString:(nullable NSString *)numberValue;
- (instancetype)initWithString:(nullable NSString *)numberValue locale:(nullable id)locale;

+ (NSDecimalNumber *)decimalNumberWithMantissa:(unsigned long long)mantissa exponent:(short)exponent isNegative:(BOOL)flag;
+ (NSDecimalNumber *)decimalNumberWithDecimal:(NSDecimal)dcm;
+ (NSDecimalNumber *)decimalNumberWithString:(nullable NSString *)numberValue;
+ (NSDecimalNumber *)decimalNumberWithString:(nullable NSString *)numberValue locale:(nullable id)locale;
```

恩 这里是一些实例初始化和类初始化方法, 写法很相似, 但是这两种初始化方法中的后三个还好, 第一个是个什么鬼? (其实是英语不好) 由于在 Xcode 中 command 点击查看 DecimalNumber 的 API 时没有注释, 我就去翻了苹果的文档, 他是这么举例的:
> The arguments express a number in a kind of scientific notation that requires the mantissa to be an integer. So, for example, if the number to be represented is –12.345, it is expressed as 12345x10^–3—mantissa is 12345; exponent is –3; and isNegative is YES, as illustrated by the following example.

```mm
NSDecimalNumber *number = [NSDecimalNumber decimalNumberWithMantissa:12345
                                           exponent:-3
                                           isNegative:YES];
```

这样我大概就明白了他的意思, 这个表达的是一个 mantissa 这个参数 *10 的 exponent 次幂 后面的参数用于指定结果是正还是负, 如果期望为负则填 YES.

但是这个方法在我平时的工作中并不怎么用, 所以就先不纠结了. 其实平时工作中比较常见的还是四则运算, 那就从四则运算开始看吧:

```mm
- (NSDecimalNumber *)decimalNumberByAdding:(NSDecimalNumber *)decimalNumber;
- (NSDecimalNumber *)decimalNumberByAdding:(NSDecimalNumber *)decimalNumber withBehavior:(nullable id <NSDecimalNumberBehaviors>)behavior;

- (NSDecimalNumber *)decimalNumberBySubtracting:(NSDecimalNumber *)decimalNumber;
- (NSDecimalNumber *)decimalNumberBySubtracting:(NSDecimalNumber *)decimalNumber withBehavior:(nullable id <NSDecimalNumberBehaviors>)behavior;

- (NSDecimalNumber *)decimalNumberByMultiplyingBy:(NSDecimalNumber *)decimalNumber;
- (NSDecimalNumber *)decimalNumberByMultiplyingBy:(NSDecimalNumber *)decimalNumber withBehavior:(nullable id <NSDecimalNumberBehaviors>)behavior;

- (NSDecimalNumber *)decimalNumberByDividingBy:(NSDecimalNumber *)decimalNumber;
- (NSDecimalNumber *)decimalNumberByDividingBy:(NSDecimalNumber *)decimalNumber withBehavior:(nullable id <NSDecimalNumberBehaviors>)behavior;
```

...这一看就应该是加减乘除吧, 但是他每种算法都提供了两个方法, 不同地方是后面多了一个 behavior 的参数. 那这个 behavior 是个啥呢? 点进去 NSDecimalNumberBehaviors 一看是一个 protocol. 于是我就去找了遵循这个 protocol 的类, 发现了这个类: NSDecimalNumberHandler, 它的初始化方法是这样的:
```mm
- (instancetype)initWithRoundingMode:(NSRoundingMode)roundingMode scale:(short)scale raiseOnExactness:(BOOL)exact raiseOnOverflow:(BOOL)overflow raiseOnUnderflow:(BOOL)underflow raiseOnDivideByZero:(BOOL)divideByZero NS_DESIGNATED_INITIALIZER;

+ (instancetype)decimalNumberHandlerWithRoundingMode:(NSRoundingMode)roundingMode scale:(short)scale raiseOnExactness:(BOOL)exact raiseOnOverflow:(BOOL)overflow raiseOnUnderflow:(BOOL)underflow raiseOnDivideByZero:(BOOL)divideByZero;
```

其实这个无非是对于结果的一些定制, 我们先来看它的参数:

_roundingMode_: 这个参数代表对结果的四舍五入操作, 苹果文档中有一个表格是这样介绍的:

![](/img/roundingmode.png)

_scale_: 结果保留到小数点后几位.

_exact_, _overflow_, _underflow_, _divideByZero_, 这几个参数都是 Bool 值, 表示如果当前进行的计算如果有精确错误, 溢出错误, 和除以0这样的计算错误时是否把异常抛出来中断程序

比如说这样:

```mm
NSDecimalNumberHandler *handler = [NSDecimalNumberHandler decimalNumberHandlerWithRoundingMode:NSRoundBankers scale:2 raiseOnExactness:NO raiseOnOverflow:NO raiseOnUnderflow:NO raiseOnDivideByZero:YES];
        
NSDecimalNumber *num1 = [NSDecimalNumber decimalNumberWithString:@"3"];
NSDecimalNumber *num2 = [NSDecimalNumber decimalNumberWithString:@"0"];

NSDecimalNumber *resultNumber = [num1 decimalNumberByDividingBy:num2 withBehavior:handler];
NSLog(@"%@", resultNumber);
```

注意 __divideByZero__ 这个参数我填写的是 YES 这个时候运行程序是会这样的:

![](/img/decimalNumCrash.png)

如果我把这个参数改成 NO 结果就不一样了:

![](/img/decimalNumNan.png)

结果返回了一个 NaN (not a number) 而不会崩溃. 其他的几个参数效果和这个类似.

了解这以上的东西之后我们就能做一些数据运算了:

```mm
NSDecimalNumberHandler *handler = [NSDecimalNumberHandler decimalNumberHandlerWithRoundingMode:NSRoundBankers scale:2 raiseOnExactness:NO raiseOnOverflow:NO raiseOnUnderflow:NO raiseOnDivideByZero:NO];

NSDecimalNumber *num1 = [NSDecimalNumber decimalNumberWithString:@"3.33"];
NSDecimalNumber *num2 = [NSDecimalNumber decimalNumberWithString:@"0.9234"];

NSDecimalNumber *resultNumber = [num1 decimalNumberByAdding:num2 withBehavior:handler];
NSLog(@"%@", resultNumber); // 4.25
```

这样还是会四舍五入的, 如果你不确定你的计算小数点后面有几位的话(当然上面的计算很明显), 你可以多给几位 _scale_ 比如:
```mm
 NSDecimalNumberHandler *handler = [NSDecimalNumberHandler decimalNumberHandlerWithRoundingMode:NSRoundBankers scale:10 raiseOnExactness:NO raiseOnOverflow:NO raiseOnUnderflow:NO raiseOnDivideByZero:NO];
        
NSDecimalNumber *num1 = [NSDecimalNumber decimalNumberWithString:@"3.333"];
NSDecimalNumber *num2 = [NSDecimalNumber decimalNumberWithString:@"0.9234"];

NSDecimalNumber *resultNumber = [num1 decimalNumberByAdding:num2 withBehavior:handler];
NSLog(@"%@", resultNumber); // 4.2564
```

这样即使你保留了小数点后面很多位, 但是如果都是0的话, 编译出来也会帮你自动去掉的

当然 还有一些其他的运算:
```mm
// n 次方运算
- (NSDecimalNumber *)decimalNumberByRaisingToPower:(NSUInteger)power;
- (NSDecimalNumber *)decimalNumberByRaisingToPower:(NSUInteger)power withBehavior:(nullable id <NSDecimalNumberBehaviors>)behavior;
// 指数运算
- (NSDecimalNumber *)decimalNumberByMultiplyingByPowerOf10:(short)power;
- (NSDecimalNumber *)decimalNumberByMultiplyingByPowerOf10:(short)power withBehavior:(nullable id <NSDecimalNumberBehaviors>)behavior;
// 定制 DecimalNumber
- (NSDecimalNumber *)decimalNumberByRoundingAccordingToBehavior:(nullable id <NSDecimalNumberBehaviors>)behavior;
// 比较运算
- (NSComparisonResult)compare:(NSNumber *)decimalNumber;
```

用法都差不多, 就不一一列举了