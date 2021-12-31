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


xcode排序错乱问题 https://www.jianshu.com/p/d3827b515e15

autoreleasepool 一般用于产生临时变量很多的位置, 比如sdwebimage的磁盘读取缓存, 他本身要做从nsdata到uiimage的转换

https://logoly.pro/#/

https://tech.glowing.com/

// webpack bootstrap
https://stevenwestmoreland.com/2018/01/how-to-include-bootstrap-in-your-project-with-webpack.html

// eclipse to intellij idea
https://www.cnblogs.com/ceshi2016/p/7890809.html
解决控制台乱码问题(运行tomcat时，控制台乱码)
https://blog.csdn.net/u012611878/article/details/80723491
tomcat 热部署
https://www.jianshu.com/p/038700295d02

jsp 中有vue 页面加载时 会挤在一起 解决方式:
<style>
    [v-cloak] {
        display: none;
    }
</style>
<div class="page" id="app" v-cloak>

ie 11 不支持promise


framework 制作
必须使用纯语言 不可混编 即使能引入两种语言但是无法架起桥接文件
oc项目使用swift framework要添加桥接文件 即新建一个swift file 然后根据提示添加即可 不然会报100个错误(真 100)
如果Mach -O type 不是 static 打出来的framework进到工程里面引用会报dyld: Library not loaded:错误
swift项目制作的framework会把pod中内容也打进去，包比较大，oc制作的framework不会把pod中的内容打进去，包比较小，依赖需要在使用framework中的项目中安装。

iOS 打包framework
http://noxchen.com/2020/07/09/Cocoapods%E6%89%93%E5%8C%85Framework/

动态库和静态库的区别:

编译时不连接,动态库在main之前被加载, 过多的动态库会使启动过程变慢,动态库体积小,不绑定程序(理论上说动态库只存在一份)
静态库在编译的时候被连接,静态库体积大,多个程序中会有多个静态库


cocoapods 具体制作过程
pod lib create GQEnergie 创建项目按需求选择条件
What platform do you want to use?? [ iOS / macOS ]
 >         
ios
What language do you want to use?? [ Swift / ObjC ]
 > ObjC

Would you like to include a demo application with your library? [ Yes / No ]
 > 
yes
Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None

Would you like to do view based testing? [ Yes / No ]
 > 
yes
What is your class prefix?
 > GQ

Running pod install on your new library.

工程建好之后把.podspec所在的同名文件夹引入到工程中, 在class中编写代码, 并提交到同名的github仓库, 打一个和.podspec中版本号相同的tag

pod trunk me 检查登录人员

pod trunk register 你的邮箱 '用户名' --description='描述内容' （邮箱参数是必须的，用户名和--description参数可省略）

pod spec lint GQEnergie.podspec --verbose 检查pod是否有错误

[!] The spec did not pass validation, due to 1 warning (but you can use `--allow-warnings` to ignore it).

可以使用 pod spec lint GQEnergie.podspec --verbose --allow-warnings 跳过一些警告

pod trunk push GQEnergie.podspec --allow-warnings 发布项目

--------------------------------------------------------------------------------
 🎉  Congrats

 🚀  GQEnergie (0.1.0) successfully published
 📅  August 19th, 23:31
 🌎  https://cocoapods.org/pods/GQEnergie
 👍  Tell your friends!
--------------------------------------------------------------------------------

发布成功了

私有项目发布到gitlab上面 同样需要打tag然后在其他项目中引用的时候需要像这样
pod 'UTest', :git=>'http://106.15.88.88/gaoqi/UTest.git'


// 参数签名原理
https://blog.csdn.net/qq_15901351/article/details/80175169

RSA 加密原理
1. 首先找到两个质数 p q
2. 求出两个数字的乘积 n = p * q
3. 通过欧拉函数得到 f(n) = (p - 1)(q - 1)
4. 找出公钥和私钥 
    公钥 e 是 1 < e < f(n) 并且 e 与 f(n) 互质
    私钥 d 是 e * d / f(n) 的余数为1
5. 明文 m 加密实现
    m 的 e 次幂 / n 的余数 c 即为密文
6. 密文c的解密实现
    c 的 d 次幂 / n 的余数 即可还原明文 m

// iOS 触摸事件如何被runloop处理
https://www.jianshu.com/p/d547e5393373

// 手动取消kvo
https://www.jianshu.com/p/8b600bcf605e

// 面试题
https://github.com/colourful987/bytedance-alibaba-interview


atomic问题
// https://www.jianshu.com/p/c40b312153c1

po主理解错了。atomic是绝对安全的。
我们知道，在64位的操作系统下，所有类型的指针，包括void * 都是占用8个字节的。超过4个字节的基本类型数据都会有线程并发的问题。
那所有的指针类型都会有这个问题。
以oc 下的 NSArray * 为例子，如果一个多线程操作这个数据，会有两个层级的并发问题
1、指针本身
2、指针所指向的内存 

指针本身也是占用内存的，并且一定是8个字节，第二部分，指针所指向的内存，这个占多少字节就不一定了，有可能非常大，有可能也就1个字节


