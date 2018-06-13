---
title: Carthage Build 失败的问题 
tags: [Terminal]
date: 2018-5-4
---
大概情况类似这样
```bash
*** Fetching SwiftyJSON
*** Fetching Alamofire
*** Checking out Alamofire at "4.7.2"
*** Checking out SwiftyJSON at "4.1.0"
*** xcodebuild output can be found in /var/folders/dg/byxrmzh501ggbc21lsnk_z4h0000gn/T/carthage-xcodebuild.RY5lcm.log
A shell task failed with exit code 72:
xcrun: error: unable to find utility "xcodebuild", not a developer tool or in PATH
```
这个是由于Xcode偏好设置中Location的Command Line Tools没有设置导致的
![](/img/xcode-commandlinesetting.png)
我原来圈起来的那个地方是空白的。
```bash
*** Checking out Alamofire at "4.7.2"
*** Checking out SwiftyJSON at "4.1.0"
*** xcodebuild output can be found in /var/folders/dg/byxrmzh501ggbc21lsnk_z4h0000gn/T/carthage-xcodebuild.aMbcFh.log
*** Building scheme "Alamofire iOS" in Alamofire.xcworkspace
*** Building scheme "SwiftyJSON iOS" in SwiftyJSON.xcworkspace
```
再次运行`carthage update --platform iOS`就可以了