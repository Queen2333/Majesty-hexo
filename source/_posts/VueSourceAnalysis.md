---
title: 前端架构--Vue源码分析
featured_image: ./images/ironman1.jpg
---

#### v-if 和 v-for 的优先级

1.v-for 优先于 v-if

2.如果同时出现，每次都会先执行循环再条件判断，浪费性能

3.避免方法可在外层嵌套 template，在这一层进行 v-if

4.如果循环内部子元素有相应判断条件，应该使用计算属性过滤出要展示的元素来进行优化，而不是使用 v-if

---

#### Vue 组件 data 为什么必须是函数，而 Vue 的根实例没有此限制

1.Vue 组件可能存在多个实例，如果用对象形式定义 data，会导致他们共用一个 data 对象，状态改变将影响所有组件实例

2.采用函数形式定义，在 initData 时会将其作为工厂函数返回全新 data 对象，有效避免实例间状态污染

3.根实例只能有一个，不需要担心这种情况

4.（源码角度）源码中，数据初始化的部分会检测 data 形式从而执行具体方式。根实例创建对象时，合并选项会有实例能拿到，这是只有根实例才有的，可以有效躲过 data 形式的校验。普通组件一开始不存在实例，所以无法躲过检验的 if 逻辑，从而被检测到类型不符，无法跳过。

---

#### Vue 中的 key 的作用和工作原理

1.key 的主要作用是为了高效更新虚拟 DOM，从源码角度，原理就是在 patch 过程中，会执行 patchVnode，这个过程中又会执行 updateChildren，它会去更新所有两个新旧子元素，这个过程中通过 key 可以精准判断两个节点是否是同一个，从而避免频繁更新不同元素，使整个 patch 更高效，减少 DOM 操作。

2.不设置 key 可能在列表更新时引发一些隐蔽 bug（如：更新了没有更新节点）

3.vue 在使用相同标签名元素的过渡切换时，也会使用到 key 属性。目的也是让 vue 可以区分它们，否则 vue 只会替换内部属性而不会触发过渡效果

---

#### Vue 中的 diff 算法

diff 算法并非 vue 专用，凡是涉及虚拟 DOM 都用到了 diff 算法

必要性：(lifecycle.js-mountComponent()) 组建中可能存在很多个 data 中的 key 使用

执行方式：(patch.js-patchVnode()) patchVnode 是 diff 发生的地方，整体策略：深度优先，同层比较

高效性：(patch.js-updateChildren)

1.diff 算法是虚拟 DOM 技术的必然产物：通过新旧虚拟 DOM 做对比（即 diff），将变化的地方更新在真实 DOM 上；另外也需要 diff 高效的执行对比过程，从而降低时间复杂度为 O(n)

2.Vue2.x 中为了降低 Watcher 粒度，每个组件只有一个 Watcher 与之对应，只有引入 diff 才能精确找到发生变化地方

3.vue 中的 diff 执行的时刻是组件实例执行其更新函数时，它会对比上一次渲染结果 oldVnode 和新的渲染结果 newVnode，此过程为 patch

4.diff 过程整体遵循深度优先，同层比较的策略；两个节点比较会根据它们是否拥有子节点或者文本节点做不同操作；比较两组子节点的算法是重点，首先假设头尾节点可能相同做 4 次对比尝试，如果没有找到相同节点才按照通用方式遍历查找，查找结束再按情况处理剩下的节点；借助 key 通常可以非常精确找到相同节点，因此整个 patch 过程非常高效

---

#### Vue 组件化的理解

组件定义：(src/core/global-api/assets.js)可复用的 Vue（VueComponent）实例

组件化优点：(lifecycle.js-mountComponent()) 每个组件有 watcher 与之对应，合理切割可以减少重新渲染面积

组件化实现：(构造函数，src/core/global-api/extend.js;实例和挂载，src/core/vdom/patch.js-createElm())

1.组件是独立和可复用的代码组织单元。组件系统是 Vue 核心特性之一，她使开发者使用小型、独立和通常可复用的组件构建大型应用

2.组件化开发能大幅提高应用开发效率、测试性、复用性等

3.组件使用分类有：页面组件、业务组件、通用组件

4.vue 是基于配置的，我们通常编写的组件是组件配置而非组件，框架后续会生成其构造函数，他们基于 VueComponent 类，扩展于 Vue。扩展过程中会继承 vue 中已有的选项

5.vue 中常见的组件化技术有：prop，自定义事件，插槽等，他们主要用于组件通信、拓展等

6.合理划分组件，有助于提升应用性能