所以我们考虑NSArray * array 这个数据array 多线程操作的时候，必须分成两部分来描述，一个是&array这个指针本身，另一个则是它所指向的内存 array
大家注意下 &array 和 array 的区别 ，其实不用纠结，你就想象现在有两块内存，一块是8字节，一块n字节，8字节里面放的值，就是n字节内存的首地址，

ok 现在联系上atomic，如果用@property(atomic)NSArray *array 修饰之后，会有什么影响？网上说的很多，不再赘述，我只想从内存的角度来解释这个过程

首先第一点，你要记住，@property(atomic)NSArray *array 其实修饰的是这个指针，也就是这个8字节内存，跟第二部分数据n字节没有任何关系，被atomic 修饰之后，你不可能随意去多线程操作这个8字节，但是对8字节里面所指向的n字节没有任何限制！这就是所有网络上所说的 atomic 不安全的真相 ！！！

我们来看一下，这能怪atomic？ 本身你修饰的是一个指针，并且atomic 已经完美的履行了它的指责，你现在不可能对这个8字节进行无序的多线程操作，这就够了呀！atomic没有任何鸟问题。有问题的是人，你本身并未对n字节做任何的限制，所以把问题怪罪到atomic 上真的是很不合理

私有项目发布到gitlab上面 同样需要打tag然后在其他项目中引用的时候需要像这样
pod 'UTest', :git=>'http://106.15.88.88/gaoqi/UTest.git'


// 参数签名原理
https://blog.csdn.net/qq_15901351/article/details/80175169

RSA 加密原理
1. 首先找到两个质数 p q
2. 求出两个数字的乘积 n = p * q
3. 通过欧拉函数得到 f(n) = (p - 1)(q - 1)
4. 找出公钥和私钥 
    公钥 e 是 1 < e < f(n) 并且 e 与 f(n) 互质
    私钥 d 是 e * d / f(n) 的余数为1
5. 明文 m 加密实现
    m 的 e 次幂 / n 的余数 c 即为密文
6. 密文c的解密实现
    c 的 d 次幂 / n 的余数 即可还原明文 m

// iOS 触摸事件如何被runloop处理
https://www.jianshu.com/p/d547e5393373

// 手动取消kvo
https://www.jianshu.com/p/8b600bcf605e

// 面试题
https://github.com/colourful987/bytedance-alibaba-interview

// 符号表
<起始地址> <结束地址> <函数> [<文件名>:<行号>]
https://www.jianshu.com/p/a30dac2328cb


# 赛马算法

https://zhuanlan.zhihu.com/p/103572219

# 七层http模型和五层tcp/ip

http:      
* 应用层            
* 表达层
* 会话层
* 网络层
* 传输层
* 数据链路层
* 物理层

tcp/ip
* 应用层
* 网络层
* 传输层
* 数据链路层
* 物理层

# https实现过程

1. 首先客户端需要去服务端请求RSA的公钥
2. 服务端返回公钥，客户端验证其有效性，如果是有效的，继续下一步，如果无效则警告
3. 客户端生成一个随机AES秘钥，并用公钥加密，传输给服务端
4. 服务端接受到加密的秘钥之后用RSA私钥解密，得到客户端的AES秘钥
5. 服务端用AES秘钥加密内容传输给客户端
6. 客户端用生成的AES秘钥解密内容

# 请求头：

Accept，Content-Type，Orgin，Content-Length，Cookie，User-Agent

# 响应头

Access-Control-Allow-Origin，Content-Encoding，Content-Length，Content-Type，Status

# 腾讯

## 自我介绍
## mrc和arc

* MRC Mannul Reference Counting 手动引用计数
1. 每当一个对象创建时，引用计数为1
2. 当这个对象被其他指针引用时，引用计数加1
3. 当其他指针不在引用这个对象时，引用计数减1
4. 当引用计数为0的时候，对象释放（最后在dealloc中再释放一次即为0）
5. 原则：谁创建，谁释放，谁引用，谁管理

* ARC Auto Reference Counting 自动引用计数
1. 编译器会在编辑及运行的过程中手动的插入`retain`，`release`，`autorelease`
2. mrc与arc共存：mrc下使用 `-fobjc-arc`，arc下使用`-fno-objc-arc`

## 内存五大区域
1. 栈区

创建临时变量时由编译器自动分配，在不需要的时候自动清除的变量存储区，内存分配时连续的。

2. 堆区

由程序员手动管理的内存区域，如果不释放，在程序结束时由系统回收

3. 全局区

全局变量和静态变量时放在一块的，初始化的在同一块区域，未初始化的在另一块区域，

4. 常量区

常量，不允许被修改，比如常量字符串

5. 代码段

存放函数体的二进制代码

## 自动释放池
MRC中使用 `autorelease` 方法及 `NSAutoReleasePool` 

ARC中使用 `@autorelease` 

`autorelease` 是为了更好的管理内存，当我们希望一个对象在将来的某一个时刻释放（这个时刻由系统确定）时可以是用 `- autorelease` 方法或 `@autorelease{}`

![](./autorelease.jpg)

