---
title: ReactNative 入门 Demo
tags: [ReactNative]
date: 2017-02-08
---
接着上次的环境搭建完成之后, 我们就可以来写写简单的 RN 代码了, 这次的 Demo 中包括 RN 的布局, props, state 等一些东西, 记录一下使用过程

首先来看一下效果:
![](/img/IMG-RN-CARD/Rn_card_result.png)
大概就是这样的一个卡片效果

<!--more-->
首先, 新建一个工程, 把 `index.ios.js` 中自带的内容都删掉, 只剩一个 View 然后我们在建一个 Card 的 Class 然后把这个 Card 添加就可以了, 实现如下:  

```JavaScript
class Card extends Component {
    render() {
        return (
            <View>
                
            </View>
        );
    }
}

export default class Rn_card extends Component {
    render() {
        return (
            // 内部样式: flex = 1 撑满整个空间 即这个底层的 View 是屏幕的大小
            <View style={{flex: 1}}>
                <Card>

                </Card>
            </View>
        );
    }
}
```
RN 的布局方式和 CSS 布局的方式基本一致, 上手不是太困难, 我们先来把主要的背景写出来, 随便写几个值就好:
```JavaScript
class Card extends Component {
    render() {
        return (
            <View style={styles.mainContainer}>

            </View>
        );
    }
}

const styles = StyleSheet.create({
    mainContainer: {
        height: 200,
        backgroundColor: 'skyblue',
        marginTop: 20,
    }
});
```
效果:

![](/img/IMG-RN-CARD/Rn_card_background.png)

恩, 好的这样就有一个背景了

其实上面的效果图片我们可以先把他拆成两部分, 即头像和个人信息为一部分, 下面的个人简介为一部分, 所以最底部的 View 定下来了以后, 我们把这两部分写出来:
```JavaScript
class Card extends Component {
    render() {
        return (
            <View style={styles.mainContainer}>
                <View style={styles.cardInfoContainer}>

                </View>
                <View style={styles.cardDecContainer}>

                </View>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    mainContainer: {
        height: 200,
        backgroundColor: 'skyblue',
        marginTop: 20,
    },
    cardInfoContainer: {
        flex: 4,
        backgroundColor: 'red',
        marginLeft:15,
        marginRight:15
    },
    cardDecContainer: {
        flex: 3,
        backgroundColor: 'yellow',
        marginLeft:15,
        marginRight:15
    }
});
```
样式中的属性都很直观, 至于这个 flex 指的是一个弹性宽高, 在其父视图有一定的大小的前提下(__如果这个视图有一定的 width 或 height 或 flex 否则无效果__), flex 可以指定一个子视图占用父视图的空间的百分比, 并且这个 flex 的值只能为整数, 现在我们看到的效果大概是这样的:

![](/img/IMG-RN-CARD/Rn_card_topBottom.png)

从这个图中我们可以了解到, RN 默认的布局方式是采用纵向布局的, 现在我们要在红色的部分写一个头像和一些跟人信息, 同样把个人信息和头像拆分成两部分, 然后红色 View 采用横向布局 个人信息的三列采用纵向布局
```JavaScript
class Card extends Component {
    render() {
        return (
            <View style={styles.mainContainer}>
                <View style={styles.cardInfoContainer}>
                    <Image style={styles.avatar}>

                    </Image>
                    <View style={styles.cardPersonInfo}>
                        <Text style={styles.cardName}>

                        </Text>
                        <Text style={styles.cardGenderAndAge}>

                        </Text>
                        <Text style={styles.cardGenderAndAge}>

                        </Text>
                    </View>
                </View>
                <View style={styles.cardDecContainer}>

                </View>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    mainContainer: {
        height: 200,
        backgroundColor: 'skyblue',
        marginTop: 20,
    },
    cardInfoContainer: {
        flex: 4,
        backgroundColor: 'red',
        marginLeft:15,
        marginRight:15,
        // 采用横向布局 (纵向布局 'column')
        flexDirection: 'row'
    },
    cardDecContainer: {
        flex: 3,
        backgroundColor: 'yellow',
        marginLeft:15,
        marginRight:15
    },
    avatar: {
        width: 80,
        height: 80,
        marginTop: 15,
        backgroundColor: 'gray'
    },
    cardPersonInfo: {
        // 这里有个坑, 前面头像的大小是固定的, 如果这里不用 flex = 1 填满剩下的空间或者不写绝对大小的话 这个视图不会显示 
        flex: 1,
        marginTop: 15,
        height: 80,
        marginLeft: 10,
        backgroundColor: 'orange'
    },
    cardName: {
        marginTop: 0,
        height: 20,
        backgroundColor: 'green'
    },
    cardGenderAndAge: {
        marginTop: 10,
        height: 20,
        backgroundColor: 'green'
    }
});
```
如果直接运行的话是会报错的, 因为没有导入 Image 的包, 所以我们在头部导入一下:
```JavaScript
import React, {Component} from 'react';
import {
    AppRegistry,
    StyleSheet,
    Text,
    View,
    Image // 导入 Image
} from 'react-native';
```
好了, 现在我们的布局基本上就写好了, 效果如图:

