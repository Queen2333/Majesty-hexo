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

---

#### Hook

hook 的基本操作

```
  import React, { uaeState, useEffect } from "react";

  export default function HookPage(props) {
    // 定义一个叫count的state变量，初始化为0
    const {count, setCount} = useState(0)

    return (
      <div>
        <h3>HookPage</h3>
        <p>{count}</p>
        <button onClick={()=>setCount(count + 1)}>add</button>
      </div>
    )
  }
```

Effect Hook

可在函数组件中执行副作用操作

1.数据获取（ajax）

2.设置订阅以及手动更改 React 组件中的 Dom

```
  import React, { uaeState, useEffect } from "react";

  export default function HookPage(props) {
    // 定义一个叫count的state变量，初始化为0
    const [count, setCount] = useState(0)
    const [date, setDate] = useState(new Date())
    //和didMount、didUpdate类似

    //条件执行
    useEffect(() => {
      // 只需要在count发生改变的时候执行
      document.title = `点击了${count}次`
    }, [count])

    useEffect(() => {
      // 只需要在didMount的时候执行
      const timer = setInterval(() => {
        setData(new Date())
      }, 1000)

      // 清除定时器，类似willUnmount
      return () => clearInterval(timer)

    }, [])
    return (
      <div>
        <h3>HookPage</h3>
        <p>{count}</p>
        <button onClick={()=>setCount(count + 1)}>add</button>
        <p>{date.toLocaleTimeString()}</p>
      </div>
    )
  }
```

---
