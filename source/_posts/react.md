---
title: 前端架构--React知识点
featured_image: ./images/loki.jpeg
---

#### setState 到底是异步还是同步

先给出答案:  有时表现出异步,有时表现出同步

setState 并不是单纯同步/异步的，它的表现会因调用场景的不同而不同。在源码中，通过 isBatchingUpdates 来判断setState 是先存进 state 队列还是直接更新，如果值为 true 则执行异步操作，为 false 则直接更新。

异步： 在 React 可以控制的地方，就为 true，比如在 React 生命周期事件和合成事件中，都会走合并操作，延迟更新的策略。
同步： 在 React 无法控制的地方，比如原生事件，具体就是在 addEventListener 、setTimeout、setInterval 等事件中，就只能同步更新。

https://juejin.cn/post/6941546135827775525#heading-30

setState 如何链式更新

```
  changeValue = v => {
    this.setState(state => {
      return {
        counter: state.counter + v
      }
    })
  }

  setCounter = () => {
    this.changeValue(1)
    this.changeValue(2)
    this.changeValue(3)
  }
```

---

#### react 生命周期

装载阶段（Mount），组件第一次在DOM树中被渲染的过程；
  constructor
  getDerivedStateFromProps
  render
  componentDidMount

更新过程（Update），组件状态发生变化，重新更新渲染的过程；
  getDerivedStateFromProps
  shouldComponentUpdate
  render
  getSnapshotBeforeUpdate
  componentDidUpdate

卸载过程（Unmount），组件从DOM树中被移除的过程；
  componentWillUnmount()

错误处理阶段

  componentDidCatch(error, info)；


新的生命周期（16.4 之后）:

Mounting:

Constructor()：

    组件的构造函数，第一个被执行，若没有显式定义它，会有一个默认的构造函数，但是若显式定义了构造函数，我们必须在构造函数中执行 super(props)，否则无法在构造函数中拿到this。
    如果不初始化 state 或不进行方法绑定，则不需要为 React 组件实现构造函数Constructor。
    constructor中通常只做两件事：

    初始化组件的 state
    给事件处理方法绑定 this

static getDerivedStateFromProps(props, state):

    这是个静态方法，所以不能在这个函数里使用 this，有两个参数 props 和 state，分别指接收到的新参数和当前组件的 state 对象，这个函数会返回一个对象用来更新当前的 state 对象，如果不需要更新可以返回 null。
    该函数会在装载时，接收到新的 props 或者调用了 setState 和 forceUpdate 时被调用。如当接收到新的属性想修改 state ，就可以使用。

render():

    render是React 中最核心的方法，一个组件中必须要有这个方法，它会根据状态 state 和属性 props 渲染组件。这个函数只做一件事，就是返回需要渲染的内容，所以不要在这个函数内做其他业务逻辑，通常调用该方法会返回以下类型中一个：

    React 元素：这里包括原生的 DOM 以及 React 组件；
    数组和 Fragment（片段）：可以返回多个元素；
    Portals（插槽）：可以将子元素渲染到不同的 DOM 子树种；
    字符串和数字：被渲染成 DOM 中的 text 节点；
    布尔值或 null：不渲染任何内容。

componentDidMount():

    会在组件挂载后（插入 DOM 树中）立即调。该阶段通常进行以下操作：

    执行依赖于DOM的操作；
    发送网络请求；（官方建议）
    添加订阅消息（会在componentWillUnmount取消订阅）；

    如果在 componentDidMount 中调用 setState ，就会触发一次额外的渲染，多调用了一次 render 函数，由于它是在浏览器刷新屏幕前执行的，所以用户对此是没有感知的，但是我应当避免这样使用，这样会带来一定的性能问题，尽量是在 constructor 中初始化 state 对象。

Updating:

static getDerivedStateFromProps(props, state):

    此时需要判断shouldUpdate，以便下一个生命周期使用

