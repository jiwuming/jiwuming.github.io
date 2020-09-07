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