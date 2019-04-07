---
title: Cordova 集成 Cocoapods 与 极光推送收不到的问题等解决
tags: [Cordova, iOS]
date: 2019-4-7
---
最近使用 Cordova 进行混合开发了, 前端使用 Angular + ionicUI 感觉还是不错的, 但是也不可避免用到原生的功能, 比如相册, 相机, 推送等, 刚好在写推送的时候遇到了一些问题, 记录一下 Cordova 集成 Cocoapods 的过程。

首先还是在根目录创建`Podfile`然后编写想要安装的pods, 然后执行`pod install`
![](img/cdvpoddir.jpg)
pods 安装完之后再build工程会报一个错误
```sh
diff: /../Podfile.lock: No such file or directory   
diff: /Manifest.lock: No such file or directory error: The sandbox is not in sync with the Podfile.lock. 
Run 'pod install' or update your CocoaPods installation. 
```
这个时候我们去工程的目录里面, 找到`Resources`, 找到`build-debug.xcconfig`文件
![](/img/cdvxcc.jpg)
这是他原本的内容
```sh
#include "build.xcconfig"

GCC_PREPROCESSOR_DEFINITIONS = DEBUG=1

#include "build-extras.xcconfig"

// (CB-11792)
// @COCOAPODS_SILENCE_WARNINGS@ //
#include "../pods-debug.xcconfig"
```
现在我们找到pod的`debug.xcconfig`, 拿到它的路径在上面的`build-debug.xcconfig`文件里面引入
![](/img/cdvpodxcdir.jpg)
那`Resources`中的`build-debug.xcconfig`就变成了这样
```sh
#include "build.xcconfig"

GCC_PREPROCESSOR_DEFINITIONS = DEBUG=1

#include "build-extras.xcconfig"

// (CB-11792)
// @COCOAPODS_SILENCE_WARNINGS@ //
// 这句注释掉了 没有这个路径
//#include "../pods-debug.xcconfig"

#include "Pods/Target Support Files/Pods-HelloCordova/Pods-HelloCordova.debug.xcconfig"
```
之后再 clean + build 即可正常运行, 也可以正常引入第三方库了

还有两个问题 一个是 Cordova 貌似对 `#DEBUG` 这种宏命令不支持 可以通过这个解决 在`Build Setting`中:
![](/img/cdvdebugparams.jpg)

之后就是极光推送收不到的问题, 我用我新建的工程是可以收到推送的, 但是生成的 Cordova 中却怎么也收不到, 需要设置这里:
![](/img/pushnotwork.jpg)
这里原来的选项是`New Build System`, 改成`Legacy Build System`, 删掉手机中的工程, 重新运行, 问题解决~