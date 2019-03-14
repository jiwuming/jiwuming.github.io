---
title: Cordova iOS环境搭建
tags: [cordova]
date: 2019-3-14
---
由于现在 App 改为前端开发, iOS原生作为辅助开发, Cordova 打包工具是个不错的选择, 而且它允许你编写原生插件来供前端使用, 这大大减轻了原生的开发负担, 记录一下 Cordova 的 iOS 端环境搭建过程。
<!-- more -->
首先去 [Cordova 的官网](https://cordova.apache.org/#getstarted) 去看文档下载 Cordova。其实很简单, 就是几个命令:
安装
```bash
npm install -g cordova
```
创建工程:
```bash
cordova create MyApp
```
添加平台:
```bash
cd MyApp
cordova platform add browser
cordova platform add ios
cordova platform add android
```
运行:
```bash
cordova run browser
```
上面这些过程都是非常顺利的, 生成的目录结构如下
![](/img/cordovadir.png)
所以我们只要把前端打包放进www目录下面, 然后在`config.xml`中`<content src="index.html" />`这里来指定入口就行了。

第一个问题来了, 比如我想用示例工程在手机上跑一下, 这时候需要用 cordova 编译
```bash
cordova build ios
```
这里我出现了一个错误:
```bash
error: archive not found at path 'cordova工程路径/platforms/ios/HelloCordova.xcarchive'
** EXPORT FAILED **

CordovaError: Promise rejected with non-error: 'Error code 65 for command: xcodebuild with args: -exportArchive,-archivePath,HelloCordova.xcarchive,-exportOptionsPlist
```
我一开始以为是权限问题, 但其实并不是这样, 你给他分配了读写权限还是不行, 后台才知道编译还需要借助工具:
```bash
xcode-select --install
sudo npm install -g ios-sim
sudo npm install -g ios-deploy --unsafe-perm=true --allow-root
```
其中第一步一般是不太需要的, 因为装过 Xcode 的一般这个也不用再次安装了, 第三步看情况, 如果不能安装就把后面的参数去掉。

在安装了上面的两个插件以后, iOS 端可以正常编译了, 但还是可能存在导出包错误, 不能通过命令行运行真机( Xcode.app 中一个 python 脚本报错)等情况, 不过这些都不要紧, 他们是不会影响工程运行的, 我现在是通过 Xcode 来运行工程, 这个问题先放在这里以后解决。

第二个问题: 

我是使用的 Angular 开发的前端, 在打包完放进 www 目录之后前端是一片空白~ 在 vConsole 中观察 `<approot></approot>` 中的标签没有渲染出来, 所以怀疑是因为目录的问题js并没有被执行。这个时候也是借助了这个插件()
```bash
cordova plugin add cordova-plugin-ionic-webview
```
顺便安装了这个(访问远程网站的)
```bash
cordova plugin add cordova-plugin-remote-injection
```
这样就能顺利的访问页面了。

