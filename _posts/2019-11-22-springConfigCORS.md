---
title: Zuul 配置跨域和 HTTP 请求状态码为 0 的解决
tags: [Java, Spring, iOS]
date: 2019-11-22
---
今天遇到了`ionic`在`iOS`端请求数据`HTTP`状态码为0的情况, 记录一下解决过程。
<!-- more -->
首先是一个跨域请求问题, (后端)主要是由两部分组成, 一部分是对`OPTION`请求的手动处理, 另一部分则是设置跨域请求头。我们可以通过两个网关过滤器来解决这个问题。
```java
// 手动给与OPTION请求状态
@Component
public class preFilter extends ZuulFilter {
   
    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 1;
    }
    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest req = ctx.getRequest();
        String methodName = req.getMethod();
        if(methodName.equals("OPTIONS")){
            ctx.setResponseStatusCode(200);
            ctx.setResponseBody("");
            ctx.setSendZuulResponse(false);
        }
        return null;
    }
}
```
因为我们的请求都是`POST`请求方式, 所以设置跨域请求头的时候在这一步:
```java
@Component
public class AccessCrossFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "post";
    }

    @Override
    public int filterOrder() {
        return 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest req = ctx.getRequest();
        String url = req.getHeader("Origin");
        // 这里最好不要写 * 有的浏览器还是会有问题, 所以从 Origin 取出来 赋上去
        ctx.addZuulResponseHeader("Access-Control-Allow-Origin",url);
        ctx.addZuulResponseHeader("Access-Control-Allow-Credentials","true");
        ctx.addZuulResponseHeader("Access-Control-Allow-Methods","GET,PUT,POST,OPTIONS");
        ctx.addZuulResponseHeader("Access-Control-Allow-Headers","Content-Type,自定义的请求头,自定义的请求头...");
        return null;
    }
}
```
~~过滤器配置到这里就结束了, 但是一开始的时候会出现请求立刻失败的情况, 加上了`ribbon`配置可解决此问题~~(不过后来这个问题又消失了, 待验证)
```yml
zuul:
  routes:
    user:
      path: /api-user/**
      serviceId: service-user
    common:
      path: /api-common/**
      serviceId: service-common
  host:
    connect-timeout-millis: 20000
    socket-timeout-millis: 60000

ribbon:
  ReadTimeout: 60000
  ConnectTimeout: 60000
```
至此所有的跨域配置都已经完成, `pc`端和`Android`端可以正常请求数据, 唯独在`iOS`上不能正常请求数据, 请求可以发送, 但HTTP的状态码是0, 后来我在本地调试了一下发现请求过网关的时候出线了以下错误:
```bash
com.netflix.zuul.exception.ZuulException: Forwarding error
```
然后下面还有一行
```bash
Load balancer does not have available server for client 服务名
```
网上解决这个问题有很多种办法。。。但是我最终的到有效的解决办法是下面两个, 一个是设置`ribbon`:
```yml
ribbon:
  ReadTimeout: 60000
  ConnectTimeout: 60000
```
还需要引入一个依赖:
```xml
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```
上面的两个缺一不可, 否则网关层还是会报错的。