shouldComponentUpdate(nextProps, nextState):

    在说这个生命周期函数之前，来看两个问题：

    setState 函数在任何情况下都会导致组件重新渲染吗？例如下面这种情况：

    ```this.setState({number: this.state.number})```

    如果没有调用 setState，props 值也没有变化，是不是组件就不会重新渲染？

    第一个问题答案是 会 ，第二个问题如果是父组件重新渲染时，不管传入的 props 有没有变化，都会引起子组件的重新渲染。
    那么有没有什么方法解决在这两个场景下不让组件重新渲染进而提升性能呢？这个时候 shouldComponentUpdate 登场了，这个生命周期函数是用来提升速度的，它是在重新渲染组件开始前触发的，默认返回 true，可以比较 this.props 和 nextProps ，this.state 和 nextState 值是否变化，来确认返回 true 或者 false。当返回 false 时，组件的更新过程停止，后续的 render、componentDidUpdate 也不会被调用。
    
    注意： 添加 shouldComponentUpdate 方法时，不建议使用深度相等检查（如使用 JSON.stringify()），因为深比较效率很低，可能会比重新渲染组件效率还低。而且该方法维护比较困难，建议使用该方法会产生明显的性能提升时使用。

render()

getSnapshotBeforeUpdate(prevProps, prevState) :

    这个方法在 render 之后，componentDidUpdate 之前调用，有两个参数 prevProps 和 prevState，表示更新之前的 props 和 state，这个函数必须要和 componentDidUpdate 一起使用，并且要有一个返回值，默认是 null，这个返回值作为第三个参数传给 componentDidUpdate。

componentDidUpdate(prevProps，prevState，snapshot):

    componentDidUpdate() 会在更新后会被立即调用，首次渲染不会执行此方法。 该阶段通常进行以下操作：

    当组件更新后，对 DOM 进行操作；
    如果你对更新前后的 props 进行了比较，也可以选择在此处进行网络请求；（例如，当 props 未发生变化时，则不会执行网络请求）。

    该方法有三个参数：

    prevProps: 更新前的props
    prevState: 更新前的state
    snapshot: getSnapshotBeforeUpdate()生命周期的返回值

Umounting:

componentWillUnmount():

    卸载阶段只有一个生命周期函数，componentWillUnmount() 会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作：

    清除 timer，取消网络请求或清除
    取消在 componentDidMount() 中创建的订阅等；

    这个生命周期在一个组件被卸载和销毁之前被调用，因此你不应该再这个方法中使用 setState，因为组件一旦被卸载，就不会再装载，也就不会重新渲染。


Error Handing:

componentDidCatch(error, info)

    此生命周期在后代组件抛出错误后被调用。 它接收两个参数∶

    error：抛出的错误。
    info：带有 componentStack key 的对象，其中包含有关组件引发错误的栈信息

---

#### react为什么废弃那几个生命周期

废弃掉的生命周期 :( componentWillMount ， componentWillReceiveProps ， componentWillUpdate)

被废弃的三个函数都是在render之前，因为fiber的出现，很可能因为高优先级任务的出现而打断现有任务导致它们会被执行多次。
另外的一个原因则是，React想约束使用者，好的框架能够让人不得已写出容易维护和扩展的代码。

https://juejin.cn/post/6941546135827775525#heading-55

---

#### 组件通信

