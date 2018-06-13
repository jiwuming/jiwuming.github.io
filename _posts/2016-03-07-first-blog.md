---
title: Hexo博客搭建过程
tags: 其他
date: 2016-03-07
---
以前一直是用 Jekyll 模板的, 这次换了 Hexo, 感觉 Hexo 要比 Jekyll 搭建 github 博客麻烦些, 记录一下

我使用的平台是 Mac 平台, 首先需要安装必要的环境, [Node](https://nodejs.org/en/), 用这个来生成静态页面. 其次就是安装 git (Xcode) 已经自带了. 接下来就是配置 github pages. 值得一提的是, 我以前申请的 github 地址都是 xxx.github.io/xxx/ 这种的, 原来在设置里面 rename 成 xxx.github.io 这样之后, 你的地址就变成 xxx.github.io 这样的了.

安装 Node 之后来安装 Hexo, 首先在你想放置博客的地方建立一个空的文件夹, 移动到这个文件夹目录下, 安装 Hexo.
<!-- more -->

执行命令
``` ruby
sudo npm install -g hexo
```
安装之后初始化Hexo
``` ruby
hexo init
```
Hexo就安装完成了. 

接下来你可以去github上挑选自己喜欢的Hexo主题下载下来, 把这个主题拷贝到themes/的目录下, 然后在你博客目录下的_config.yml配置主题, 一般在_config.yml的最下面, 有个deploy, 我是这样配置的: 
``` ruby
deploy:
  type: git
  repository: https://github.com/jiwuming/jiwuming.github.io.git
  branch: master
```
然后执行命令 hexo g 生成静态页面. 再hexo d 发布到github.
也可以用hexo s 在本地生成 看下效果

我用的是 [hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia) 这个主题, 前几天升级了以下主题, 发现增加了许多新的功能, 其中我比较喜欢的的是新出的搜索, 但是搜索的功能不是一开始就能使用的 提示了这个:

![](/img/searchCannotWork.png)

于是我就按照命令来写:

先在博客的根目录运行 `npm i hexo-generator-json-content --save` 然后又在根目录里面的 `_config.yml` 写入了图中的配置, 然后 hexo g 运行了一下, 出现了这个

```bash
ERROR Plugin load failed: hexo-generator-json-content
SyntaxError: Block-scoped declarations (let, const, function, class) not yet supported outside strict mode
    at Object.exports.runInThisContext (vm.js:53:16)
    at /Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/lib/hexo/index.js:227:17
    at tryCatcher (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:502:31)
    at Promise._settlePromise (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:559:18)
    at Promise._settlePromise0 (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:604:10)
    at Promise._settlePromises (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:683:18)
    at Promise._fulfill (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:628:18)
    at Promise._resolveCallback (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:423:57)
    at Promise._settlePromiseFromHandler (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:514:17)
    at Promise._settlePromise (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:559:18)
    at Promise._settlePromise0 (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:604:10)
    at Promise._settlePromises (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:683:18)
    at Promise._fulfill (/Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/promise.js:628:18)
    at /Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/bluebird/js/release/nodeback.js:42:21
    at /Users/huxinjinrongimac-ios/myBlog/node_modules/hexo/node_modules/hexo-fs/node_modules/graceful-fs/graceful-fs.js:78:16
    at FSReqWrap.readFileAfterClose [as oncomplete] (fs.js:380:3)
```
大概的意思就是: 插件加载失败, js 的语法有问题. 我猜测这里可能是使用了 ES6 的新语法和严格模式的问题. 然后我去博客的根目录里面去找 `hexo-generator-json-content` 相关的 js 文件, 发现了一个这样的东西:

![](/img/hexoplugindir.png)

然后抱着试试的心态打开了这个 js 文件在最上面加上了这句 `"use strict"`

再次 hexo g 果然成功了.