---
title: ReactNative Navigator
tags: [ReactNative]
date: 2017-02-20
---

再来了解一下 RN 中的导航控制器 Navigator 这部分内容包括页面的跳转, 传值, 回调等. 记录一下使用方法.

RN 中的导航控制器有两个, 一个是 Navigator 另一个是 NavigatorIOS, 不过 NavigatorIOS 只能在 iOS 设备上用, 我还是选择了一个通用的 Navigator 来学习

<!--more-->
首先导入 Navigator
```js
import React, {Component} from 'react';
import {
    AppRegistry,
    StyleSheet,
    Text,
    View,
    Navigator,
    TouchableOpacity
} from 'react-native';
```
然后在导出模块中写入 Navigator
```js
 render() {
    return (
        <Navigator style={{flex: 1}} initialRoute={{component: RootScreen}} configureScene={this.configureScene} renderScene={this.renderScene} />
    );
}
```
参数 initialRoute 可以理解成 iOS 中的 RootViewController 用来指定最先显示的场景
参数 configureScene 为场景切换动画, 切换动画有很多:
```js
 Navigator.SceneConfigs.PushFromRight (default)
 Navigator.SceneConfigs.FloatFromRight
 Navigator.SceneConfigs.FloatFromLeft
 Navigator.SceneConfigs.FloatFromBottom
 Navigator.SceneConfigs.FloatFromBottomAndroid
 Navigator.SceneConfigs.FadeAndroid
 Navigator.SceneConfigs.HorizontalSwipeJump
 Navigator.SceneConfigs.HorizontalSwipeJumpFromRight
 Navigator.SceneConfigs.VerticalUpSwipeJump
 Navigator.SceneConfigs.VerticalDownSwipeJump
```
参数 renderScene 可以理解成在 Navigator 初始化以后每个场景可以拿到的对象, 它里面主要包括一个对象和一个导航控制器本身. 这样以后的每个场景就可以使用导航控制器跳转页面和传递参数

导出模块:
```js
export default class Rn_nav extends Component {
    configureScene() {
        return Navigator.SceneConfigs.PushFromRight; // push
    }

    renderScene(route, navigator) {
        return <route.component navigator={navigator} {...route.params} />
    }

    render() {
        return (
            <Navigator style={{flex: 1}} initialRoute={{component: RootScreen}} configureScene={this.configureScene} renderScene={this.renderScene} />
        );
    }
}
```
由于我在 Navigator 中没有找到定义 NavigatorBar 的方法(网上是有介绍的, 但是我写就是各种报错, 提示找不到 Navigator.NavigatorBar 这个属性)我就自己定义了一个 NavigatorBar:

首先, 先写一个 BaseScreen 继承于 Component 然后写一个自定义 View
```js
class BaseScreen extends Component {
    navView (navName, color) {
        return (
            <View style={{height: 64, backgroundColor: color, justifyContent: 'center'}}>
                <Text style={{marginLeft:54, marginRight: 54, textAlign: 'center', color: 'white', fontSize: 16, lineHeight: 64}}>
                    {navName}
                </Text>
            </View>
        )
    }
}
```
然后写一个 RootScreen 继承于 BaseScreen, 在创建视图的时候添加 navView :
```js
class RootScreen extends BaseScreen {
    render() {
        return(
            <View style={{flex: 1, backgroundColor: 'white'}}>
                {this.navView('红色的导航栏', '#bc403a')}
                {this.createUI()}
            </View>
        )
    }

    createUI (){
        return (
            <TouchableOpacity style={{height: 50, marginLeft: 15, marginRight: 15, marginTop: 15, backgroundColor: 'red', justifyContent: 'center'}}>
                <Text style={{color: 'white', textAlign: 'center'}}>点击push</Text>
            </TouchableOpacity>
        )
    }
}
```
这个时候效果大概是这个样子的:

![](/img/IMG-RN-NAV/Rn_nav_init.png)

接下来我们为红色的 Button 添加点击事件:

```js
<TouchableOpacity style={{height: 50, marginLeft: 15, marginRight: 15, marginTop: 15, backgroundColor: 'red', justifyContent: 'center'}} onPress={() => this.pushAction()}> // 别忘了使用 () => {...} 否则 this 的指向不对
    <Text style={{color: 'white', textAlign: 'center'}}>点击push</Text>
}
</TouchableOpacity>

// 点击事件 push 方法
pushAction() {
   this.props.navigator.push({
       component: SecondScreen // 跳转到了第二个场景
   })
}

// 如果需调用返回方法 像下面这样
popAction() {
    this.props.navigator.pop();
}
```
第二个场景的代码就不贴了, 和第一个场景一样就好, 可以稍微改一些文字之类的

这个时候基本上就能完成 导航控制器的跳转了, 那接下来看一下传值和回调:

我们在注册导航控制器的时候有个 renderScene :
```js
renderScene(route, navigator) {
    return <route.component navigator={navigator} {...route.params} />
}
```
后半部分 `{...route.params}` 中的 `params` 是可以随便起名的, 这个就是你传值时所需的参数, 一般的 push 传值可以写成这样:
```js
pushAction() {
   this.props.navigator.push({
       component: SecondScreen, // 跳转到第二个场景
       params: { // 注册导航的时候写的传值参数名字 params
           navTitle: '测试传值'
       },
   })
}
```
这样我们就传过去了一个 navTitle 对应的字符串, 在 SecondScreen 这个场景中 我们可以直接使用 `this.props.navTitle` 来使用传过来的值, 比如这是我在第二个场景中的代码:
```js
render() {
    return(
        <View style={{flex: 1, backgroundColor: 'white'}}>
            {this.navView(this.props.navTitle, 'green')}
            {this.createUI()}
        </View>
    )
}
```
效果是这样的: 

![](/img/IMG-RN-NAV/Rn_nav_getValue.png)

还是比较简单的吧

看完传值看回调, RN 中的回调也非常简单, 如果你想在 pop 的时候执行一些方法, 只需要在第一个场景传一个方法到第二个场景, 然后在第二个场景 pop 之前调用这个方法就行了

代码如下:
```js
// 第一个场景中的 push 方法
 pushAction() {
   this.props.navigator.push({
       component: SecondScreen,
       params: {
           navTitle: '测试传值',
           getData: (person) => { // 传过去了一个方法 带了一个 person 参数
               // 回调成功之后 打印 person 的值
               console.log(person)
           }
       },
   })
}

// 第二个场景中的 pop 方法
popAction() {
    this.props.getData({ // 执行传过来的方法 把值传给第一个场景
        person: {
            name: '回调',
            age: '0',
            gender: 'm'
        }
    });
    this.props.navigator.pop(); // 之后调用 pop 方法回到第一个场景
}
```

![](/img/IMG-RN-NAV/Rn_nav_chromeLog.png)

这样就回调成功了
