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

#### 自定义 Hook 和 使用规则

实现组件的状态逻辑共用

```
// 自定义一个hook命名要以use开头,
  return (
    <div>
      <h3>HookPage</h3>
      <p>{count}</p>
      <button onClick={()=>setCount(count + 1)}>add</button>
      <p>{useClock().toLocaleTimeString()}</p>
    </div>
  )

  function useClock() {
    const [date, setDate] = useState(new Date())
    useEffect(() => {
      // 只需要在didMount的时候执行
      const timer = setInterval(() => {
        setData(new Date())
      }, 1000)

      // 清除定时器，类似willUnmount
      return () => clearInterval(timer)

    }, [])
    return date
  }
```

使用 Hook 的额外规则：

    1.只能在函数最外层调用hook，不要在循环、条件判断或者子函数中调用

    2.只能在React的函数组件中调用Hook。不要在其他js函数中调用（在自定义Hook中可以调用）

---

#### Hook 的 api

##### useMemo

把创建函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖改变时才重新计算 memoized 的值。

```
import React, { uaeState, useEffect, useMemo } from "react";

export default function useMemoPage(props) {
  const [count, setCount] = useState(0)

  const expensive = useMemo(() => {
    let sum = 0
    for(let i = 0; i < count; i++) {
      sum += i
    }
    return sum
    // 只有count改变时，当前函数才会重新执行
  }, [count])
  const [value, setValue] = useState('')

  return (
    <div>
      <h3>useMemoPage</h3>
      <p>{count}</p>
      <p>expensive: {expensive()}</p>
      <button onClick={()=>setCount(count + 1)}>add</button>
      <input value={value} onChange={event=>setValue(event.target.value)}/>
    </div>
  )
}
```

##### useCallback

把内联回调函数以及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某依赖项改变时才会更新。
当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（如 shouldComponentUpdate）的子组件时非常有用。

```
import React, { uaeState, useEffect, PureComponent } from "react";

export default function useCallbackPage(props) {
  const [count, setCount] = useState(0)

  const addClick = useCallback(() => {
    let sum = 0
    for(let i = 0; i < count; i++) {
      sum += i
    }
    return sum
  }, [count])
  const [value, setValue] = useState('')

  return (
    <div>
      <h3>useCallbackPage</h3>
      <p>{count}</p>
      <p>expensive: {expensive()}</p>
      <button onClick={()=>setCount(count + 1)}>add</button>
      <input value={value} onChange={event=>setValue(event.target.value)}/>
      <Child addClick={addClick}/>
    </div>
  )
}

class Child extends PureComponent {
  render() {
    const {addClick} = this.props
    return (
      <div>
        <h3>Child</h3>
        <button onClick={()=>console.log(addClick())}>add</button>
      </div>
    )
  }
}

```
