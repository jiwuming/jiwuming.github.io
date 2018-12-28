---
title: Angular 使用第三方的 js 库
tags: [Angular, TypeScript]
date: 2018-12-28
---
由于 Angular 使用 TS 编程, 所使用到的库自然也是支持 TS 的, 但是一些特殊情况下我们想使用其他的 js 框架, 那这个时候怎么办呢?
<!-- more -->
这里还分两种情况, 有一些第三方是支持 TS 的, 比如 Swiper, 那么引用 Swiper 的过程就是这样的:
```bash
// 首先通过 npm 安装 Swiper
npm install swiper --save
// 然后在 `angular.json` 中引用它的样式和 js 
// 这是目录
// architect -> build -> options
"styles": [
    "src/base/styles.scss",
    "./node_modules/swiper/dist/css/swiper.css",
],
"scripts": [
    "./node_modules/swiper/dist/js/swiper.js",
]
// 然后安装模组定义档
npm install @types/swiper --save
// 配置`tsconfig.json`一般这个配置都自带了
"typeRoots": [
    "node_modules/@types"
],
// 然后配置`tsconfig.app.json`这个东西在src目录下
// 配置节点
"compilerOptions": {
    "outDir": "../out-tsc/app",
    "types": [
        "swiper"
    ]
},
// 然后在你想用的类里面导入就好了
import Swiper from 'swiper';
```
但是有些情况下, 某些库是没有`npm install @types/frameworkname --save`这种东西的, 而 TS 又不能直接的调用 js。对于这种情况可以采用下面的方式:
```bash
// 首先安装和上面的一样 npm install 然后引入样式和js
// 然后在src目录下, 新建 `typings.d.ts`
// 然后写入内容 这里以 MiniRefresh 为例 一定要写运用的第三方类的名字
declare var MiniRefresh: any;
// 然后在`tsconfig.json`中 compilerOptions 节点下加入:
"allowJs": true
// 接下来的代码就可以这么写:
// 定义刷新控件属性
private refreshControl: any;
// 实现方法
this.refreshControl = new MiniRefresh({
    container: '#minirefresh',
    down: {
        callback: () => {
            this.refreshControl.endDownLoading();
    },
    up: {
        callback: () => {
            // 注意，由于默认情况是开启满屏自动加载的，所以请求失败时，请务必endUpLoading(true)，防止无限请求
            this.refreshControl.endUpLoading(true);
        }
    }
});
```
这样配置之后 TS 就能使用 js 的类库了。