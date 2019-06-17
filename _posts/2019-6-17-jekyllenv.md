---
title: jekyll 本地环境问题记录
tags: [jekyll, ruby]
date: 2019-6-17
---
前两天在百度上买了个空间玩一玩, 最初想把我的静态博客扔上去试试, 结果在生成网站的时候遇到了一些问题, 记录一下解决过程。
<!-- more -->
首先我本地是没有 jekyll 环境的, 因为我的博客是放在 github 上面, 这不需要本地环境直接写完 post 上去就可以了, 但是放到自己的服务器上需要在本地先生成静态网站在扔上去。

首先安装 jekyll 环境 
```
sudo gem install jekyll
```
这一步是没有问题的, 关于镜像源的问题就不提了, 国内的镜像源网上随便搜个就行, 但是到安装依赖的时候就出现了问题, jkeyll 依赖的安装和 cocoapods 依赖安装还是很像的, 有 Gemfile 和 Gemfile.lock 安装依赖的工具需要使用到 bundle
```
sudo gem install bundle
```
如果你不安装这个 bundle 直接执行 `jkeyll server` 他会提示你某些包找不到, 比如这样:
```
`block in materialize': Could not find i18n-0.8.6 in any of the sources
```
虽然可以使用`sudo gem install i18n --version 0.8.6`这样的命令去安装, 不过这样也太麻烦了一点, 每遇到一个依赖包都要这样执行一遍, 所以要用到 bundle 这个命令。

安装 bundle 这一步也是没有什么问题的, 但是当安装依赖的时候就开始出现问题了:
```
bundle install
/Library/Ruby/Site/2.3.0/rubygems.rb:289:in `find_spec_for_exe': can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
	from /Library/Ruby/Site/2.3.0/rubygems.rb:308:in `activate_bin_path'
	from /usr/bin/bundle:23:in `<main>'
```
这个问题我是在 Stack Overflow 上解决的, 也就是他推荐的那一条[地址在这里](https://stackoverflow.com/questions/54042692/rbenv-find-spec-for-exe-cant-find-gem-bundler-0-a-with-executable-bun)
```
This is how I finally solved this problem:

$ cd /path/to/my/project/
$ gem install bundler -v 1.17.3
$ bundle install
```
安装指定版本`1.17.3`的 bundle 解决了不能安装依赖的问题, 然后继续`bundle install`这次成功了, 等了一会安装完了依赖。

然后我们来启动本地服务吧, `jekyll server`, 没想到又报错了:
```
jekyll server
/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/2.3.0/universal-darwin18/rbconfig.rb:215: warning: Insecure world writable dir /usr/local/bin in PATH, mode 040777
/Library/Ruby/Site/2.3.0/bundler/runtime.rb:313:in `check_for_activated_spec!': You have already activated i18n 0.9.5, but your Gemfile requires i18n 0.8.6. Prepending `bundle exec` to your command may solve this. (Gem::LoadError)
	from /Library/Ruby/Site/2.3.0/bundler/runtime.rb:31:in `block in setup'
	from /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/2.3.0/forwardable.rb:202:in `each'
	from /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/2.3.0/forwardable.rb:202:in `each'
	from /Library/Ruby/Site/2.3.0/bundler/runtime.rb:26:in `map'
	from /Library/Ruby/Site/2.3.0/bundler/runtime.rb:26:in `setup'
	from /Library/Ruby/Site/2.3.0/bundler.rb:107:in `setup'
	from /Library/Ruby/Gems/2.3.0/gems/jekyll-3.8.5/lib/jekyll/plugin_manager.rb:50:in `require_from_bundler'
	from /Library/Ruby/Gems/2.3.0/gems/jekyll-3.8.5/exe/jekyll:11:in `<top (required)>'
	from /usr/bin/jekyll:23:in `load'
	from /usr/bin/jekyll:23:in `<main>'
```
这个问题很简单, 就是有多个版本的包, 想要解决这个问题可以在所运行的命令前面加上`bundle exec`, 所以要开启本地环境需要运行的命令为`bundle exec jekyll server`, 就可以成功开启了, 千万不要一个一个的去删除不同版本, 这样只会浪费时间...