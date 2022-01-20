---
title: 前端架构--React源码分析
featured_image: ./images/ironman1.jpg
---

#### react 和 vue 的区别

react 和 vue 都是做组件化的，整体的功能都类似，但是他们的设计思路是有很多不同的。

##### 1>数据是不是可变的

react 整体是函数式的思想，把组件设计成纯组件，状态和逻辑通过参数传入，所以在 react 中，是单向数据流，在 setState 之后会重新走渲染的流程。而 vue 的思想是响应式的，也就 是基于是数据可变的，通过对每一个属性建立 Watcher 来监听，当属性变化的时候，响应式 的更新对应的虚拟 dom。 react 的性能优化需要手动去做，而 vue 的性能优化是自动的，但是 vue 的响应式机制也 有问题，就是当 state 特别多的时候，Watcher 也会很多，会导致卡顿，所以大型应用(状 态特别多的)一般用 react，更加可控。

##### 2>通过 js 来操作一切，还是用各自的处理方式

react 的思路是 all in js，通过 js 来生成 html，所以设计了 jsx，还有通过 js 来操作 css。 vue 是把 html，css，js 组合到一起，用各自的处理方式，vue 有单文件组件，可以把 html、 css、js 写到一个文件中，html 提供了模板引擎来处理。

##### 3>类式的组件写法，还是声明式的写法(vue3.0 后无此区别了)

react 是类式的写法，api 很少，而 vue 是声明式的写法，通过传入各种 options，api 和参 数都很多。所以 react 结合 typescript 更容易一起写，vue 稍微复杂。 vue 结合 vue-class-component 也可以实现类式的写法，但是还是需要通过 decorator 来 添加声明，并不纯粹。 react 可以通过高阶组件(Higher Order Components--HOC)来扩展，而 vue 需要通过 mixins 来扩展。

##### 4>什么功能内置，什么交给社区去做

react 做的事情很少，很多都交给社区去做，vue 很多东西都是内置的。 比如 redux 的 combineReducer 就对应 vuex 的 modules。vuex 的 mutation 是直接改变的 原始数据，而 redux 的 reducer 是返回一个全新的 state，所以 redux 结合 immutable 来优 化性能，vue 不需要。

##### 总结:

react 整体的思路就是函数式，所以推崇纯组件，数据不可变，单向数据流，当然需要双向 的地方也可以做到，比如结合 redux-form，而 vue 是基于可变数据的，支持双向绑定。react 组件的扩展一般是通过高阶组件，而 vue 组件会使用 mixin。vue 内置了很多功能，而 react 做的很少，很多都是由社区来完成的，vue 追求的是开发的简单，而 react 更在乎方式是否 正确。

---

#### setState 到底是异步还是同步

先给出答案:  有时表现出异步,有时表现出同步

setState 只在合成事件和生命周期钩子函数中是“异步”的，在原生事件和 setTimeout 中都是同步的。

setState 的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形成了所谓的“异步”。当然可以通过第二个参数 setState(partialState, callback) 中的 callback 拿到更新后的结果，也可以利用 setTimeout 达到同步效果。

setState 的批量更新优化也是建立在“异步”(合成事件、钩子函数)之上的，在原生事件和 setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次 setState，setState 的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时 setState 多个不同的值，在更新时会对其进行合并批量更新，所以 state 值可能会被合并。

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

废弃掉的生命周期 :( componentWillMount ， componentWillReceiveProps ， componentWillUpdate)

新的生命周期（16.4 之后）:

Mounting:

Constructor()：

    源码中ReactFiberBeginWork.js中，updateClassComponent()函数，最终返回一个nextUnitOfWork，也就是获取下一个执行的单元，在此之前先执行当前的单元。
    updateClassComponent()接受的参数中，Component就是当前类。函数中，拿到当前instance后（在workInProgress.stateNode）,stateNode第一次渲染时也就
    是挂载阶段为null，即进入构造阶段constructClassInstance（在ReactFiberClassComponent.js）。此方法接受三个参数，函数中获取当前instance，也就是
    new ctor(props, context)。

static getDerivedStateFromProps(props, state):

    在 render 前调用，并且在初始挂载及后续更新时都会被调用。static属性，函数是无法被函数实例化的instance获取到。通过ctor获得getDerivedStateFromProps，
    判断是否为function。首先获取原先的state，即prevState，存在memorizedState中。partialState和prevState进行合并。他应该返回一个对象来更新 state，
    如果返回 null则不更新任何内容。

render():

    初始值shouldUpdate为true，根据finishClassComponent获取nextUnitOfWork，接着instance实例化，然后更新class（获取children即。instance.render()）,
    最终返回child元素。

