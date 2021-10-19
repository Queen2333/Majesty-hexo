---
title: 前端架构--React高级用法
featured_image: ./images/ironman3.jpg
---

#### Redux 检查点

1.createStore 创建 store

2.reducer 初始化、修改状态函数

3.getState 获取状态值

4.dispatch 提交更新

5.subcribe 变更订阅

---

#### react-router 的三种渲染方式

Route 渲染的优先级 children > component > render

三种方式互斥，只能用一种

children: func
无论 location 是否匹配都会渲染

render: func
location 匹配时才会渲染

component: component
location 匹配时才会渲染

404 页面: 设定一个没有 path 的路由在路由列表最后，表示一定匹配，根路由要添加 exact，实现精确匹配

```
  <Switch>
    <Route>...</Route> // 常规路由
    <Route component={EmptyPage}></Route> // 404路由
  </Switch>
```
