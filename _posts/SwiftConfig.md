---
title: Swift OC 混编设置
tags: [Swift]
date: 2016-03-20
---
自从 Swfit 2.2 之后就没写过 Swift 了, 最近又捡起了 Swift, 以前写的东西也忘得差不多了, 写个笔记记录一下.

格式化控制台输出

```js
func ALLog<T>(_ messsage : T, file : String = #file, funcName : String = #function, lineNum : Int = #line) {
    #if DEBUG
        let fileName = (file as NSString).lastPathComponent
        print("➡️\(fileName)⬅️ ⚠️:\(funcName) 第\(lineNum)行 😯😯\n\(messsage)")
    #endif
}
```
<!--more-->
在 Swift 中, 直接使用 #if DEBUG 是会报错的, 需要现在配置中设置:

target -> Build Settings -> Other Swift Flags 中添加 `-D DEBUG` 

使用:

```js
ALLog("hello world")
```

由于大多数项目是由 OC 转到 Swift 的, 避免不了 Swift 和 OC 的交互, 先来看看头文件的导入方法:

Swift 导入 OC 的头文件:

新建一个 .h 文件, 我们就叫他 BridgeFile 吧, 然后在 target -> Build Settings -> Swift Compiler - General -> Objective-C Bridging Header 中 写入 '工程目录'/BridgeFile.h 然后把需要使用的 OC 的头文件写在 BridgeFile.h 中, 这样新建的 Swift 文件就能使用 BridgeFile.h 中的类了

OC 导入 Swift 

在 OC 的 .pch 中 #import "工程名-Swift.h"
这样就能使用 Swift 的类了

