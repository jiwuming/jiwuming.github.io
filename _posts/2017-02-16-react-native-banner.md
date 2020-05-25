---
title: ReactNative ScrollView
tags: [ReactNative]
date: 2017-02-16
---

这次做的 RN 的 Demo 是一个轮播图, 其中主要是 ScrollView 的使用, 模块导入, 样式封装等, 记录一下学习过程<!--more-->

首先来看效果:

![](/img/IMG-RN-BANNER/Rn_bannerGif.gif)

就是两张图片实现了一个轮播效果, 主要用到的控件是 scrollView, 首先先创建一个 js 文件, 把这个轮播图单独拿出来:

![](/img/IMG-RN-BANNER/Rn_bannerFile.png)

然后导入一些必要的包:
```js
import React, {Component} from 'react';
import {
    StyleSheet,
    View,
    Image,
    ScrollView,
    Dimensions
} from 'react-native';
```
接下来我们就可以在导出模块里面写代码了

我们在封装一个控件之前会先写好一部分内部的逻辑处理, 然后把数据接口和需要用户改变的一些变量暴露出来以供调用, 那么在我上面实现的栗子中, 我们只需要把数据接口暴露出来就可以了, 但是我想保留一部分必要样式, 让外部去写一些其他的样式, 这种情况应该怎么做呢? 如下
```js
 <View style={[styles.privateStyle, this.props.mainStyle]} />
```
这里面 privateStyle 是封装的控件内部保留的样式, 而属性 mainStyle 则是提供给外部使用的 style

这个是我的 scrollView:
```js
 <ScrollView style={{flex: 1}}
             ref='scrollView' // 标记这个控件 在其他位置可以拿到 scrollView
             horizontal={true} // 横向显示内容
             pagingEnabled={true} // 翻页式
             contentOffset={{x: screenWidth, y:0}} // 初始偏移量
             onMomentumScrollEnd={this.scrollViewEndScroll}> // 滑动结束调用事件
             {viewArray} // scrollView 内的内容
</ScrollView>
```
前面几个属性没啥说的, 中间的那个 {viewArray} 我的一个 UI 数组, 它的写法是这样的:
```js
for (let i = 0; i < itermsArray.length; i++) {
    viewArray.push(
        <View style={styles.listCellStyle} key={i}>
            <Image style={styles.listCellImageStyle} source={{uri: itermsArray[i]}}/>
        </View>
    )
}
```
其中 itermsArray 是经过我处理的一个图片数组, 也可以说是数据源吧, 我们可以通过这种方式去循环创建视图, 这也是 JSX 的特性

完整的控件代码:
```js
// 全局变量:
let screenWidth = Dimensions.get('window').width;
let itermsArray = [];
export default class Banner extends Component {
    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {};
        // 由于使用了 scrollView 中使用了 ref 属性, 所以要绑定方法
        this.scrollViewEndScroll = this.scrollViewEndScroll.bind(this);
    }
    render() {
        // 这里简单的处理了一下数据, 无限轮播的原理大概就是把最后一张图片添加到数据数组第一位,
        // 把第一张图片添加到数据数组最后一位, 然后在判断偏移量就可以了
        // dataSourceArray 由外部传入
        let firstImg = this.props.dataSourceArray[0];
        let lastImg = this.props.dataSourceArray[this.props.dataSourceArray.length - 1];
        itermsArray.push(lastImg);
        itermsArray = itermsArray.concat(this.props.dataSourceArray);
        itermsArray.push(firstImg);
        let viewArray = [];
        // 创建 scrollView 内的 imageView
        for (let i = 0; i < itermsArray.length; i++) {
            viewArray.push(
                <View style={styles.listCellStyle} key={i}>
                    <Image style={styles.listCellImageStyle} source={{uri: itermsArray[i]}}/>
                </View>
            )
        }
        return (
        // 渲染页面
            <View style={[styles.privateStyle, this.props.mainStyle]}>
                <ScrollView style={{flex: 1}}
                            ref='scrollView'
                            horizontal={true}
                            pagingEnabled={true}
                            contentOffset={{x: screenWidth, y:0}}
                            onMomentumScrollEnd={this.scrollViewEndScroll}>
                    {viewArray}
                </ScrollView>
            </View>
        );
    }
    // scrollView 滑动结束方法
    scrollViewEndScroll(Event) {
        console.log(Event.nativeEvent.contentOffset);
        let scrollView = this.refs.scrollView; // 通过 ref 找到 scrollView
        if (Event.nativeEvent.contentOffset.x >= (this.props.dataSourceArray.length + 1) * screenWidth) {
            scrollView.scrollTo({x: screenWidth, y:0, animated: false});
        }
        else if (Event.nativeEvent.contentOffset.x <= 0) {
            scrollView.scrollTo({x: screenWidth * this.props.dataSourceArray.length , y:0, animated: false});
        }
    }
}
```
样式很简单, 就不贴了, 那我们在别的类里面如何使用这个类呢? 来看一下如何导入这个组件到其他文件中:
```js
import Banner from './Banner'
```
`import` 类名  `from` './...' (这个...是会自动提示的)

使用 Banner:
```js
export default class Rn_banner extends Component {
    render() {
        let array = [
            'https://timgsa.baidu.com/timg?image&quality=80&size=b10000_10000&sec=1487837472&di=1f3480ba69bc2851a5d46c2aadf981e9&src=http://imgsrc.baidu.com/forum/w=580/sign=9c740633b1003af34dbadc68052bc619/762778f0f736afc3d396d95fb419ebc4b64512de.jpg',
            'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1487847517873&di=39f7cab76abc9c5a6733c12624e04624&imgtype=0&src=http%3A%2F%2Fimgsrc.baidu.com%2Fforum%2Fw%3D580%2Fsign%3Dff39ddaa3dc79f3d8fe1e4388aa1cdbc%2F44b85666d01609249d1c3a4ed20735fae6cd346b.jpg'
        ];
        return (
            <View style={styles.container}>
                <Banner mainStyle={styles.bannerStyle} dataSourceArray = {array}>

                </Banner>
            </View>
        );
    }
}
```