React组件间通信常见的几种情况:

    父组件向子组件通信
    子组件向父组件通信
    跨级组件通信
    非嵌套关系的组件通信

    1.父子间

    父组件向子组件通信：父组件通过 props 向子组件传递需要的信息。

    ```// 子组件: Child
      const Child = props =>{
        return <p>{props.name}</p>
      }
      // 父组件 Parent
      const Parent = ()=>{
          return <Child name="react"></Child>
      }
    ```

    子组件向父组件通信：: props+回调的方式。

    ```
    // 子组件: Child
    const Child = props =>{
      const cb = msg =>{
          return ()=>{
              props.callback(msg)
          }
      }
      return (
          <button onClick={cb("你好!")}>你好</button>
      )
    }
    // 父组件 Parent
    class Parent extends Component {
        callback(msg){
            console.log(msg)
        }
        render(){
            return <Child callback={this.callback.bind(this)}></Child>    
        }
    }
    ```

    2.跨级组件的通信方式:

    使用props，利用中间组件层层传递,但是如果父组件结构较深，那么中间每一层组件都要去传递props，增加了复杂度，并且这些props并不是中间组件自己需要的。
    使用context，context相当于一个大容器，可以把要通信的内容放在这个容器中，这样不管嵌套多深，都可以随意取用，对于跨越多层的全局数据可以使用context实现。

    ```
    // context方式实现跨级组件通信 
    // Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据
    const BatteryContext = createContext();
    //  子组件的子组件 
    class GrandChild extends Component {
        render(){
            return (
                <BatteryContext.Consumer>
                    {
                        color => <h1 style={{"color":color}}>我是红色的:{color}</h1>
                    }
                </BatteryContext.Consumer>
            )
        }
    }
    //  子组件
    const Child = () =>{
        return (
            <GrandChild/>
        )
    }
    // 父组件
    class Parent extends Component {
          state = {
              color:"red"
          }
          render(){
              const {color} = this.state
              return (
              <BatteryContext.Provider value={color}>
                  <Child></Child>
              </BatteryContext.Provider>
              )
          }
    }
    ```

    3. 非嵌套关系组件
   
    即没有任何包含关系的组件，包括兄弟组件以及不在同一个父级中的非兄弟组件。

    可以使用自定义事件通信（发布订阅模式）
    可以通过redux等进行全局状态管理
    如果是兄弟组件通信，可以找到这两个兄弟节点共同的父节点, 结合父子间通信方式进行通信。

    总结：组件通信的方式有哪些

    ⽗组件向⼦组件通讯: ⽗组件可以向⼦组件通过传 props 的⽅式，向⼦组件进⾏通讯
    ⼦组件向⽗组件通讯: props+回调的⽅式，⽗组件向⼦组件传递props进⾏通讯，此props为作⽤域为⽗组件⾃身的函 数，⼦组件调⽤该函数，将⼦组件想要传递的信息，作为参数，传递到⽗组件的作⽤域中
    兄弟组件通信: 找到这两个兄弟节点共同的⽗节点,结合上⾯两种⽅式由⽗节点转发信息进⾏通信
    跨层级通信: Context 设计⽬的是为了共享那些对于⼀个组件树⽽⾔是“全局”的数据，例如当前认证的⽤户、主题或⾸选语⾔，对于跨越多层的全局数据通过 Context 通信再适合不过
    发布订阅模式: 发布者发布事件，订阅者监听事件并做出反应,我们可以通过引⼊event模块进⾏通信
    全局状态管理⼯具: 借助Redux或者Mobx等全局状态管理⼯具进⾏通信,这种⼯具会维护⼀个全局状态中⼼Store,并根据不同的事件产⽣新的状态

---

#### vue 和 react 的 diff 算法比较

相同点：
Vue 和 react 的 diff 算法，都是不进行跨层级比较，只做同级比较。
都是深度优先算法。

不同点：

    1.Vue 进行 diff 时，调用 patch 打补丁函数，一边比较一边给真实的 DOM 打补丁
    2.Vue 对比节点，当节点元素类型相同，但是 className 不同时，认为是不同类型的元素，删除重新创建，而 react 则认为是同类型节点，进行修改操作
    3.
      ① Vue 的列表比对，采用从两端到中间的方式，旧集合和新集合两端各存在两个指针，两两进行比较，如果匹配上了就按照新集合去调整旧集合，每次对比结束后，指针向队列中间移动；
      ② 而 react 则是从左往右依次对比，利用元素的 index 和标识 lastIndex 进行比较，如果满足 index < lastIndex 就移动元素，删除和添加则各自按照规则调整；
      ③ 当一个集合把最后一个节点移动到最前面，react 会把前面的节点依次向后移动，而 Vue 只会把最后一个节点放在最前面，这样的操作来看，Vue 的 diff 性能是高于 react 的

https://github.com/pfan123/Articles/issues/62

---

#### React 中的 key 是什么，有什么作用

1.源码中 ReactChildFiber.js 文件中的 reconcileSingleElement 方法协调，比较老的 key 和最新的 key 是否相等，

再比较 type 是否相等，都相同则判定为同一元素，则复用原来的元素，否则重新创建（createFiberFromFragment）。

2.fiber 都为链表，如数组形式进行对比，upDateSlot 中同上，和 type 一起标识唯一性来更新节点。