componentDidMount():

    提交阶段（ReactFiberWorkLoop.js）commitLifeCycles
    组件装载之后调用，此时我们可以获取到 DOM 节点并操作，比如 对 canvas，svg 的操作，服务器请求，订阅都可以写在这个里面，但是记得在 componentWillUnmount 中取消订阅

Updating:

static getDerivedStateFromProps(props, state):

    此时需要判断shouldUpdate，以便下一个生命周期使用

shouldComponentUpdate(nextProps, nextState):

    有两个参数 nextProps 和 nextState，表示 新的属性和变化之后的 state，返回一个布尔值，true 表示会触发重新渲染，false 表示不会 触发重新渲染，默认返回 true,我们通常
    利用此生命周期来优化 React 程序性能。源码中根据上面的shouldUpdate来确定返回true或false

render()

getSnapshotBeforeUpdate(prevProps, prevState) :

    这个方法在 render 之后 ， componentDidUpdate 之前调用。在最近一次渲染输出（提交到 DOM 节点）之前调用，它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如滚动位置）。
    有两个参数 prevProps 和 prevState，表示之前的属性和之前的 state，这个函数有一个返回值，会作为第三个参数传给 componentDidUpdate，如果你不想要返回值，可以返回 null，
    此生命周期必须与 componentDidUpdate 搭配使用。

componentDidUpdate(prevProps，prevState，snapshot):

    该方法在 getSnapshotBeforeUpdate 方法之后被调用，有三个参数 prevProps，prevState，snapshot，表示之前的props，之前的 state，和 snapshot。第三个 参数是 getSnapshotBeforeUpdate 返回的,如果触发某些回调函数时需要用到 DOM 元素的 状态，则将对比或计算的过程迁移至 getSnapshotBeforeUpdate，然后在 componentDidUpdate
    中统一触发回调或更新状态。此阶段源码同componentDidMount

Umounting:

componentWillUnmount():

    只能发生一次。当我们的组件被卸载或者销毁了就会调用，我们可以在这个函数 里去清除一些定时器，取消网络请求，清理无效的 DOM 元素等垃圾清理工作

Error Handing:

componentDidCatch(error, info)

---

#### React 中的 key 是什么，有什么作用

```
  return (
    <div>
      <button onClick={changeCount}>{count}</button>
      {
        arr.map(item => {
          return (
            <p className="border" key={item}>{item}</p>
          )
        })
      }
    </div>
  )
```

作用

1.源码中 ReactChildFiber.js 文件中的 reconcileSingleElement 方法协调，比较老的 key 和最新的 key 是否相等，

再比较 type 是否相等，都相同则判定为同一元素，则复用原来的元素，否则重新创建（createFiberFromFragment）。

2.fiber 都为链表，如数组形式进行对比，upDateSlot 中同上，和 type 一起标识唯一性来更新节点。

3.fiber 拿子元素只能拿到第一个，如果进行两个数组对比，react 的 diff 算法中提供了 mapRemainingChildren 方法，

用 key 和 fiber 当作此方法的 key，value，把链表结构变成了 map 图，获取节点直接用 key 通过 get 就能得到，删除同理。

---

#### Virtual DOM 内核

##### Shadow DOM 和 Virtual DOM 是一回事吗？

    Shadow DOM是一种浏览器技术，主要用于在web组件中封装变量和CSS。

    Virtual DOM则是一种由js类库基于浏览器API实现的概念。

##### 什么是 React Fiber

    Fiber是React16中新的协调引擎。它的主要目的是使Vitural DOM可以进行增量式渲染

##### waht

    用 JavaScript 对象表示 DOM 信息和结构，当状态变更时候，重新渲染这个 JavaScript 的对象结构。这个
    JavaScript对象称为virtual dom;

##### why

    DOM操作很慢，轻微的操作都可能导致⻚⾯重新排版，⾮常耗性能。相对于DOM对象，js对象处理起来更快， ⽽而且更简单。
    通过diff算法对⽐比新旧vdom之间的差异，可以批量的、最⼩化的执行dom操作，从⽽提⾼性能。

##### where

    react中⽤用JSX语法描述视图，通过babel-loader转译 后它们变为React.createElement(...)形式，该函数将⽣生成 vdom来描述
    真实dom。将来如果状态变化，vdom将作出相 应变化，再通过diff算法对⽐新⽼vdom区别从⽽做出最终dom操作。

---

#### jsx

    原理:

      babel-loader会预编译JSX为React.createElement(...)

    与vue的异同:

    react中虚拟dom+jsx的设计⼀一开始就有，vue则是演进 过程中才出现的 jsx本来就是js扩展，转义过程简单直接的多;vue把 template编
    译为render函数的过程需要复杂的编译器器 转换字符串串-ast-js函数字符串串

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
