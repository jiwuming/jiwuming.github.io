---
title: IntelliJ IDEA 编辑注解模板
tags: [IntelliJ IDEA, Java]
date: 2018-05-15
---
xml中编辑部署servlet的方式是这样的
```xml
<servlet>
    <servlet-name>testservlet</servlet-name>
    <servlet-class>testpackage.Testservlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>testservlet</servlet-name>
    <url-pattern>/testservlet</url-pattern>
</servlet-mapping>
```
每次都这样部署很麻烦, 基本上都会使用注解去部署, 但是注解默认参数只带一个ServletName, 如果希望每次创建文件把其他参数也带出来可以像下面这样编辑模板:
![](/img/autoServlet.png)
<!-- more -->
也就是在原来的基础上加了一个参数:
```java
@javax.servlet.annotation.WebServlet(name = "${Entity_Name}", urlPatterns="/${Entity_Name}")
```
那么这样他帮我们创建的Servlet是这样的
```java
package testpackage;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "TestServlet", urlPatterns = "/Testservlet")
public class Testservlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
    }
}
```