7.组件应该是高内聚、低耦合

8.遵循单项数据流的原则（修改数据提交给父组件更改或使用 vuex 更改）

---

#### Vue 的设计理念

1.渐进式 js 框架：自底向上逐层应用

2.易用性：响应式

3.灵活性：规模小可只保留核心功能，规模大则引入工具

4.高效性：虚拟 DOM，diff 算法，proxy

---

#### 对 MVC、MVP 和 MVVM 的理解

1.三者都是框架模式，他们设计目标都是为了解决 Model 和 View 的耦合问题

2.MVC 模式出现较早主要应用在后端，如 Spring MVC、ASP.NET MVC 等，在前端领域的早期也有应用，如 BackBone.js。优点是分层清晰，缺点是数据流混乱，灵活性带来的维护性问题

3.MVP 模式是 MVC 的进化形式，Presenter 作为中间层负责 MV 通信，解决了两者耦合问题，但 P 层过于臃肿会导致维护问题

4.MVVM 模式在前端领域有广泛应用，它不仅解决 MV 耦合问题，还同时解决了维护两者映射关系的大量繁杂代码和 DOM 操作代码，在提高开发效率、可读性同时还保持了优越的性能问题

---

#### Vue3.0 新特性

更快： 1.虚拟 DOM 重写 2.优化 slots 的生成 3.静态树提升 4.静态属性提升 5.基于 Proxy 的响应式系统

更小： 通过摇树优化核心库体积

更容易维护： TypeScript + 模块化

更友好： 跨平台，编译器核心和运行时核心与平台无关，使得 Vue 更容易与任何平台（Web、Android、ios）一起使用

更容易使用： 1.改进的 TypeScript 支持，编辑器能提供强有力的类型检查和错误及警告 2.更好的调试支持 3.独立的响应式模块 4.Composition API

1.虚拟 DOM 重写

期待更多的编译时提示减少运行时开销，使用更有效的代码创建虚拟节点。

组件快速路径+单个调用+子节点类型检测

    跳过不必要的条件分支

    js引擎更容易优化

2.优化 slots 生成

vue3 中可以单独重新渲染父级和子级

    确保实例正确的跟踪依赖关系

    避免不必要的父子组件重新渲染

3.静态树提升

使用静态树提升，意味着 vue 3 的编译器能够检测到什么是静态的，然后将其提升，从而降低了渲染成本

    跳过修补整棵树，从而降低渲染成本

    即使多次出现也能正常工作

4.静态属性提升

使用静态属性提升，Vue 打补丁时将跳过这些属性不会改变的节点

5.基于 Proxy 的数据响应式

Vue2 的响应式系统使用 Object.defineProperty 的 getter 和 setter。Vue3 将使用 ES2015 Proxy 作为其观察机制，这将带来以下变化

    组件实例初始化的速度提升100%

    使用Proxy节省以前一半的内存开销，加快速度，但是存在低浏览器版本的不兼容

    为了继续支持IE11，Vue3将发布一个支持旧观察机制和新Proxy版本的构建

6.高可维护性

Vue3 将带来更可维护的源代码，它不仅会使用 TypeScript，而且许多包被解耦，更加模块化

---

#### 对 Vue 生命周期的理解（结合生命周期图）

1.实例化 new Vue()

    实例化之后，会执行以下操作。根据Vue源码，可看到Vue本质就是一个function。new Vue()的过程就是初始化
    参数、生命周期、事件等一系列过程（src/core/instance/index.js）

2.初始化事件 生命周期函数

    此时这个对象身上只有默认的一些生命周期和默认事件，其他东西未被创建

3.beforeCreated(创建前)

    在实例初始化之后，数据观测（data observe）和event/watcher事件配置之前就被调用。此时拿不到data和
    props里面的数据，data和methods中的数据还没初始化

4.注射响应

    injection(注射器) reactivity(响应)给数据添加观察者

5.created(创建后)

    在实例创建后被立即调用，此时实例已完成数据观测，属性和方法的运算，watch/event事件回调，挂载阶段还
    没开始，$el尚不可用。因为上一步给数据添加了观察者，所以现在可访问到data里的数据。此钩子常用，可以
    请求数据。如果要调用methods中的方法或者操作data中的数据，要在created里操作。因为请求数据是异步的，
    所以发送请求宜早不宜迟。（src/core/instance/init.js initMixin）

6.是否存在 el

    el指明挂载目标，这个步骤就是判断是否有el，如果没有就判断有没有调用实例上的$mount('')方法，这两个是
    等价的。（src/core/instance/init.js）

