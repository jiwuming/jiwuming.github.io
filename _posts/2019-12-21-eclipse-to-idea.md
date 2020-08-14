---
title: Eclipse项目导入到Intellij IDEA
tags: [Java, Eclipse, IDEA]
date: 2019-12-23
---
又是踩坑的一天。。


因为实在受不了`Eclipse`的开发方式，所以决定还是使用`IDEA`来进行开发。

首先用`IDEA`打开项目，然后点击右上角带有蓝色脚标的按钮，也就是`Project Structure`。
将`Java`环境切换到1.6，`Java`多版本环境配置方式已经在`Mac Eclipse`中写过了。
![](/img/idea-proj.png)

点击左侧的`Modules`然后点击`Dependencies`将报红的`jar`包删掉。
![](/img/idea-proj-modules.png)

然后点击`Factes`点击加号添加一个`web`
![](/img/idea-proj-facets.png)

这时候还需要设置一些东西，将`Deployment Desciptor`中的路径改为工程目录中`WebContent/WEB-INF`下的`web.xml`的路径。
然后将`Web Resource Dictionaries`的目录设置成工程中`WebContent`的目录，右侧的Path不用改使用`/`就好，最后点击右下角的`Create Artifact`。

之后会跳转这里
![](/img/idea-proj-artifacts.png)
这里只有一步，点击右下角的`Fix`选择`Add all missing dependencies...`

到这里工程配置基本上算是完成了，然后配置好`Tomcat`（这个没什么特殊的操作就不说了），先运行一下，工程中可能会有缺少`jar`包的情况，这时候打开`Project Structure`选择`Modules`，`Dependencies`选择加号
![](/img/idea-proj-addde.png)
选择添加`jar`包的选项也就是第一个，然后把工程中依赖的`jar`包全选然后添加进去，不要选择添加目录。

然后选择`Artifact`点击右下角的`Fix`选择`Add all missing dependencies...`

之后再运行`Tomcat`即可。