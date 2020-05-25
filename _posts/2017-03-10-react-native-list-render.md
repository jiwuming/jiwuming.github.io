---
title: ReactNative ListView 更改 renderRow 样式的问题
tags: [ReactNative]
date: 2017-03-10
---

最近开始用 RN 来写项目了, 记录一下今天使用 ListView 中的一个坑

引出这个问题的功能也很简单, 就是一个带有分页功能的 ListView , 然后我点击 ListView 其中的一个 cell , 就让这个 cell 的背景颜色改变(项目中的功能是单选or多选cell)

<!--more-->

![](/img/ReactNative-ListView-Keng/demo.png)

于是我先写了一个 ListView :
```js
// UI
 <ListView style={styles.listViewStyle}
           dataSource={this.state.dataSource}
           renderRow={this.renderRow}
 /}>
// 初始化 dataSource
constructor(props) {
    super(props);
    // 初始状态
    this.state = {
        dataSource: new ListView.DataSource({
            rowHasChanged: (r1, r2) => r1 !== r2
        }),
        dataArray: []
    };
}
// renderRow
renderRow(data, sectionID, rowID) {
    return(
        <TouchableOpacity style={{backgroundColor: data['color']}} activeOpacity={1} onPress={() => {
            console.log('这行的颜色改变之前是', data['color'])
            this.cellDidSelect(rowID)
            console.log('这行的颜色应该是', data['color'])
        }} />
        // 剩余代码和目标功能无关...
    )
}
```
然后贴一下点击 cell 的方法:
```js
cellDidSelect(rowID) {
    let arr = this.state.dataArray
    arr[rowID]['color'] = 'red'
    arr[rowID]['name'] = '改变了的名字'
    this.setState({
        dataSource: this.state.dataSource.cloneWithRows(arr),
        dataArray: arr,
    })
}
```

这样 我以为点击了 cell 就能改变 这个 cell 的背景颜色了...但是事实却不是这样, 无论我怎么点击这个 cell, 他的背景颜色还是那样

于是我在google上搜了以下, 和我这种情况最相近的解决办法有两个, 一个是在点击 setState 时, 数据源不能和以前的相同, 于是我这么做了
```js
cellDidSelect(rowID) {
    let arr = []
    arr = this.state.dataArray.slice() // 再拷贝一份数据
    arr[rowID]['color'] = 'red'
    arr[rowID]['name'] = '改变了的名字'
    this.setState({
        dataSource: ds.cloneWithRows(arr),
        dataArray: arr,
    })
}

// 结果一样不好用, 我又使用了网上提供的 另一种解决办法:

componentWillReceiveProps() {
    // 在这个方法中 setState dataSource
}
```

这个办法然我比较奇怪的是, 我在不 setState dataSource 的时候居然是好用的, 就是点击哪一行那一行就改变了背景颜色, 但是紧接着问题又来了: 如果我进行了上拉加载操作, 那么再次点击, cell 的背景颜色就不会发生改变了, 上拉加载操作本质就是加载了更多数据, 然后放到 ListView.DataSource 里面更新 ListView, 由于改变外观与数据无关, 那我可以猜测问题是不是出在这句话上`dataSource: this.state.dataSource.cloneWithRows(arr)`结果我看了一下他们的 ListView.DataSource 是这样写的
```js
var ds = new ListView.DataSource({
      rowHasChanged: (r1, r2) => r1 != r2
});
// 刷新数据
this.state = {
  ds:[{AwayTeam: "TeamA", HomeTeam: "TeamB", Selection: "AwayTeam"},{AwayTeam: "TeamC", HomeTeam: "TeamD", Selection: "HomeTeam"}],
  dataSource:ds,
}
```
我抱着试一试的想法, 把我的 ListView.DataSource 也写成了这样, 居然达到了目标, 无论怎么刷新,点击哪一行, 那一行的背景颜色就改变了

那为什么这两个看起来一样的方法使用起来会有差别呢? 

参考:[这篇博客](http://www.lynull.com/2016/04/03/%E6%B5%85%E6%9E%90listview%E7%BB%84%E4%BB%B602%E4%B9%8B%E5%88%B7%E6%96%B0%E6%95%B0%E6%8D%AE/)

他讲述了 `this.state.dataSource.cloneWithRows(data)` 与 自定义一个`var ds = new ListView.DataSource({rowHasChanged: (r1, r2) =>{return r1 !== r2 }});` 再 `ds.cloneWithRows(data)` 的区别, 观察区别的方法是这样的:
```js
// 重写一个数据源 打印一下什么时候 数据发生了改变
var ds = new ListView.DataSource({
    rowHasChanged: (r1, r2) =>{
        if(r1 !== r2){
            console.log('rowHasChanged!')
        }else {
            console.log('rowHasNotChanged')}
        return r1 !== r2
}});
```
具体做法就是: 用以上不同的两个 ListView.DataSource 去充当 ListView 的 dataSource 然后在主页面上写两个按钮分别对应两种 ListView.DataSource, 当按钮点击的时候, 就去 cloneWithRows:
```js
// 按钮1的点击事件
changeds(){
    console.log('ds data changing...');
    var data2 =[1,2,3] ;
    this.setState(
        {dataSource:ds.cloneWithRows(data2)}
    );
}

// log: 
// ds data changing...
// renderRow is 1
// renderRow is 2
// renderRow is 3

// 按钮2的点击事件
changeds2(){
    console.log('state ds data changing...');
    var data2 =[1,2,3] ;
    this.setState(
        {dataSource:this.state.dataSource.cloneWithRows(data2)}
    );
}

//log:
// ds data changing...
// renderRow is 1
// renderRow is 2
// renderRow is 3
// state ds data changing...
// rowHasNotChanged
// rowHasNotChanged
// rowHasNotChanged
```
重复刷新几下app, 会发现`ds.cloneWithRows(data)`这种方式会把数据直接放上去, 而`this.state.dataSource.cloneWithRows(data2)`会比较刷新前后两种数据是否相同, 相同的话不会再次刷新这行 cell 不同的话才会刷新

















