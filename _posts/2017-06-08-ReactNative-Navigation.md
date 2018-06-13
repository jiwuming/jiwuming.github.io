---
title: react-navigation的使用
tags: [ReactNative]
date: 2017-06-08
---

由于 ReactNative 以后不再更新 Navigator, 这一次项目的导航换成了它主推的 react-navigation 记录一下使用方法<!--more-->

[这里是文档地址](https://reactnavigation.org/docs/intro/)

首先, 安装 react-navigation `npm install --save react-navigation`

导航[StackNavigator](https://reactnavigation.org/docs/navigators/stack)

创建一个 Navigaiton: 
```js
import { StackNavigator } from 'react-navigation';
import Main from './Main' 

export default App = StackNavigator({
    Main: {screen: Main}
}, {
    navigationOptions: {
        title: '这是一个title'
    }
})
```
运行 app 我们看到的页面是这个样子的:

![](/img/react-navigation/firstnav.png)

有一个白色的导航栏和一个标题, 其实这个导航栏上面的配置也可以写在各个页面中, 比如
```js
export default class Main extends Component {

    static navigationOptions = {
        title: '一个标题',
    }

    render() {
        return(
            <View style={{flex: 1, backgroundColor: 'skyblue'}}>

            </View>
        )
    }
}
```
这样的写法会覆盖掉第一种写法中的内容, [这里还有很多关于 StackNavigator 的属性](https://reactnavigation.org/docs/navigators/stack), 由于我们一般都是自定义 Navigation 所以一般不多用, 接下来看一下它的跳转传值与回调:

首先是跳转, 我们可以先写一个这样的页面: 

![](/img/react-navigation/fistpage.png)

然后复制一份代码给第二个页面, 稍微改一下文字即可, 再在导航控制器中注册一下页面:

```js
export default App = StackNavigator({
    Main: {screen: Main},
    SecondPage: {screen: SecondPage}
})
```

然后再写出 button 的点击事件
```js
this.props.navigation.navigate('SecondPage')
```

这样就成功的跳转到第二个页面了

![](/img/react-navigation/secondpage.png)

如果想传一些值过去可以这样写:
```js
this.props.navigation.navigate('SecondPage', {'name': '新的标题'})
```
在第二个页面中这样取值:
```js
const {params} = this.props.navigation.state // 这个 params 不是随便写的 它本身是一个属性名称
<Text>
   {params['name']}
</Text>
```

![](/img/react-navigation/newtitle.png)

回调:

和 Navigator 类似, 我们可以现在第一个页面穿进去一个函数, 上面带上我们想要的参数传给第二个页面, 例如:
```js
// 第一个页面
<TouchableOpacity onPress={() => {
    this.props.navigation.navigate('SecondPage', {'handler': (name)=>{
        console.log(name) // 传了一个方法过去, 里面有一个字符串参数需要第二个页面回调过来, 当他回调过来之后会走这个方法
    }})
}} />
```

```js
// 第二个页面
const {params} = this.props.navigation.state
<TouchableOpacity onPress={() => {
    console.log(params.handler)
    params.handler('新的名字6不6') // 这样就把这个字符串传到了上个页面
    this.props.navigation.goBack()
}} />
```

[返回到指定页面的方法](https://github.com/react-community/react-navigation/issues/652)

![](/img/react-navigation/navgoback.png)

其实就是获取到每个页面唯一的 key 就可以了, 这个 key 是一个可变的字符串, 可以通过 `this.props.navigation.state.key` 来获取到, 然后: `this.props.navigation.goBack(key)` 就行了

TabBar [TabNavigator](https://reactnavigation.org/docs/navigators/tab)

也就是 tabbarController 他的写法大概是这样的: 
```js
export default App = TabNavigator({
    Main:{
        screen: StackNavigator({
            Main: {screen: Main},
        })
    },
    SecondPage:{
        screen: StackNavigator({
            SecondPage: {screen: SecondPage},
        })
    },
    ThirdPage:{
        screen: StackNavigator({
            ThirdPage: {screen: ThirdPage},
        })
    }
})
```
效果: 

![](/img/react-navigation/tab.png)

...安卓和 iOS 效果差别还是很大的, 我们可以通过一些属性来使他们看起来类似

```js
{
    tabBarPosition: 'bottom',
    lazy: true,
    tabBarOptions: {
        activeTintColor: '#30aef3',
        inactiveTintColor: '#677887',
        showIcon: true,
        labelStyle: {
            fontSize: 10,
        },
        style: {
            backgroundColor: 'white',
            height: Platform.OS === 'android' ? 55 : 49
        },
        indicatorStyle: {
            backgroundColor: '#ffffff'
        }
    },
    swipeEnabled: false,
    tabBarVisible: true
}
```

![](/img/react-navigation/tabafter.png)

具体属性在上面的链接中可以找到 

关于 tabbar 图标切换的问题, 我是根据 tabBarIcon 的 tintColor 属性去判断的, 因为它貌似只有这一个参数...
```js
export default App = TabNavigator({
    Main:{
        screen: StackNavigator({
            Main: {screen: Main},
        })
    },
    SecondPage:{
        screen: StackNavigator({
            SecondPage: {screen: SecondPage},
        })
    },
    ThirdPage:{
        screen: StackNavigator({
            ThirdPage: {screen: ThirdPage},
        }, {
            navigationOptions: ({navigation}) => ({
                header: null, // 不显示 Navigationbar
                tabBarLabel: '我的', // 标题
                tabBarIcon: ({ tintColor }) => ( // 图片切换
                    <Image source={tintColor === '#30aef3' ? require('../images/icon_tab_wode2.png') : require('../images/icon_tab_wode.png')}/>
                ),
                tabBarVisible: navigation.state.routeName === 'My', // 是否显示 tabbar
            })
        })
    }
})
```


