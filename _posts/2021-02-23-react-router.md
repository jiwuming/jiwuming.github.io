---
title: react-router-dom
tags: [前端]
date: 2021-02-23
---
# 基本配置
```xml
<HashRouter>
    <!-- 如果不使用 Switch 则多层的路由都会渲染到页面上， exact 精准匹配 -->
    <Switch>
        <Route exact={true} path={'/home'} component={HomePage}/>
        <Route exact={true} path={'/second'} component={SecondPage}/>
        <Route exact={true} path={'/third'} component={ThirdPage}/>
        <Route exact={true} path={'/second/third'} component={ThirdPage}/>
    </Switch>
</HashRouter>
```

# 传参 & 获取参数
```js
<!-- 定义一个id参数 -->
<Route exact={true} path={'/home/:id'} component={HomePage}/>
<!-- 跳转 -->
const history = useHistory()
history.push({
    pathname: `/second/123`, // 路由 + 显示参数，也可以把query拼上去 比如 ?param=xxx&param2=xxx 然后用 useRouteMatch 获取
    state: {
        name: 'wo' // 隐式传参
    }
})
<!-- 返回上一级 -->
history.goBack()
<!-- 获取参数 -->
const routeParams = useParams()
<!-- 获取路由 -->
const history = useHistory()
<!-- 获取location 可以用这个获取state传参 location.state -->
const location = useLocation()
<!-- 比较详细的路由对象 -->
const match = useRouteMatch()
```

# 重定向
```xml
<!-- 可以使用Redirect重定向到某个路由，也可以直接把路由组件写上去 -->
<Route exact={true} path={'/'} render={props => {
    return props.history.length > 0 ? (<Redirect to={'/home'} />) : (<Redirect to={'/second'}/>)
}} />
```

# 404
```xml
<HashRouter>
    <Switch>
        <Route exact={true} path={'/home'} component={HomePage}/>
        <Route exact={true} path={'/second'} component={SecondPage}/>
        <Route exact={true} path={'/third'} component={ThirdPage}/>
        <Route exact={true} path={'/second/third'} component={ThirdPage}/>
        <!-- 不用写路由 放到页面最下面 -->
        <Route component={NotFoundPage}/>
    </Switch>
</HashRouter>
```

# 动态路由
```xml
<!-- 目前react 官方不支持对象配置方式的路由了 onEnter onLeave 钩子也被取消了，需要自己做一个组件实现，返回<Route /> 或 <Redirct />。 -->
<HashRouter>
    <Switch>
        <组件实现 />
    </Switch>
</HashRouter>
```

