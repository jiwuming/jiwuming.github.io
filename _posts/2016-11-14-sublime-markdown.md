---
title: 使用Sublime + Markdown 插件来写博客
tags: [Sublime, Markdown]
date: 2016-11-14
---
大多数的博客都是使用markdown来编写, 以前都是直接用Xcode来编辑.md文件😂😂, 现在想想也是醉了, 后来使用的Sublime + OmniMarkupPreviewer 感觉还是非常好用的, 记录一下安装过程.

我使用的是Mac平台, 首先需要下载[Sublime](http://www.sublimetext.com/3), 可以根据需要去下载不同平台的版本.
<!--more-->

对于写markdown来说, 我们首先需要安装一个__PackageControl__的插件. 那么介个插件怎么安装呢? 很简单, 打开Sublime, 然后按下__control + ~__按下之后Sublime的底部会弹出命令行窗口, 执行命令:

(注意自己安装的版本: Sublime2)
```bash
import urllib2,os; pf='Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); open( os.path.join( ipp, pf), 'wb' ).write( urllib2.urlopen( 'http://sublime.wbond.net/' +pf.replace( ' ','%20' )).read()); print( 'Please restart Sublime Text to finish installation')
```

(注意自己安装的版本: Sublime3)
```bash
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

安装完之后会弹出一个窗口, 点击OK就好了.

接下来安装markdown的插件, 安装__PageControl__完毕之后, 重启Sublime. 重启之后按下__Command + Shift + P__ 打开命令版, 搜索__packge install__, 然后会看到有__Package Control: Install Package__这一项, 点击这一项, Sublime请求插件仓库索引, 可以在Sublime看到请求进度. 

请求完毕之后, 输入__markdown editing__点击安装, 安装之后重启Sublime, 当你再打开.md的文件的时候Sublime的界面就改变了, 之后再输入markdown语法时就会出现语法标记了.

但是这样还不够爽, 因为我写完一次, 就要把我的hexo生成到本地来看一下还要去终端执行命令, 还要去刷新网页...如果能有个实时观察编辑结果的插件就好了.

还好Sublime也为我们提供了这样的插件它叫__OmniMarkupPreviewer__安装步骤同上. 
```bash
// 打开命令版
command + shift + p
// 请求插件仓库
package install
// 搜索插件
omnimarkuppreviewer
// 点击安装...
```

安装之后, 重启Sublime, 在Sublime的任意处点击右键, 有个选项叫__Preview Markup in Browser__点击之后, 你就可以在浏览器里面实时的观察你的编辑结果了, 完全不需要生成页面还有刷新的操作. 是不是很爽!


