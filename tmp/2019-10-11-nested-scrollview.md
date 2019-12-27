---
title: 关于 iOS UIScrollView 嵌套的一些问题
tags: [iOS]
date: 2019-10-11
---
最近看了一下有关`UIScrollView`嵌套的一些页面，发现了一些问题。而且各个厂商实现的方式好像还有区别，记录一下这种页面的实现过程。
<!-- more -->

### 嵌套scrollView在市场软件中的应用
现在一般的app都会有以下这种设计，上下滑动时有一部分视图会吸附到顶部，并且下面的视图可以左右滑动，我分别截取了bilibili个人页，抖音个人页，淘宝主页，网易云音乐歌手页，美团首页进入的外卖页作为测试

滑动前：
![](/img/nested-scrollview/normal.png)

滑动后顶部会吸附一个视图：
![](/img/nested-scrollview/top.png)

下拉回来会回到滑动前的那种状态，也就是到达了指定的区域，顶部吸附的视图也就随之下来了。

### 最开始的思路
首先布局还是挺简单的，无非就是一个`UIScrollView`加一个`childViewController`的组合
![](/img/nested-scrollview/first-layout.png)

整个页面是一个`UIScrollView`，这个`UIScrollView`上部分是我们自定义的一个视图。下部分是一个`ViewPager`其封装原理无非也就是一个自定义视图（顶部的吸附视图）加上一个左右滑动的`UICollectionView`，然后在`UICollectionViewCell`上放置`childViewController`。然后在`childViewController`上面添加`UITableView`。只不过这个`viewPager`的高度要比最外层的`UIScrollView`高出一个个红色的自定义头部那一部分的高度，用来达到最外层滚动的目的。

这样我们就有了两个滚动视图，我们可以通过代理或者block把最里面的`UITableView`的滚动方法`- (void)scrollViewDidScroll:(UIScrollView *)scrollView`获取到，然后判断是否滚动到紫色区域的头部（临界值）就可以了。


。网上已经有很多的实现方式，其核心就是嵌套`UIScrollView`，然后启动多重手势识别代理，根据两层`UIScrollView`的`- (void)scrollViewDidScroll:(UIScrollView *)scrollView`这个代理去判断是否去滚动当前的`UIScrollView`即可