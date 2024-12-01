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

#### 父组件调用子组件的方法

https://blog.csdn.net/qq_36990322/article/details/109858890

---

#### react为什么使用虚拟dom

https://zhuanlan.zhihu.com/p/411785091

---

#### 虚拟dom的原理

虚拟 DOM 的实现原理主要包括以下 3 部分：

用 JavaScript 对象模拟真实 DOM 树，对真实 DOM 进行抽象；
diff 算法 — 比较两棵虚拟 DOM 树的差异；
pach 算法 — 将两个虚拟 DOM 对象的差异应用到真正的 DOM 树。

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

#### hooks的原理

https://mp.weixin.qq.com/s/J0_PLrbVZMRAiwjWK2WDrw

https://zhuanlan.zhihu.com/p/443264124

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

#### redux的reducer为什么要设计成不能直接修改原状态，而返回修新状态对象

第一，从源码角度，因为redux会对reducer返回的状态进行引用地址的比较，不同才更新，所以直接修改旧状态不会更新。

第二，从设计角度，如果要知道reducer返回的状态是否有变化，必须进行状态对象的深度比较，这样比较消耗性能，所以仅进行状态对象引用地址的比较，由开发者来决定是否更新，返回新状态才更新。

https://segmentfault.com/q/1010000037662437

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

---

#### 特殊情况解决问题

##### 在 React 中，父组件传递给子组件的回调函数中涉及异步操作（例如 fetch 请求）。子组件调用该回调后，如何确保在回调异步请求期间组件未被卸载，从而避免内存泄漏？

在 React 中，如果组件在异步请求未完成前被卸载，会导致内存泄漏警告。为了解决此问题，可以在 useEffect 钩子中返回清理函数，或使用一个标志变量来取消异步操作

```jsx
const MyComponent = ({ fetchData }) => {
  useEffect(() => {
    let isMounted = true;
    fetchData().then((data) => {
      if (isMounted) {
        // 更新状态
      }
    });
    return () => {
      isMounted = false;
    };
  }, [fetchData]);
};
```

---

##### React 的 useEffect 钩子中如何优雅地处理依赖项更新，避免由于闭包问题导致引用的变量值不正确？请举例说明如何处理这种情况。

React 的 useEffect 使用了闭包，可能导致其访问的变量值为旧值。可以通过将依赖项正确地传入 useEffect，或者使用 useRef 来存储最新值来解决。

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  const interval = setInterval(() => {
    setCount((prevCount) => prevCount + 1); // 使用回调函数来确保获取最新的 `count`
  }, 1000);
  return () => clearInterval(interval);
}, []);

```

---

##### 在一个 React 项目中，多个组件共享同一个状态并需要及时同步更新。如何在不使用全局状态管理库（如 Redux 或 Context）的情况下实现这一功能？

可以使用 useState 和 useContext 结合来共享状态

```jsx
const SharedStateContext = React.createContext();

const SharedStateProvider = ({ children }) => {
  const [sharedState, setSharedState] = useState(null);
  return (
    <SharedStateContext.Provider value={{ sharedState, setSharedState }}>
      {children}
    </SharedStateContext.Provider>
  );
};

// 子组件使用
const ChildComponent = () => {
  const { sharedState, setSharedState } = useContext(SharedStateContext);
  return <div>{sharedState}</div>;
};

```

---

##### 如何在 React 中实现一个动态表单，其中的字段数量和类型由服务器返回的数据决定？请描述实现步骤并说明如何确保每个字段都能正确更新状态。

可以使用 useState 来存储动态字段，并渲染它们

```jsx

const DynamicForm = ({ fields }) => {
  const [formState, setFormState] = useState(() =>
    fields.reduce((acc, field) => ({ ...acc, [field.name]: '' }), {})
  );

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormState((prev) => ({ ...prev, [name]: value }));
  };

  return (
    <form>
      {fields.map((field) => (
        <input
          key={field.name}
          name={field.name}
          value={formState[field.name]}
          onChange={handleChange}
        />
      ))}
    </form>
  );
};