7.判断是否有 template

    如果有则渲染template里的内容
    如果没有则渲染el指明的挂载对象里的内容（src/platforms/web/entry-runtime-with-compiler.js $mount）

8.beforeMount(挂载前)

    挂载之前被调用，相关render函数首次被调用

9.替换 el

    这时会在实里例上创建一个el（vm.$el），替换掉原来的el，也是真正的挂载

10.mounted(挂载后)

    实例挂载后调用此时el已被新创建的vm.$el替换，若根实例挂载到了文档上的元素上，当mounted调用时vm.$el
    也在文档内。注意mounted不会保证所有子组件一起挂载。DOM已加载完成，可以操作DOM了。只要执行完mounted，
    就代表整个vue实例已经初始化完毕，一般在此操作DOM
    （src/platforms/web/runtime/index.js $mount -> src/core/instance/lifecycle.js mountComponent）

11.dataChange

    当数据变化时：

    在beforeUpdate发生在虚拟dom打补丁前，这时适合在更新前访问现有dom，如手动移除已添加的事件监听器
    在beforeUpdate(更新前)和updated(更新后)之间会进行DOM的重新渲染和补全
    （src/core/instance/lifecycle.js）

    接着是updated（src/core/observer/scheduler.js callUpdatedHooks）：组件dom已更新，可执行依赖于
    dom的操作，多数情况下应在此期间更改状态。
    如需改变，最好使用watcher或计算属性取代。注意updated不会保证所有子组件一起被重绘

12.callDestory(src/core/instance/lifecycle.js lifecycleMixin $destroy)

    beforeDestory(销毁前)和destroy(销毁后)这两个钩子是需要我们手动调用实例上的$destroy方法才会触发

    当$destroy方法调用后

    beforeDestroy销毁前触发：此时实例仍可用

    移除数据劫持、事件监听、子组件属性 所有东西还保留只是不能修改

    destroy销毁后触发： Vue实例所有指令被解绑，所有事件监听器被移除，所有子实例也被销毁

13.新增钩子

    activated: keeo-alive组件激活时调用，类似created没有真正创建，只是激活

    deactivated: keep-alive组件停用时调用，类似destroyed没有真正移除，只是禁用

    在2.2.0及其更高版本中，activated和deactivated将会在树内的所有嵌套组件中触发

---

#### nextTick 原理

vue 如何检测到 DOM 更新完毕：MutationObserver API
MutationObserver：是 HTML5 新增属性，用于监听 DOM 修改事件，能见听到节点的属性、文本内容、子节点等的改动

```
var observer = new MutationObserver(function() {
    console.log('DOM被修改了')
})
var article = document.querySelector('article')
observer.observer(article)
```

vue 中的 nextTick 实现(src/core/util/env.js):

    如果检测到浏览器支持MutationObserver则创建一个文本节点，监听文本节点的改动，以此触发nextTickHandler，也就
    是dom更新完毕的回调的执行

事件循环机制（Event Loop）

    浏览器使用事件循环机制协调各种事件的处理。
    事件循环会维护一个或多个任务队列，事件作为任务源往队列中加入任务，每执行完一个就从队列中移除它，这就是一次事
    件循环

    setTimeOut就是在队列末尾加入了一个任务

    每次事件循环最后都会有一个ui render步骤即更新dom
    for循环同属一个task，浏览器只在task执行完后进行一次dom更新

    所以vue运用此思路，并不是用MutationObserver监听dom，而是用队列控制的方式达到目的

    vue数据响应过程包含：数据更改->通知watcher->更新dom。而数据更改不受我们控制，可能在任何时候发生，如果刚好
    在重绘之前，就会发生多次渲染，性能浪费，这是vue不希望的。

    于是vue的队列控制还需要了解microtask

microtask 微任务

    每次事件循环都包含一个微任务队列，在循环结束后依次执行队列中的微任务并移除，再开始下一次事件循环

    宏任务要等为任务执行完才能执行，微任务有更高的优先级

    microtask此特性是做队列控制的最佳选择，vue进行DOM更新内部也是调用nextTick来做异步队列控制。当我们调用
    nextTick，它就在更新DOM的那个微任务后追加了我们自己的回调函数，从而确保我们的代码在DOM更新后执行，也避
    免了setTimeOut可能存在的多次执行问题

    常见微任务：Promise、MutationObserver、Object.observer(已废弃)、以及node.js中的process.nextTick

    vue用MutationObserver是想利用其微任务特性而并非做DOM监听，用不用都行，在vue2.5版本中已删去了
    MutationObserver相关代码，因为它是HTML5新增的特性，在IOS上尚有bug

    最优的微任务策略就是Promise，但是Promise是es6新特性，也存在兼容问题，于是vue面临降级策略

