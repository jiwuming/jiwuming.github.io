---
title: TypeScipt 代码超过 WebStorm 的基准线报错的解决方案
tags: [TypeScript]
date: 2018-12-28
---
Intellij IDEA 的编译器在右面默认都会有一条白色的基准线, 在用 WebStorm 写 TypeScript 的时候 超过这条基准线 TS 会报错, 加上这句注释就好了。(其实也只是在测试数据的时候会出现这么长的代码吧)
```js
// tslint:disable-next-line:max-line-length
var dataSource = balabala.....longlonglong
```