---
title: java.util.concurrent.ExecutionException： org.apache.catalina.LifecycleException：Failed to start component XXX
tags: [Java, Maven]
date: 2018-05-22
---
由Maven搭建的Spring+Struts2开发环境, 用Maven引入struts2-spring-plugin, 换了一台电脑启动Tomcat的时候报的错, 但是编译的时候并不会报错, 这个是由于Maven仓库缺失包导致的, 把另一台电脑的Maven仓库复制一份拿过来就好了...

Mac Maven仓库的默认路径
```bash
/Users/username/.m2/repository
```