- AutoreleasePool并没有单独的结构，而是由若干个AutoreleasePoolPage以双向链表的形式组合而成（分别对应结构中的parent指针和child指针）
- AutoreleasePool是按线程一一对应的（结构中的thread指针指向当前线程）
- AutoreleasePoolPage每个对象会开辟4096字节内存（也就是虚拟内存一页的大小），除了上面的实例变量所占空间，剩下的空间全部用来储存autorelease对象的地址
- 上面的id *next指针作为游标指向栈顶最新add进来的autorelease对象的下一个位置
- 一个AutoreleasePoolPage的空间被占满时，会新建一个AutoreleasePoolPage对象，连接链表，后来的autorelease对象在新的page加入


// 撤销commit
https://www.cnblogs.com/lfxiao/p/9378763.html


// git rebase
https://www.jianshu.com/p/6960811ac89c

// WKWebView 播放视频问题，在清空本地的tmp目录后不能播放视频 贼奇葩

swift codeable 协议字典转模型


中间人攻击
git merge rebase 区别
如何校验https公钥有效性
遍历二叉树
链表
断点续传状态码
loadview是否一定会执行

多个wkwebview共享缓存
swift可选协议
单线程如何实现异步
js class的本质如何实现继承
flutter如何实现并发
flutter context 本质是什么
atomic和noatomic
自旋锁风险
自旋锁锁住一个nil会发生什么
组件化 https://halfrost.com/vue_ios_modularization/


jsdelivr 加速github 仓库可以当图床用
github 镜像 hub.fastgit.org

浏览器校验公钥有效性 https://www.v2ex.com/t/411144

hybird app 离线包解决方案
https://github.com/mcuking/blog/issues/63
http://nanhuacoder.top/2019/04/11/iOS-WKWebView02/

wkwebview 通过loadrequest加载的post请求会丢失body字段, 这个可以通过把body中的字段放入header中解决, header不会丢掉

iOS 本地服务器
https://juejin.cn/post/6844903492537024525

intrinsic size placeholder xib 用内容撑开布局
在约束面板最下面 设置 intrinsic 属性为 placeholder
https://stackoverflow.com/questions/19677142/what-is-the-difference-of-intrinsic-size-vs-system-width-height-constraints

charles 中间人攻击原理
https://github.com/yuansirios/iOS-Learning-Collection/tree/master/04

路由设计模式

模块(module)
逻辑(handler)
事件(event)
对象工厂(objectProvider)
数据工厂(dataProvider需要考虑数据变化之后发送通知)


需要整理知乎, github上的内容


腾讯mars
https://github.com/AlloyTeam/Mars
移动端font-family https://github.com/AlloyTeam/Mars/blob/master/solutions/font-family.md

vw适配 mobile.width = 750px  => 750px = 100% =100vw => 750px = 100vw => 1px = 0.1333vw => 100px = 13.33vw => 1rem =100px;

iOS 启动速度优 二进制重拍
https://juejin.cn/post/6844904168201666574



基本：
两列列表实现
1像素线
Style写在页面上和写在link标签中区别
闭包的概念
原生dom方法增加删除class样式

VUE：
vue 网络请求是在什么生命周期发起
组件调用(父,子)
如何实现响应式布局
动码倒计时
diff算法更新视图对比
vue白屏优化
uniapp为什么能支持多平台小程序

项目实例


Git reabse

git checkout master
git pull
git checkout local
git rebase -i HEAD~2  //合并提交 --- 2表示合并两个
git rebase master---->解决冲突--->git rebase --continue
git checkout master
git merge local
git push




移动端 H5 相关问题汇总：
1px 问题
响应式布局
iOS 滑动不流畅
iOS 上拉边界下拉出现白色空白
页面件放大或缩小不确定性行为
click 点击穿透与延迟
软键盘弹出将页面顶起来、收起未回落问题
iPhone X 底部栏适配问题
保存页面为图片和二维码问题和解决方案
微信公众号 H5 分享问题
H5 调用 SDK 相关问题及解决方案
H5 调试相关方案与策略
iOS 短信验证码重复输入问题
rem vs vw 方案

关于Hybrid App 开发相关问题：

离线包 iframe 跨域解决方案
jsBridge 通信机制






整理前端一些常见的问题解决方案

空白屏监控
曝光上报
bug 收集上报
webpack 打包大小优化
前端 http 缓存设置
iframe 离线包解决方案





手写实现各种方案、实践等


手写 React：toy-react
手写 Promise
手写 bind
手写 umi 框架
手写 webpack
手写 tapable
手写 mvvm


流程图绘制工具
https://www.processon.com/login?backUrl=/diagraming/619625e10e3e7409b9c634e4


Dan的博客
https://overreacted.io/zh-hans/

react hooks 原理
https://github.com/brickspert/blog/issues/26


cocoapods 私有库,版本号,指定分支
https://www.cnblogs.com/maybe-liu/p/8658347.html

iOS ANR
https://blog.csdn.net/lfdanding/article/details/99710420

前端 移动端适配的坑
https://zhuanlan.zhihu.com/p/268677938


一些前端问题

空白屏监控
曝光上报
bug 收集上报
webpack 打包大小优化
前端 http 缓存设置
iframe 离线包解决方案


