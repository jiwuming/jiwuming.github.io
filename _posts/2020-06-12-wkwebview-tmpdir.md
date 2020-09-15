---
title: WKWebView 一个奇怪的bug
tags: [iOS]
date: 2020-06-12
---

今天在进行测试的时候发现了一个`WKWebView`奇怪的bug。h5页面中有视频播放，在 iPhone X 以下的手机上都没有问题，但在 iPhone X 以上的手机都不能播放视频，视频会一直加载。

本来我以为这个问题是前端的兼容性问题。因为同时测试的 Android 同事也有一些机型和系统不能播放视频。我们的 h5 同事是用`video`标签直接加载视频的。后来他选用了[video.js](https://github.com/videojs/video.js)作为播放器组件，虽然解决了部分机型问题，但仍然没有完全解决问题。

后来我把`WKWebView`换成`UIWebView`，在我两个同事的 iPhone 11 上都没有问题，但是到了测试手里有一个 iPhone X 还是不行。。。考虑到`UIWebView`比较古老，且更改了之后实现交互的方式也全都要变掉，就没有采用这种方式，即使它也能解决一部分问题。。。

于是我新建了一个空白工程，使用`WKWebView`只加载那个 h5 页面，发现视频是可以正常播放的。

这样就可以判断前端基本上没有什么问题了。或许是工程中哪里出现了问题，于是我把app入口的控制器直接改成自己的webView控制器。运行，发现还是不行。。。

干脆把`Appdelegate`中所有的代码都注释掉了，只保留入口控制器，发现视频是可以播放的。

于是我开始一步一步的打开刚刚的注释，发现清空本地`tmp`文件夹会影响`WKWebView`的视频播放。。。

```swift
func clearTmpPath() {
    var realPath = self.getPathString(systemPath: .documentDirectory)
    let count = realPath.count
    realPath = realPath.subString(rang: NSRange.init(location: 0, length: count - domainStr.count)) + "tmp"
    do {
        let arr = try FileManager.default.contentsOfDirectory(atPath: realPath)
        for i in arr {
            debugPrint("将要删除文件 " + i)
            let p = realPath + "/" + i
            try FileManager.default.removeItem(atPath: p)
        }
    } catch let err {
        debugPrint(realPath)
        debugPrint("清空临时文件夹出错")
        debugPrint(err)
    }
}
```
这个是在`Appdelegate`中执行的。去掉就可以了。原因是app在启动的过程中会向`tmp`目录中创建`Webkit`文件夹，这个文件夹就是在app启动的时候才被检测创建的，之后的过程都不会被检测，所以视频无法被缓冲，才会导致的结果。

亏我以前还研究过`WKWebView`的缓存路径。。。却忽略了这里。