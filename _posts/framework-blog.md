---
title: 解决Xcode导入自定义库报错的问题
tags: iOS
date: 2016-07-02
---
这个问题在使用富有支付的时候发生过一次, 大概是这个样子 `dyld: Library not loaded` 记录一下

今天在工程里面新添加了一个支付的sdk, 这个sdk是不支持模拟器运行的, 我就用了真机测试, 但是真机运行的时候报了以下错误:
```mm
dyld: Library not loaded: @rpath/FUMobilePay.framework/FUMobilePay
  Referenced from: /var/containers/Bundle/Application/B58F6BE1-6FA8-4060-8C0F-6424D5127DD2/ChaoDai.app/ChaoDai
  Reason: image not found
(lldb) 
```
<!-- more -->
这个不是sdk的问题, 我也在`Link Binary With Libraries` 里面导入了 `FUMobilePay.framework` 但结果还是报这个错误, 后来找到了以下的解决方法:

![](/img/framework.jpg)

再次真机运行, 就可以了

__注意: 不要把库的 Required 的状态改成 Optional 如果这样更改是不会报错 但是运行有时候不会加载这个库 导致什么效果也没有__ 
