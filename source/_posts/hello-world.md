---
title: 前端架构--Vue源码分析
---

基于源码的 Vue 相关面试知识点

<!-- Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues). -->

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

组件定义：(src/core/global-api/assets.js)

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

#### Vue 中组件之间的通信

阐述过程（1.分几类；2.使用；3.选用）

方法：

1.props(父子组件简单数据传递)

2.$emit/$on 事件总线

3.vuex(需要保存状态)

4.$parent/$children

5.$attrs/$listeners

6.provide/inject

分三类

1.父子组件通信

2.兄弟组件通信

3.跨层组件通信

---

#### Vue 的性能优化方法

1.路由懒加载

```
const router = new vueRouter({
    routes: [
        { path: '/foo', component:() => import('./Foo.vue')}
    ]
})
```

2.keep-alive 缓存页面

```
<template>
    <div id="app>
        <keep-alive>
            <router-view />
        </keep-alive>
    </div>
</template>
```

3.使用 v-show 复用 DOM

4.v-for 遍历避免同时使用 v-if(选项有判断直接放在计算属性)

5.长列表性能优化
.纯数据展示，不会有任何变化，就不需要做响应化

```
export default {
    data: () => ({
        users: []
    })
}
async created() {
    const users = await axios.get("/api/users");
    this.users = object.freeze(users);
}
```

.如果是大数据长列表，可采用虚拟滚动，只渲染少部分区域的内容

```
<recycle-scroller class="items" :items="items" :item-size="24">
    <template v-slot="{item}">
        <FetchItemView :item="item" @vote="voteItem(item)"/>
    </template>
</recycle-scroller>
```

6.事件的销毁

Vue 组件销毁时，会自动解绑它的全部指令及事件监听器，但仅限于组件本身的事件,和组件关系不大则需要手动在 beforeDestory 清除

7.图片懒加载

页面内未出现在可视区域内的图片先不做加载，滚动到可视区域后再加载

```
<img v-lazy="/static/img/1.png">
```

8.第三方插件按需引入

9.无状态的组件标记为函数式组件

```
<tempalte functional>
    <div>
        <div v-if="props.value" class="on"></div>
        <section v-else class="off"></section>
    </div>
</template>
<script>
export default {
    props: ['value']
}
</script>
```

10.子组件分割

```
<template>
    <div>
        <childComp/>
    </div>
</template>
<script>
export default {
    components: {
        childComp: {
            methods: {
                heavy () { /*耗时任务*/}
            },
            render (h) {
                return h('div', this.heavy())
            }
        }
    }
}
</script>
```

11.变量本地化

```
<template>
    <div>
        {{ result }}
    </div>
</template>

<script>
import { heavy } from '@/utils

export default {
    props: ['start'],
    computed: {
        base () { return 42 },
        result () {
            const base = this.base // 不要频繁引用this.base
            let result = this.start
            for (let i = 0; i < 1000; i++) {
                result += heavy(base)
            }
            return result
        }
    }
}
</script>
```

12.SSR

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

#### computed、watch、methods 对比

1.computed 属性的结果会被缓存，除非依赖的响应式属性变化才会重新计算。

    主要当作属 性来使用;(注:在引用的时候，一定不要加 () 去调用，直接把它当作普通 属性去使用 就好了;
    只要 计算属性，这个 function 内部，所用到的 任何 data 中的数据发送了变 化，就会立即重新计算这个
    计算属性的值;计算属性的求值结果，会被缓存起来，方便 下次直接使用; 如果 计算属性方法中，所有的任
    何数据，都没有发生过变化，则不会重 新对计算属性求值;)适合对于任何复杂逻辑或一个数据属性在它所依赖
    的属性发生变化时，也要发生变化，即一个属性受多个属性影响时使用。

2.methods 方法表示一个具体的操作，主要书写业务逻辑;

3.watch 一个对象，键是需要观察的表达式，值是对应回调函数。

    主要用来监听某些特定数 据的变化，从而进行某些具体的业务逻辑操作;可以看作是 computed 和 methods
    的结合体。需要在数据变化时执行异步或开销较大的操作时使用，即当一条数据影响多条数据的时候。

---

#### Vue 拓展某现有组件

1.使用 Vue.mixin 全局混入

    mixin的调用顺序：混入对象的钩子将在组件自身钩子之前调用，如果遇到全局混入（Vue.mixin），全局混入
    的执行顺序要先于混入和组件里的方法

2.加 slot 拓展

    默认插槽和匿名插槽：slot用来获取组件中的原内容

    具名插槽

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

#### Vuex 使用和理解

## Quick Start

### Create a new post

```bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

```bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

```bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

```bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
