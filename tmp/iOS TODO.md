reactiveObjc/rxswift (一次性动作用原生方法 持续性动作用Observeable)

yogalayout

cocoapods私有仓库

iOS MVVM

Runtime 埋点 (点击 停留事件 下载事件)

armv7 armv7s arm64 app瘦身(工程角度 压缩图片 减少没用到的方法... 打包部分 选择rebuild...framework 架构部分也就是前面的)

iOS 靠近底部按钮点击响应慢

iOS runtime objc_setAssociatedObject 普通字符串无法取值 必须为静态(Swift要定义在类的外面)https://stackoverflow.com/questions/31594463/objc-getassociatedobject-returning-null

贝塞尔曲线

贝塞尔曲线 + cashapelayer + 动画 https://www.cnblogs.com/pretty-guy/p/8268745.html

单项数据流(主要是react的概念 iOS中应该怎么使用 (iOS rxswift应该是单项数据流的概念(net -> viewmodel -> view component -> viewmodel -> view) 不能算mvvm mvvn是 基于observe的))

常见的排序算法 https://www.cnblogs.com/onepixel/articles/7674659.html

(OK)swift 项目中同时引入 oc桥接文件 和 oc pch文件

去掉黄警告 https://blog.csdn.net/zxw_xzr/article/details/72469110

https://www.cnblogs.com/qianyindichang/p/10911510.html

childViewController

idea 激活

(OK)mac 传感器坏掉 电脑卡顿 风扇转速很快

https://blog.csdn.net/youshaoduo/article/details/101482771

腾讯云IM接入 本地推送 hook机制 

webview 坑 https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653578513&idx=1&sn=961bf5394eecde40a43060550b81b0bb&chksm=84b3b716b3c43e00ee39de8cf12ff3f8d475096ffaa05de9c00ff65df62cd73aa1cff606057d&scene=0&key=18f25f38d156819652293ebc9f45c2c764b733698d14b3fc14c929943e23fcbcafe74634a026860fa9dd877aa69ca12c41eb0b389fa33a62b6e31e7d58ea18597ae0a1235330e198d20094c51b88cc13&ascene=0&uin=MTYxNjEyMjAwMA==&devicetype=iMac%20MacBookPro11,5%20OSX%20OSX%2010.12.2%20build%2816C67%29&version=12010210&nettype=WIFI&fontScale=100&pass_ticket=WOq/WYccfl4MsTFLBPFJ0uDFJaQTttFtzZuBNAXgp/0PsHlLsQieFLlcsVkJldKK



富文本 修改图片大小 https://www.jianshu.com/p/8d97f7772c2c


swift class func 和 static func

屏幕旋转方向

cocoapods github 443









//
//  ViewController.m
//  ObjcMsgForward
//
//  Created by greg on 2020/5/18.
//  Copyright © 2020 greg. All rights reserved.
//

#import "ViewController.h"
#import "TestButton.h"
#import <objc/runtime.h>

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    // 1.测试提供实现的
    // 按钮调用
//    TestButton *button = [[TestButton alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
//    button.backgroundColor = [UIColor redColor];
//    [button addTarget:self action:@selector(buttonAction) forControlEvents:UIControlEventTouchUpInside];
//    [self.view addSubview:button];
//    // 直接调用
//    [self performSelector:@selector(customAction:second:) withObject:@"1" withObject:@"2"];
    
    
    // 2.测试提供对象的
    // [self performSelector:@selector(targetTestAction)];
    
    
    
    // 3. NSInvocation
    [self performSelector:@selector(customAction:second:) withObject:@"234" withObject:@"haha"];
}

// 1.提供实现
//+ (BOOL)resolveInstanceMethod:(SEL)sel {
//    // button实现方法没有找到 会走这里
//    NSLog(@"%@", NSStringFromSelector(sel));
//    // 手动添加方法
//    class_addMethod(self, sel, class_getMethodImplementation(self, @selector(replaceImpl)), "v@:"); // 无返回值 无参数类型
//
//    // 如果上面有了一个 class_addMethod 则不会执行下面这一段了
//    // 如果返回值和参数比较复杂 则获取 Method 然后通过 method_getTypeEncoding 手动获取 types
//    Method m = class_getClassMethod(self, @selector(customActionttttt:second:));
//    class_addMethod(self, sel, class_getMethodImplementation(self, @selector(customActionttttt:second:)), method_getTypeEncoding(m));
//
//    return true;
//}

// 2.提供对象实现
//- (id)forwardingTargetForSelector:(SEL)aSelector {
//    return [[TestButton alloc] init];
//}

// 3.NSInvocation
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSLog(@"%@", NSStringFromSelector(aSelector));
    // 直接这么写是不行的
//    NSMethodSignature * sign = [NSMethodSignature methodSignatureForSelector:aSelector];
//    return sign;
    Method m = class_getInstanceMethod([self class], @selector(customActionttttt:second:));
    return [NSMethodSignature signatureWithObjCTypes:method_getTypeEncoding(m)];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    // 获取到了 Invocation 对象 接下来就是实现
    NSLog(@"%@", anInvocation);
    [anInvocation setTarget:self];
    [anInvocation setSelector:@selector(customActionttttt:second:)];
    // 手动设置参数
    NSString * arg1 = @"arg1";
    NSString * arg2 = @"arg2";
//    BOOL b = true;
    // 第一个参数是2 前面的两个参数是target和selector
    [anInvocation setArgument:&arg1 atIndex:2];
    [anInvocation setArgument:&arg2 atIndex:3];
//    [anInvocation setReturnValue:&b]; // 返回值不写也没关系
    [anInvocation invoke];
}





- (void)replaceImpl {
    NSLog(@"替换了实现");
}

- (BOOL)customActionttttt:(NSString *)str second:(NSString *)second {
    NSLog(@"这里也被替换了 %@  %@", str, second);
    return true;
}



@end

