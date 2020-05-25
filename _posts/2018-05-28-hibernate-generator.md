---
title: Hibernate的逆向工程操作
tags: [Maven, Intellij IDEA]
date: 2018-06-01
---
写完了 Mybaits 的逆向操作也来写一下 Hibernate 的逆向工程操作吧, 
虽然这个库以后几乎不能用到了...

首先创建一个工程, 导入 Hibernate jar 包和注解包, 以及编写 hibernate.cfg.xml 文件
![](/img/hibernateStart.png)
```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.2.12.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate.javax.persistence</groupId>
    <artifactId>hibernate-jpa-2.1-api</artifactId>
    <version>1.0.0.Final</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.38</version>
</dependency>
```
<!-- more -->
然后连接数据库
![](/img/connectSql.png)
接下来完善数据库信息然后点击ok
![](/img/h-mysqlInf.png);
然后看工程的左下角:
![](/img/entry.png)
如果这个选项没有的话, 也可以在 View->Tool Windows 中找到
如果 View->Tool Windows 也没有的话, 右键工程总目录文件夹, 然后有个 Add Framework Support 点击
![](/img/addframeworksupport.png)
之后ok就可以了, 如果有这个按钮就走下面的操作:
点击了 By Database Schema 之后会出现下面的页面:
![](/img/choosetable.png)
选择好包路径之后, 这个时候选择你想要生成的表点击确定即可生成实体类了, 并且会自动帮你在 hibernate.cfg.xml 中配置好mapping。
![](/img/resultentity.png)
![](/img/resultmapping.png)