3.fiber 拿子元素只能拿到第一个，如果进行两个数组对比，react 的 diff 算法中提供了 mapRemainingChildren 方法，

用 key 和 fiber 当作此方法的 key，value，把链表结构变成了 map 图，获取节点直接用 key 通过 get 就能得到，删除同理。

---

#### vue和react的key对比

https://www.zhoulujun.cn/html/webfront/ECMAScript/vue/8295.html

---

#### fiber

 fiber原理 https://segmentfault.com/a/1190000039189408

 hooks和fiber的关系 https://mp.weixin.qq.com/s/aXqPdPVxY4_D--dU3mILpQ

---

#### 合成事件

    1.事件委托：把所有事件注册到外层

    2.映射表： 根据映射表搜索当前事件，判断是否为合成事件，再处理。ReactDOMComponent.js中的registrationNameModules）

    生成映射表流程分析：

    ReactDOMComponent.js中引入的ReactDOMClientInjection即事件注入的地方，injectEventPluginOrder(DOMEventPluginOrder)
    规定插件顺序，setComponentTree传入函数。injectEventPluginsByName管理五种事件注入。把所有事件注册到一个大的对象中，这里的对象
    也就是前面五种事件的其中一种，根据不同分类进行注册。根据这个大的对象找到对应key，最后派发找到event，event接收到的参数包含target,
    也就是派发对象。

    injectEventPluginsByName执行会触发recomputePluginOrdering，最后执行publishEventForPlugin。在publishEventForPlugin中，
    执行了publishRegistrationName，最终在此事件中赋值给了registrationNameModules，形成映射表。

    事件的注册：

    legacyListenToEvent执行，层层调用，最终到达addTrappedEventListener，完成了事件的注册

https://juejin.cn/post/6844903988794671117


----

#### Redux 检查点

基本用法
https://juejin.cn/post/6844904200225161223

1.createStore 创建 store

2.reducer 初始化、修改状态函数

3.getState 获取状态值

4.dispatch 提交更新

5.subcribe 变更订阅


考点合集
https://juejin.cn/post/6940942549305524238#heading-1

---


#### 几种Hook的使用

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

https://blog.csdn.net/qq_42335214/article/details/111746800

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

---

#### hooks模拟生命周期

https://juejin.cn/post/6844903921442373639

---

#### react优化点

1.PureComponent/ShouldComponentUpdate
2.React.memo
3.React.useMemo
4.React.useCallback

https://github.com/brickspert/blog/issues/36

---

#### component和pureComponent

https://juejin.cn/post/6844903864341102599

---

#### refs 使用方式

1.createRef()挂到原生 dom 节点或者 class 组件

2.function 组件使用 ref 需要用 forwardRef()实现转发操作

3.ref 可以是 callback 形式（尽量不使用内联函数）

4.string 方式（过时，不建议使用）

---

#### react 核心 api

    webpack+babel编译时，替换JSX为 React.createElement(type,props,...children)，实际上，转换过程需要经过⼀一个diff过程，
    ⽐比对出实际更更 新补丁操作dom

    React.createElement: 创建虚拟DOM，执行结束后得到一个JS对象即 vdom，它能够完整描述dom结构

    React.Component: 实现自定义组件

    ReactDOM.render: 渲染真实DOM

      ReactDOM.render(element, container[, callback])：当首次调用时，容器节点里的所有 DOM 元素都会被替换， 后续的
      调用则会使用 React 的 DOM 差分算法(DOM diffing algorithm)进⾏⾼效的更新。如果提供了可选的回调函数，该回调将
      在组件被渲染或更新之后被执行。

      ReactDOM.render(vdom,container)可以将vdom转换为 dom并追加到container中

---

#### react实现双向绑定

一、通过事件对象获取值

  event.target.value可以获取到input框的值。

  具体实现步骤可拆分为：
  1.给input定义value属性，值为state的msg，即value={this.state.msg}，是为了保证初始页面时，两值相等。
  2.给input定义onChange属性，值为当前React组件里的方法inputChange，即onChange={this.inputChange}。
  3.inputChange方法中通过event.target.value方法获取到当前input的值，并赋值给state的msg。

