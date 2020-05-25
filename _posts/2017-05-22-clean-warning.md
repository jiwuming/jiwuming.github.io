---
title: iOS 去掉工程中的黄色警告
tags: [iOS]
date: 2017-05-22
---
对于有强迫症的人来说, 工程中有很多警告是在是很痛苦, 自己写的代码还好可以时刻注意。但是导入某些第三方库的时候回不可避免的产生一些错误, 比如整型和长整型的问题, 其实这些错误都是可以屏蔽掉的, 记录一下屏蔽过程。

# 对于使用CocoaPods的库
在`Podfile`中加入这句`inhibit_all_warnings!`即可屏蔽掉`Pod`中的文件产生的错误。

# 对于手动导入的库产生的错误
可以通过在错误中右键找到`Reveal in Log`
![](/img/yello-warning.png)

比如有一个废弃的api警告类似这样:
![](/img/deprecatedapi.png)

我们复制方括号中的内容`-Wdeprecated-declarations`
在`Build setting`中找到`Other Warning Flags`
以这种形式添加
```cpp
// 把 -W 后面加 no 再加 - 即可  
-Wno-deprecated-declarations
```

在`build`即不会产生该类型的错误了。