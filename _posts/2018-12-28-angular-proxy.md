---
title: Angular 配置跨域请求
tags: [Angular]
date: 2018-12-28
---
Web 开发不像 App 开发那样想请求哪就请求哪。处于安全问题考虑浏览器禁止了这一点。但是在开发过程中又不可避免的去进行跨域请求, 记录一下 Angular 配置跨域请求的过程。
<!-- more -->
首先在`package.json`目录下新建一个`json`文件`proxydir.png`:
![](/img/proxydir.png)
这个文件的内容是:
```js
// 比如我现在想请求的接口是这个:
// let urlString = "http://c.3g.163.com/recommend/getChanListNews?channel=T1457068979049&passport=zvZe9MQ7cxTSrcWyjXHlNmWlYJxG7i%2FExMsC%2FIX1t2E%3D&devId=tYiXx8W73%2BGGquuNxF8c8PkrobMEegFcSVAs152mOWH7Vqve0omfVq78wYMgkonU&size=40&version=9.0&spever=false&net=wifi&lat=&lon=&ts=1464162516&sign=rC4vOZ5ChMjNUe8YKmdGHE3VIm%2BZjtNzO49jFFbSUJx48ErR02zJ6%2FKXOnxX046I&encryption=1&canal=appstore"
// 所有以 /recommend 开头的 api 都会被反向代理到 http://c.3g.163.com 上面
{
    "/recommend":{
        "target": "http://c.3g.163.com",
        "secure": false,
        "logLevel": "debug",
        "changeOrigin": true
    }
}
```
接下来配置`package.json`找到`script`这一项:
```js
// --host 192.168.2.100 是我本机的ip地址 这样写是为了其他局域网内的设备 比如手机 可以访问我的页面
// --proxy-config proxy.config.json 添加反向代理设置
"scripts": {
    "ng": "ng",
    "start": "ng serve --host 192.168.2.100 --proxy-config proxy.config.json",
    "build": "ng build",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e"
},
```
然后配置`angular.json`找到`serve`这一项:
```js
// 在options中加入 "proxyConfig": "proxy.config.json" 
"serve": {
    "builder": "@angular-devkit/build-angular:dev-server",
    "options": {
        "browserTarget": "wm:build",
        "proxyConfig": "proxy.config.json"
    },
    "configurations": {
        "production": {
            "browserTarget": "wm:build:production"
        }
    }
},
```
然后我们在请求的时候就可以这样写了:
```js
// listUrl 的内容就是 /recommend/getChanListNews?channel=T1457068979049&passport=zvZe9MQ7cxTSrcWyjXHlNmWlYJxG7i%2FExMsC%2FIX1t2E%3D&devId=tYiXx8W73%2BGGquuNxF8c8PkrobMEegFcSVAs152mOWH7Vqve0omfVq78wYMgkonU&size=40&version=9.0&spever=false&net=wifi&lat=&lon=&ts=1464162516&sign=rC4vOZ5ChMjNUe8YKmdGHE3VIm%2BZjtNzO49jFFbSUJx48ErR02zJ6%2FKXOnxX046I&encryption=1&canal=appstore
constructor(private http: HttpClient) {
    // 注册请求
    this.datas = this.http.get(new Api().listUrl);
}
 ngOnInit() {
    this.datas.subscribe((data) => {
        console.log(data);
    });
 }
```
可以看到请求成功了:
![](/img/proxysuccess.png)