二、通过ref来获取

  ref可以获取到当前元素的DOM节点。

  具体实现步骤可拆分为：
  1.给input定义value属性，值为state的msg，即value={this.state.msg}，是为了保证初始页面时，两值相等。
  2.给input定义ref属性，即ref="msgInput"。
  3.inputChange方法中通过this.refs.msgInput获取到当前input的DOM节点，并赋值给state的msg。

---

#### react实现插槽

https://juejin.cn/post/6877115720820326407

---

#### 高阶组件

https://zhuanlan.zhihu.com/p/24776678


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

#### react-router 的实现原理

客户端路由实现的思想：

基于 hash 的路由：通过监听hashchange事件，感知 hash 的变化

改变 hash 可以直接通过 location.hash=xxx


基于 H5 history 路由：

改变 url 可以通过 history.pushState 和 resplaceState 等，会将URL压入堆栈，同时能够应用 history.go() 等 API
监听 url 的变化可以通过自定义事件触发实现

react-router 实现的思想：

基于 history 库来实现上述不同的客户端路由实现思想，并且能够保存历史记录等，磨平浏览器差异，上层无感知
通过维护的列表，在每次 URL 发生变化的回收，通过配置的 路由路径，匹配到对应的 Component，并且 render

---

#### react-router的运用

https://juejin.cn/post/6941546135827775525#heading-70

跳转的几种方式

    1. params形式，路由跳转后，参数会显示在地址栏

    跳转的方法是使用history.push({pathname: '/personal', search: 'test=22222'})，其中search键对应的值就是拼接在地址栏的数据

    接收的方法。数据都是存储在useLocation中的search获取

    2.使用state的形式，页面刷新不会丢失数据，并且地址栏也看不到数据

    跳转的方法是使用history.push({pathname: '/personal', state: {test: 'dashboard'}})，其中search键对应的值就是拼接在地址栏的数据

    接收的方法。数据都是存储在useLocation中的search获取

---

#### react-router的两种模式

React-Router 支持使用 hash（对应 HashRouter）和 browser（对应 BrowserRouter） 两种路由规则， react-router-dom 提供了 BrowserRouter 和 HashRouter 两个组件来实现应用的 UI 和 URL 同步：

BrowserRouter 创建的 URL 格式：xxx.com/path
HashRouter 创建的 URL 格式：xxx.com/#/path

（1）BrowserRouter
它使用 HTML5 提供的 history API（pushState、replaceState 和 popstate 事件）来保持 UI 和 URL 的同步。由此可以看出，BrowserRouter 是使用 HTML 5 的 history API 来控制路由跳转的：

```
<BrowserRouter
    basename={string}
    forceRefresh={bool}
    getUserConfirmation={func}
    keyLength={number}
/>
```

其中的属性如下：

basename 所有路由的基准 URL。basename 的正确格式是前面有一个前导斜杠，但不能有尾部斜杠；

```
<BrowserRouter basename="/calendar">
    <Link to="/today" />
</BrowserRouter>
```

等同于

```<a href="/calendar/today" />```

forceRefresh 如果为 true，在导航的过程中整个页面将会刷新。一般情况下，只有在不支持 HTML5 history API 的浏览器中使用此功能；
getUserConfirmation 用于确认导航的函数，默认使用 window.confirm。例如，当从 /a 导航至 /b 时，会使用默认的 confirm 函数弹出一个提示，用户点击确定后才进行导航，否则不做任何处理；
KeyLength 用来设置 Location.Key 的长度。

（2）HashRouter
使用 URL 的 hash 部分（即 window.location.hash）来保持 UI 和 URL 的同步。由此可以看出，HashRouter 是通过 URL 的 hash 属性来控制路由跳转的：
```
<HashRouter
    basename={string}
    getUserConfirmation={func}
    hashType={string}  
/>
```
其参数如下：

basename, getUserConfirmation 和 BrowserRouter 功能一样；
hashType window.location.hash 使用的 hash 类型，有如下几种：

slash - 后面跟一个斜杠，例如 #/ 和 #/sunshine/lollipops；
noslash - 后面没有斜杠，例如 # 和 #sunshine/lollipops；
hashbang - Google 风格的 ajax crawlable，例如 #!/ 和 #!/sunshine/lollipops。