![](/img/IMG-RN-CARD/Rn_card_boxResult.png)

恩, 看起来很吃藕, 接下来我们搞一些文字上去并把这些背景颜色都去掉:
```JavaScript
class Card extends Component {
    render() {
        return (
            <View style={styles.mainContainer}>
                <View style={styles.cardInfoContainer}>
                    <Image style={styles.avatar}>

                    </Image>
                    <View style={styles.cardPersonInfo}>
                        <Text style={styles.cardName}>
                            姓名:
                        </Text>
                        <Text style={styles.cardGenderAndAge}>
                            性别:
                        </Text>
                        <Text style={styles.cardGenderAndAge}>
                            年龄:
                        </Text>
                    </View>
                </View>
                <View style={styles.cardDecContainer}>
                    <Text>
                        简介:
                    </Text>
                </View>
            </View>
        );
    }
}
```
效果:

![](/img/IMG-RN-CARD/Rn_card_noData.png)

其实我们可以直接把名字之类的写上去, 但是实际开发中, 这些数据都是活的, 这时候据需要我们来定义一些 __props(属性)__ 也就是参数, 比如这样:

```JavaScript
class Card extends Component {
    render() {
        return (
            <View style={styles.mainContainer}>
                <View style={styles.cardInfoContainer}>
                    <Image style={styles.avatar} source={{uri : this.props.card['avatar']}}>

                    </Image>
                    <View style={styles.cardPersonInfo}>
                        <Text style={styles.cardName}>
                            姓名: {this.props.card['name']}
                        </Text>
                        <Text style={styles.cardGenderAndAge}>
                            性别: {this.props.card['gender']}
                        </Text>
                        <Text style={styles.cardGenderAndAge}>
                            年龄: {this.props.card['age']}
                        </Text>
                    </View>
                </View>
                <View style={styles.cardDecContainer}>
                    <Text>
                        简介: {this.props.card['dec']}
                    </Text>
                </View>
            </View>
        );
    }
}
```
这里相当于定义了一个 card 对象, 这个 card 对象里面有一些键值对, 我们只需要从外部传入一个 card 对象, 这个组件就能显示完整的信息了, 比如:
```JavaScript
export default class Rn_card extends Component {
    render() {
        // 图片为 http 的链接的话不要忘了在 info.plist 里面设置 NSAllowsArbitraryLoads 为 true
        let data = {
            avatar: 'https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=1691037026,968218084&fm=23&gp=0.jpg',
            name: '赵日天',
            gender: '男',
            age: '12',
            dec: '我赵日天第一个不服',
        };
        return (
            <View style={{flex: 1}}>
                <Card card={data}>

                </Card>
            </View>
        );
    }
}
```
这样我们就能看到理想的效果啦:

![](/img/IMG-RN-CARD/Rn_card_result.png)

但是这并没有完全结束, 因为我们平时的数据都是从网上请求下来的, 所以这些数据都是异步的, 不能直接赋值给 View 那么如何在得到数据之后再次给 View 赋值呢? 这个时候我们就需要用到__state(状态)__

每个 Class 都有一个 constructor 的构造方法, 它可以直接打出来
```JavaScript
    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {};
    }
```
state的特点是: state中的数据是可变的, 一个类中一旦调用了 this.setState() 那么就会引起 render 的重新渲染, 来达到更新页面的目的

我们可以用 setTimeOut() 来模仿网络请求, 来写一个延迟三秒改变名片的功能:
```JavaScript
export default class Rn_card extends Component {
    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {
            data: { // 初始化数据
                avatar: 'https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=1691037026,968218084&fm=23&gp=0.jpg',
                name: '赵日天',
                gender: '男',
                age: '12',
                dec: '我赵日天第一个不服',
            }
        };
        // 在ES6中，如果在自定义的函数里使用了 this 关键字，则需要对其进行“绑定”操作，否则 this 的指向不对
        // 像下面这行代码一样，在 constructor 中使用 bind 是其中一种做法（还有一些其他做法，如使用箭头函数等）
        this.loadData = this.loadData.bind(this);
    }
       
     // 已经加载虚拟 DOM 整个生命周期只会运行一次 通常用于完成异步网络请求
    componentDidMount() {
        this.loadData(); 
    }

    loadData() {
        // 模仿网络请求
        setTimeout(() => { // 函数必须用 ES6 的写法 否则在写 this.setState 时会提示没有定义的方法
            let newData = { // 模仿网络请求得到的新对象
                avatar: 'https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=1691037026,968218084&fm=23&gp=0.jpg',
                name: '叶良辰',
                gender: '男',
                age: '12',
                dec: '良辰有一万种方法让你待不下去',
            };
            this.setState({ // 注意 为了保证this在调用时仍然指向当前组件 需要在 constructor 中做绑定操作
                data: newData // 赋值 新的对象
            });
        }, 3000) // 延迟三秒
    }

    render() {
        return (
            <View style={{flex: 1}}>
                // 接收 state 中的 data
                <Card card={this.state.data}> 

                </Card>
            </View>
        );
    }
}
```
这样, 三秒之后, 我们就会看到变化了:

![](/img/IMG-RN-CARD/Rn_card_ylc.png)