```

---

##### 你在开发一个有复杂交互的表单页面，页面中有一个 useEffect 钩子监听了多个状态变量。当其中一个状态变化时，如何防止 useEffect 被不必要的触发？

可以使用 useRef 来保存不需要触发的状态值

```jsx
const [state, setState] = useState(0);
const prevStateRef = useRef();

useEffect(() => {
  if (prevStateRef.current !== state) {
    // 仅当 state 变化时执行
  }
  prevStateRef.current = state;
}, [state]);

```

---

##### 在 React 项目中，当组件使用 useRef 持有 DOM 引用时，如何确保在该 DOM 元素动态更新（例如高度改变）时 useEffect 钩子能检测到变化？

使用 ResizeObserver 结合 useRef

```jsx
const ref = useRef();

useEffect(() => {
  const observer = new ResizeObserver((entries) => {
    for (let entry of entries) {
      console.log('New size:', entry.contentRect.height);
    }
  });
  if (ref.current) {
    observer.observe(ref.current);
  }
  return () => observer.disconnect();
}, []);

```

---

##### 你需要实现一个组件，要求每次渲染时生成不同的 ID 并将其存储，但该 ID 不会在重新渲染时丢失。如何使用 React 钩子实现该功能？

使用 useMemo 或 useRef

```jsx
const uniqueId = useMemo(() => `id-${Math.random().toString(36).substr(2, 9)}`, []);
// 或者
const uniqueId = useRef(`id-${Math.random().toString(36).substr(2, 9)}`).current;

```

---

##### React 中如何防止 shouldComponentUpdate 或 React.memo 因属性对象浅比较而导致不必要的重渲染？请解释如何解决对象或数组作为 props 的优化问题。

将传递的对象或数组使用 useMemo 缓存

```jsx
const memoizedProps = useMemo(() => ({ key: value }), [value]);
return <MyComponent prop={memoizedProps} />;

