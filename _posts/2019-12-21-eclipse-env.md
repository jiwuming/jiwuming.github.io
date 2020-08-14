---
title: Mac Eclipse
tags: [Java, Eclipse, Mac]
date: 2019-12-21
---
曾经我以为我再也不会用到`Eclipse`来写项目了。。。直到最近接到了一个六七前的老项目。以前上学的时候用过一段时间, 但自从工作之后基本上都是在使用`Intellij`全家桶, 包括像`Intellij IDEA`, `WebStorm`之类的工具。但这个项目没办法, 必须要装一个`Eclipse`了, 记录项目起手的一系列坑~

# 下载
首先去[官网下载](https://www.eclipse.org/downloads/)`Eclipse`, 镜像可以选择我们学校的:
![](/img/eclipsedownload.png)

速度还行, 但是安装速度可能还是稍微感人了点, 毕竟还有一些其他的工具需要通过`Eclipse`去下载, 不过也只能慢慢等。

# 导入工程
下载之后一路默认安装就行, 打开`Eclipse`导入一个工程
![](/img/eclipse-add-dir.png)

# 配置多个jdk版本
由于我自己默认使用的是`jdk1.8`的版本, 但本次项目需要使用`jdk1.6`的版本开发, 所以需要新增一个`jdk1.6`的版本, 可以去苹果官方的[下载地址](https://support.apple.com/kb/DL1572?locale=zh_CN)下载。

下载之后安装, 因为之前安装了`jdk1.8`所以安装器会阻止我们安装程序:
![](/img/eclipse-low-version.png)

解决办法可以参考[Stack Overflow](https://stackoverflow.com/questions/26252591/mac-os-x-and-multiple-java-versions)或者[这篇博客](https://blog.csdn.net/gaofenglxx/article/details/102565883?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-3)

我用的是第二种办法, 直接下载`jdk1.6`的包, 然后去掉脚本中的判断逻辑, 重新封包:
```cpp
hdiutil mount JavaForOSX.dmg
pkgutil --expand /Volumes/Java\ for\ macOS\ 2017-001/JavaForOSX.pkg java6tmp/
sed -i '' 's/return false/return true/g' java6tmp/Distribution
pkgutil --flatten java6tmp/ java6.pkg
```
这样就在下载目录中得到了一个`java6.pkg`的包, 直接双击安装即可。

安装完毕之后我们需要配置环境变量, 打开`bash_profile`
```bash
# Java
export JAVA8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/
export JAVA6_HOME=/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home/
export JAVA_HOME=$JAVA8_HOME
alias jdk6="export JAVA_HOME=$JAVA6_HOME"
alias jdk8="export JAVA_HOME=$JAVA8_HOME"
```
这里默认使用的是`java8`, 可以使用`jdk6`和`jdk8`命令切换`java`环境:
```bash
gregdeMacBook-Pro:~ greg$ jdk6
gregdeMacBook-Pro:~ greg$ java -version
java version "1.6.0_65"
Java(TM) SE Runtime Environment (build 1.6.0_65-b14-468)
Java HotSpot(TM) 64-Bit Server VM (build 20.65-b04-468, mixed mode)
gregdeMacBook-Pro:~ greg$ jdk8
gregdeMacBook-Pro:~ greg$ java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
gregdeMacBook-Pro:~ greg$ 
```

# 设置Eclipse中的JDK版本
偏好设置, `Java` -> `Compiler`
![](/img/eclipse-set-jdk-version.png)
上面这张图是我已经设置1.6了, 现在切换一下1.8看下效果, 右边的1.8的位置是可以选择的, 选择之后下面会出现一个感叹号, 并且有一个`Configure...`可以点击, 点击`Configure...`:
![](/img/eclipse-set-jdk-version2.png)
点击右面的`Add`
![](/img/eclipse-set-jdk-version3.png)
然后下一步
![](/img/eclipse-set-jdk-version4.png)
选择我们下载的`tomcat`的根目录之后就会自动出现一堆包, 可以在`JRE Name`的选项中自己取一个名字, 之后回到`Java` -> `Compiler`, 勾选`JAVA SE 6`即可。

# 配置服务器
偏好设置, `Server` -> `Runtime Environmnets`中:
![](/img/eclipse-tomcat-config.png)
点击`Add`添加服务器版本
![](/img/eclipse-tomcat-config1.png)
选择已有的服务器目录, 没有的可以去`tomcat`的[官网下载](https://tomcat.apache.org/)
![](/img/eclipse-tomcat-config2.png)
我这里选择的是`tomcat 7`作为服务器。

# 添加服务器
在添加服务器之前, 我们先找一下服务器的面板, 有可能默认页面上是没有的, 如果有一些找不到的面板, 可以在`Window` -> `Show View`中找到
![](/img/eclipse-showview.png)
接下来就可以在服务器面板中添加服务器了
![](/img/eclipse-add-serve.png)
最下面的选项可以选择刚才配置好的服务器配置
![](/img/eclipse-add-serve2.png)
这样一来服务器也配置完毕了, 就可以启动项目了。

# 文件模板
![](/img/eclipse-code-template.png)

# 代码提示 & 代码格式化
`Eclipse`的代码提示很怪, 我下的这版默认好像都没有代码提示, 打开这个面板
![](/img/eclipse-code-alert.png)
将最下方的`Auto activation tiggers for Java`改成`.abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`
这样就会有自动提示了。附加代码格式化快捷键:`Command` + `Shift` + `F`。


实在是不想在写这种古老的工程了...赶紧结束吧。