Vue 的降级策略

    如果当前环境不支持Promise，vue只能降级为宏任务(macrotask)来做队列控制

    setTimeOut执行的最小时间间隔是4ms左右，会比较延迟，不是最优方案

    在vue2.5源码中，宏任务降级方案依次为：setimmadiate, MessageChannel, setTimeOut
    setimmadiate最理想但是只有IE和node.js支持
    MessageChannel的onmessage回调也是microtask，但也是个新API，面临兼容问题
    setTimeOut是兜底方案

总结：

    1.vue用异步队列的方式控制DOM更新和nextTick回调先后执行

    2.microtask因为其优先级特性，能确保队列中的微任务在一次事件循环前被执行完毕

    3.因为兼容性问题，vue不得不做了microtask向macrotask的降级方案

---

#### Vue 双向数据绑定(响应式)原理

设计思想：观察者模式

Dep 对象：Dependency 依赖的简写。包含三个主要属性：id，subs，target 和四个主要函数 addSub，removeSub，depend，
notify，是观察者的依赖集合，负责在数据发生改变时，使用 notify 触发保存在 subs 下的订阅列表，依次更新数据和 DOM

Observer 对象：即观察者，包含两个主要属性 value，dep。做法是使用 getter、setter 方法覆盖默认的取值和赋值操作，将对象
封装为响应式对象，每一次调用时更新依赖列表，更新值时触发订阅者。绑定在对象的 ob 原型链属性上

vue 数据双向绑定通过‘数据劫持’ + 订阅发布模式实现

数据劫持: 指的是在访问或者修改对象的某个属性时，通过一段代码拦截这个行为，进行 额外的操作或者修改返回结果 典型的有

    1.Object.defineProperty()
    2.es6 中 Proxy 对象(兼容性不太好)
    vue2.x 使用 Object.defineProperty();
    vue3.x 使用 Proxy;

订阅发布模式: 对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依 赖于它的对象都将得到通知 订阅发布模式
中事件统一由处理中心处理，订阅者发布者互不干扰。

优点: 实现更多的控制，做权限处理，节流控制之类，例如:发布了很多消息，但是不是 所有订阅者都要接收

vue 中如何实现:

    利用 Object.defineProperty();把内部解耦为三部分
    Observer: 递归的监听对象上的所有属性，当属性改变时触发对应的 watcher
    watcher(观察者): 当监听的数据值修改时，执行相应的回调函数，更新模板内容 dep:链接 observer 和 watcher，每
    一个observer 对应一个 dep,内部维护一个数组，保存 与该 observer 相关的 watcher
    proxy 实现观察者模式: 观察者模式(Observer mode)指的是函数自动观察数据对象，一 旦对象有变化，函数就会自动执
    行

---

#### Object.defineProperty 和 Proxy 的区别

defineproperty 只能监听某个属性不能全对象监听，proxy 不用设置具体属性

    defineproperty监听需要知道那个对象的那个属性，而proxy只需要知道那个对象就可以了。也就是会省去for in 循环
    提高了效率

proxy 不需要借助外部 value，也有单独相配的对象即 Reflect

    eg：
    ```
    var ob={a:1,b:2}
    ```
    在proxy的get里面有target，key，receiver三个值，其中target是对象ob，key是ob.a，receiver是，set里面除了这
    三个额外多加了一个value，value是传出来的新值。所以在get里return的就是target[key]，set里面return的是
    target[key]=value或者用proxy里的Reflect.set(target，key，value）这样写优雅一点

不会污染原对象（关键区别）

    proxy去代理了ob，他会返回一个新的代理对象不会对原对象ob进行改动，而defineproperty是去修改元对象，修改元对
    象的属性，而proxy只是对元对象进行代理并给出一个新的代理对象

---

#### 模板语法的实现原理

Vue 通过它的编译器将模板编译成渲染函数，在数据发生变化时在此执行渲染函数，通过对比两次执行结果得
出要做的 dom 操作

---
var data = [
{
text: '一部'，
children: [
{
text: '一区',
children: [
{text:''},
{text: '一组'},
{text: '二组'}
]
},
{
text: '二区',
children: [
{text:''},
{text: '一组'},
{text: '二组'}
]
},
]
},{
text: '二部'，
children: [
{
text: '一区',
children: [
{text:''},
{text: '一组'},
{text: '二组'}
]
}
]
}

]
VM1084:3 