```

---

##### 在 React 中，如果多个组件都需要订阅同一个事件（如 window 的 resize 事件），但又不希望每个组件都创建一个独立的事件监听器，如何优化实现？

将事件监听器提升到父组件，并通过 Context 或 props 将其传递

```jsx
useEffect(() => {
  const handleResize = () => {
    console.log('Resized');
  };
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);

```

---

##### 如何在 React 的 useEffect 中使用 async/await？为什么 useEffect 不能直接接受 async 函数，如何解决这一问题？

useEffect 中不能直接使用 async 函数。应在钩子内定义异步函数并调用

这是因为 React 的 useEffect 回调函数需要返回一个清理函数（或什么都不返回），而 async 函数返回的是一个 Promise，与 React 对 useEffect 的设计要求不匹配。

在 React 的设计中，useEffect 回调函数的返回值被用作清理函数。如果返回的是一个函数，React 会在组件卸载或依赖项更新时调用它以清理资源。
如果直接传递一个 async 函数作为 useEffect 的回调，async 函数实际上返回的是一个 Promise，而不是清理函数。

如果 useEffect 支持直接返回 Promise，React 可能无法在异步操作完成之前进行清理。这会导致清理操作的顺序混乱，影响依赖项的更新和资源管理。

```jsx
useEffect(() => {
  const fetchData = async () => {
    const data = await fetch('/api/data');
    // 处理 data
  };
  fetchData();
}, []);

```

---

##### 一个 useState 更新的操作需要基于其当前值进行计算，但异步操作导致状态不同步，如何确保 setState 时状态始终是最新的？

使用函数更新 setState

```jsx
setState((prevState) => {
  return prevState + 1;
});

```

---

#### 为什么 Hooks 只能在函数组件或自定义 Hook 的最外层调用

React 的设计目标是保证组件行为的一致性，防止因为调用顺序变化引起的状态管理错误。Hooks 的这一规则（称为 Hook 的“规则”或“法则”）确保了 React 内部能正确追踪每个 Hook 的状态。

1. Hook 调用顺序的保证

React 通过记录 Hook 的调用顺序来管理它们的状态。
如果 Hook 被放在条件语句或循环中，调用顺序可能会在每次渲染时发生变化，从而导致状态的错乱。例如：

```js
// 错误示例
if (condition) {
  useState(0); // 可能这次渲染被调用，下次渲染未调用
}
```

上述代码可能在某些渲染中跳过 useState 的调用，导致 React 无法正确对应 Hook 与状态，最终抛出错误。

2. React 如何管理 Hooks 状态

React 内部为每个组件维护了一个Hook 调用链表。在每次渲染时，React 根据调用顺序依次分配每个 Hook 的状态。如果 Hook 调用顺序混乱，链表的结构会被破坏，导致状态更新错误。

3. 一致的渲染逻辑

为了保持组件行为的一致性，React 规定 Hooks 必须始终按照固定顺序调用，确保：

每次渲染访问的 Hook 对应的状态相同。
不会因为某些条件分支导致状态“丢失”或“错位”。

4. 解决复杂逻辑的方式

当需要在条件语句或循环中使用 Hooks 时，可以将这些逻辑抽离到自定义 Hook 中：

```js
function useConditionalHook(condition) {
  const state = useState(0);
  if (condition) {
    // 在这里可以使用任何 Hooks，但要遵守调用规则
  }
  return state;
}

// 在组件中
function MyComponent() {
  const [value] = useConditionalHook(true);
  return <div>{value}</div>;
}

```

5. 违反规则会发生什么？

如果尝试在条件分支或循环中调用 Hook，React 会报错：

```sql
React Hook "useXXX" is called conditionally. React Hooks must be called in the exact same order in every component render.
```

---

#### 使用useState，set后有时候会拿不到最新的值是什么原因？怎么解决

##### 原因分析
状态更新是异步的
React 会将多次状态更新操作批量处理，以提高性能。setState 后，状态并不会立刻更新，而是等到组件重新渲染时，React 才会更新到最新的状态值。

闭包陷阱
在函数组件中，某些回调函数会捕获函数执行时的状态快照。如果状态更新后使用的是旧的状态引用，就会导致拿到旧值的问题

##### 解决方法

1. 使用 useEffect 确保状态更新完成后再操作
如果需要在状态更新后做一些事情，可以借助 useEffect：

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  console.log("count 更新了:", count);
}, [count]); // 当 count 变化时触发

```

2. 使用函数式更新
函数式更新可以避免闭包捕获旧值的问题。setState 接收一个函数作为参数，函数的参数是最新的状态值：

```jsx
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount((prevCount) => prevCount + 1); // 使用最新的 prevCount
  console.log("最新 count:", count); // 这里仍然是旧值，但状态已正确更新
};

```

3. 在事件处理器中使用最新状态值
如果状态更新后需要立刻使用最新的状态值，可以显式传递或维护一个变量。例如：

```jsx
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount((prevCount) => {
    const newCount = prevCount + 1;
    console.log("最新 count:", newCount);
    return newCount;
  });
};

```

4. 使用 useRef 存储最新状态值
如果需要在整个组件生命周期中获取最新状态，可以使用 useRef 存储：

```jsx
const [count, setCount] = useState(0);
const countRef = useRef(count);

useEffect(() => {
  countRef.current = count; // 同步最新状态到 ref
}, [count]);

const handleClick = () => {
  setCount(count + 1);
  console.log("最新 count:", countRef.current); // 始终获取最新值
};

```

---

#### useReducer

useReducer 是 React 提供的一个 Hook，用于管理更复杂的状态逻辑。它是 useState 的替代方案，适合当状态变化逻辑包含多个子值或者需要基于特定操作类型来更新状态的情况。

```jsx
const [state, dispatch] = useReducer(reducer, initialState, init);

```

reducer：一个纯函数，接收当前状态 (state) 和动作 (action)，返回新的状态。
initialState：状态的初始值。
init（可选）：用于惰性初始化的函数。
state：当前状态值。
dispatch：派发动作 (action) 的方法。

##### 为什么使用 useReducer？

复杂状态管理：当状态包含多个子值或依赖复杂的逻辑时，useReducer 比 useState 更清晰。
分离逻辑：通过 reducer 将状态管理逻辑抽离，方便维护和测试。
适合团队协作：useReducer 的模式类似于 Redux，团队成员可能更熟悉。

```jsx
import React, { useReducer } from "react";

// 定义 reducer 函数
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return { count: 0 };
    default:
      throw new Error("Unknown action");
  }
}

function Counter() {
  const initialState = { count: 0 }; // 初始状态
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "reset" })}>Reset</button>
    </div>
  );
}

export default Counter;

```

---

#### 用hooks实现一个redux功能

使用 useReducer 管理全局状态。
结合 Context 提供状态和 dispatch 方法。
通过 useContext 在子组件中访问状态和分发动作的方法。

步骤 1: 创建全局状态管理器
使用 createContext 和 useReducer 创建一个类似 Redux 的状态管理功能。

```jsx
import React, { createContext, useReducer, useContext } from "react";

// 定义初始状态
const initialState = {
  count: 0,
  user: null,
};

// 定义 reducer 函数
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + 1 };
    case "decrement":
      return { ...state, count: state.count - 1 };
    case "setUser":
      return { ...state, user: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
}

// 创建 Context
const GlobalStateContext = createContext();
const GlobalDispatchContext = createContext();

// 创建全局状态管理 Provider
export function GlobalProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <GlobalStateContext.Provider value={state}>
      <GlobalDispatchContext.Provider value={dispatch}>
        {children}
      </GlobalDispatchContext.Provider>
    </GlobalStateContext.Provider>
  );
}

// 自定义 Hook: 获取全局状态
export function useGlobalState() {
  const context = useContext(GlobalStateContext);
  if (context === undefined) {
    throw new Error("useGlobalState must be used within a GlobalProvider");
  }
  return context;
}

// 自定义 Hook: 获取全局 dispatch
export function useGlobalDispatch() {
  const context = useContext(GlobalDispatchContext);
  if (context === undefined) {
    throw new Error("useGlobalDispatch must be used within a GlobalProvider");
  }
  return context;
}

```

步骤 2: 在顶层组件中提供全局状态
使用 GlobalProvider 包裹应用的组件树，提供全局状态和 dispatch。

```jsx
import React from "react";
import { GlobalProvider } from "./GlobalState";
import Counter from "./Counter";
import User from "./User";

function App() {
  return (
    <GlobalProvider>
      <Counter />
      <User />
    </GlobalProvider>
  );
}

export default App;

```

步骤 3: 在子组件中消费全局状态和 dispatch

```jsx
import React from "react";
import { useGlobalState, useGlobalDispatch } from "./GlobalState";

function Counter() {
  const state = useGlobalState();
  const dispatch = useGlobalDispatch();

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </div>
  );
}

export default Counter;

```

---

#### Context和useContext区别

Context 是 React 提供的一个全局状态管理工具，用于跨组件共享数据而不需要通过逐层传递 props。

用法
Context 是通过 React.createContext 创建的一个对象，包括两个核心部分：

Provider：用于提供数据。
Consumer：用于消费数据（在较新的代码中，useContext 更常用）。

```jsx
import React, { createContext } from "react";

const MyContext = createContext(); // 创建 Context

function App() {
  return (
    <MyContext.Provider value={{ user: "Alice" }}> {/* 提供数据 */}
      <Child />
    </MyContext.Provider>
  );
}

function Child() {
  return (
    <MyContext.Consumer> {/* 消费数据 */}
      {({ user }) => <p>User: {user}</p>}
    </MyContext.Consumer>
  );
}

```

Context.Provider 包裹的组件树都可以访问到传递的值。
Consumer 需要用回调函数来获取上下文数据（旧式方式）。

useContext 是 React 的一个 Hook，用于函数组件中直接读取 Context 的值，简化了数据获取的过程，取代了旧的 Context.Consumer。

用法
在函数组件中使用 useContext 获取上下文的值。

```jsx
import React, { createContext, useContext } from "react";

const MyContext = createContext(); // 创建 Context

function App() {
  return (
    <MyContext.Provider value={{ user: "Alice" }}> {/* 提供数据 */}
      <Child />
    </MyContext.Provider>
  );
}

function Child() {
  const contextValue = useContext(MyContext); // 使用 useContext 获取值
  return <p>User: {contextValue.user}</p>;
}

```

简化了 Context 数据的使用，去掉了嵌套回调函数的复杂性。
只能在函数组件或自定义 Hook 中使用。

---

