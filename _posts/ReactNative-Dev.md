---
title: ReactNative 搭建开发环境
tags: [ReactNative, WebStrom]
date: 2017-02-06
---
记录 ReactNaive 环境搭建和开发工具 WebStrom 编写 ReactNative 的基本流程和注意事项

要运行 ReactNative(RN) 程序, 必须要有 node 环境和 App 开发工具 (Xcode 和 AndroidStudio) 其中的一个, 写 RN 代码的开发工具看自己喜欢用什么编辑器, WS 和 Sublime 都是可以的, 不过官方推荐的编辑器是 Atom + Nuclide 环境, 这个坑爹的鬼东西装了半天也没装上, 就用 WS 了
<!--more-->

[ReactNative中文网](http://reactnative.cn)上面有详细的安装教程

Mac 环境下, 用 Homebrew 来安装 Node.js
```bash
brew install node
```
如果遇到了权限问题, 就用下面的命令来修复
```bash
sudo chown -R `whoami` /usr/local
```
安装之后建议设置淘宝的 npm 镜像方便以后的访问
```bash
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```
然后安装 Yarn 和 Watchman
```bash
npm install -g yarn react-native-cli
brew install watchman
```
上面的包都安装好了以后就可以通过终端新建一个 RN 工程了
```bash
react-native init HelloWorld
cd HelloWorld
react-native run-ios
```
接下来就可以用自己喜欢的编辑器去写代码了

### 不过我个人更喜欢 IDE 开发, 现在我把它放到 WS IDE 中来:

创建工程的方法是一样的, cd 到你想创建工程的目录 然后 `react-native init projectName` 等到它创建完毕之后 用 WS 打开这个目录, 效果大概是这样的
![](/img/IMG-RN-DEV/Rn_dir.png)
然后直接运行npm的服务
![](/img/IMG-RN-DEV/Rn_npmStart.png)
这里面可能会有一些端口被占用的问题:
![](/img/IMG-RN-DEV/Rn_portUsed.png)
直接干掉被占用的端口:
```bash
// 查看占用端口的进程 
lsof -i tcp:port
// 杀死上面的进程
kill -9 pid 
```
这时再次点击左侧的运行按钮, 服务启动成功:
![](/img/IMG-RN-DEV/Rn_packgeRun.png)
之后我们就可以运行项目了, 可以直接在 WS 自带的 Terminal 中 run-ios 来启动模拟器

可能有时候会出现这种情况
![](/img/IMG-RN-DEV/Rn_Emo_Error.png)
照着错误提示改就可以了:
```bash
watchman watch-del-all
rm -rf node_modules
npm install
npm start --reset-cache
```
现在再次运行就能看见欢迎页面了
![](/img/IMG-RN-DEV/Rn_welcome.png)

安卓配置开发环境按照[ReactNative 中文网](http://reactnative.cn/docs/0.41/getting-started.html)上那样把 AndroidStudio 配置好就行了, 模拟器方面我选择的是 Genymotion 他还需要 VirtualBox 虚拟机的支持, 和官网上不一样的是: 我刚下下来的 Genymotion 自带了一个手机模拟器, 但是怎么也运行不起来, 一直提示网络配置有问题, 后来我把这个手机模拟器从 Genymotion 中删除了, 又从 Genymotion 里面下载了一个新的模拟器, 直接运行就可以了  
![](/img/Rn_android.png)

### 接下来看代码部分

如果我们的 WS 上没有装任何插件的话, 打开 RN 的工程将会有成吨的警告, 去掉警告的方法就是安装 React 的插件并修改 js 语法标准为 JSX:

#### 1.用 command + , 调出偏好设置 然后找到Languages & Frameworks->JavaScript->Libraries
![](/img/IMG-RN-DEV/Rn_React_Setting.png)
如果 WS 中没有 react 和 react-native 那两项需要点击右面的 Download 来下载
![](/img/IMG-RN-DEV/Rn_ReactPlugin_Download.png)
因为这里已经下载完了 所以第一个结果是不是 react (一般都是第一个结果) 选中之后点击 Download and Install 安装, 然后再像上面那样勾选 `react-DefinitelyTyped` 和 `react-native-DefinitelyTyped`, 然后点击 ok
#### 2.然后让 JavaScript 文件支持 JSX 语法, 还是 command + , 调出偏好设置 然后:
![](/img/IMG-RN-DEV/Rn_Select_JSX.png)
其实这样代码中的警告就已经去除一大半了, 但是工程里面免不了的有一些拼写错误, 可以通过此方法去掉:
![](/img/IMG-RN-DEV/Rn_Spell.png)
如果你写 js 不喜欢写分号 但是 IDE 又报警告时 在设置中搜索 __unterminated statement__ 然后把勾去掉就可以了 (但是官方的代码貌似都是带分号的):
![](/img/IMG-RN-DEV/Rn_Warning.png)

#### 最后的就是调试部分了

如果你在代码上做了一些改动的话, 点击模拟器按下 command + r (iOS), 双击 r (Android) 来刷新最新的结果

RN 允许我们用 Chrome 浏览器的控制台来调试 js 在模拟器的窗口中 用 command + d (iOS), command + m (Android) 能调用出调试工具:
![](/img/IMG-RN-DEV/Rn_debug.png)
选择 Debug JS Remotely 会打开 Chrome 浏览器:
![](/img/IMG-RN-DEV/Rn_Chrome.png)
然后我们直接在这个页面上 右键 检查:
![](/img/IMG-RN-DEV/Rn_Console.png)
这样就能像前端那样来调试 js 了

如果想停止调试模式 再次在模拟器上 command + d 然后 stop debug 就可以了

至于 Chrome 上面的那个黄色的警告:
>  Remote debugger is in a background tab which may cause apps to perform slowly. Fix this by foregrounding the tab (or opening it in a separate window).

这个只要在调试的时候把 Chrome 的调试窗口放在所有窗口的最前面就可以了, 如果不这样调试过程会变慢

对了, 还有一个 ReactNative 代码提示的问题, WS 本身貌似没有提供 ReactNative 的代码提示, 这里有一个插件: 

`https://github.com/virtoolswebplayer/ReactNative-LiveTemplate`

把这个插件下载下来, 然后打开 WS 点击 file->import settings 选择下载下来文件中的 ReactNative.jar 然后 ok 重启 WS 就会有代码提示了
