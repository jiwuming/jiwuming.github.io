---
title: Tomcat 启动报错 One or more listeners failed to start. Full details will be found in the appropriate container log file
tags: [Tomcat, Intellij IDEA]
date: 2018-06-04
---
这个错误产生于我新写的一个SSM的项目, 当时只配置了spring的一些配置以及log4j和web.xml, 然后想启动tomcat试一下, 结果发生了以下的错误
```bash
04-Jun-2018 16:20:16.942 严重 [RMI TCP Connection(2)-127.0.0.1] org.apache.catalina.core.StandardContext.startInternal One or more listeners failed to start. Full details will be found in the appropriate container log file

04-Jun-2018 16:20:16.961 严重 [RMI TCP Connection(2)-127.0.0.1] org.apache.catalina.core.StandardContext.startInternal Context [] startup failed due to previous errors

[2018-06-04 04:20:16,986] Artifact :war exploded: Error during artifact deployment. See server log for details.
```
大概意思是说一个或者多个 listener 启动失败, 然后让我去 web 容器下面去寻找错误日志? 有点拐弯抹角的感觉啊...
![](/img/???.jpeg)
<!-- more -->
那就去找吧, 首先打开我本地的 tomcat 的 work 文件夹, 这里面是没有任何文件的。那么项目到底发布到哪里去了呢? 后来我在网上查了一下, 目录是这个样子的
```bash
/Users/你的用户名/Library/Caches/IntelliJIdea2018.1
```
这个文件夹下面有很多项目, 是以你 tomcat 的名字 + 下划线 + 你的工程的名字命名的。找到刚才写的工程, 工程下面会有三个文件夹:
![](/img/tomcatloglocation.png)
选中的localhost文件中会有我们刚才报错的具体记录:
![](/img/dbpropertiesnotfound.png)
这已经很明显了, 这个 db.properties 的文件找不到所以报了错误, 在看一眼代码...
```bash
<context:property-placeholder location="db.properties" />
```
居然是前面忘写了 classpath...
```bash
<context:property-placeholder location="classpath:db.properties" />
```
改过之后就可以正常运行了。也算是顺便复习了一下`classpath:`__加载编译之后的classes目录下的文件__