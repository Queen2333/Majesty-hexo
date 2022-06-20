---
title: 前端架构--Vue综合知识点
featured_image: ./images/thor.jpg
---

#### Vue 中组件之间的通信

阐述过程（1.分几类；2.使用；3.选用）

方法：

1.props(父子组件简单数据传递)

2.$emit/$on 事件总线

3.vuex(需要保存状态)

4.$parent/$children

5.$attrs/$listeners/$root

https://blog.csdn.net/Kratial/article/details/108825249

https://blog.csdn.net/qdcainiao/article/details/119741534

6.provide/inject

https://blog.csdn.net/yw00yw/article/details/107341934

分三类

1.父子组件通信

2.兄弟组件通信

3.跨层组件通信

---

#### 父组件调用子组件的方法

```
this.$refs.child.func()
```

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

    SSR是服务端渲染：在后台将vue实例渲染为HTML字符串直接返回，在前端激活为交互程序。

    实现原理：

    客户端发送请求给服务器
    服务器查询数据库，使用视图、模板引擎等拼接成html字符串，返回给客户端
    客户端渲染html
    优点：

    网页内容在服务器端渲染完成，一次性传输到浏览器，所以首屏加载速度非常快
    有利于SEO，因为服务器返回的是一个完整的html，在浏览器可以看到完整的dom，对于爬虫、百度搜索等引擎就比较友好
    缺点：

    在后续跳转其它链接时，整个页面还要重复这样的操作，不断地请求响应、请求响应，相对来说，消耗的带宽资源、后续请求的时间就多了，对服务器造成过多负担。

---

#### Vue 监听数据的三种方法

1.在 input 标签里绑定 keyup 事件

2.watch 监听数据变化:监视 data 中指定数据的变化，然后触发这个 watch 中对应的 function 处理函数，
该方法可以不用绑定事件;

watch 监听路由变化:
`watch: { '$route.path': function (newVal, oldVal) { ... } }`

3.computed 计算属性的使用

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

#### Vue-router 导航钩子

1.全局导航钩子

beforeEach(to, from, next)：路由改变前调用

    常用验证用户权限

    to: 即将要进入的目标路由对象;

    from: 当前导航即将要离开的路由对象;

    next: 路由控制参数

    ```
        next() // 如一切正常，调用此方法进入下一个钩子
        next(false) // 取消导航
        next('route') // 当前导航被中断，然后进行一个新导航
        next('error') // 如果一个error实例，则导航会被终止且该错误会被传递给router.onError()

    ```

afterEach(to, from): 路由改变后的钩子

    常用自动让页面返回最顶端

    用法相似，少了next参数

2.路由配置中导航钩子

beforeEnter(to, from, next)

    用法同上

3.组件内部导航钩子

beforeRouteEnter(to, from, next)

    该组件的对应路由被comfirm前调用

    此时实例未被创建，所以不能获取实例(this)

beforeRouteUpdate(to, from, next)

    当前路由改变，但该组件被复用时调用

    该组件内可以访问实例(this)

beforeRouteLeave(to, from, next)

    当导航离开组件的对应路由时调用

    该组件内可以访问实例(this)

4.路由监测变化

watch 监听$router 对象

---

#### 递归组件

组件可以在他们自己模板中调用自身

一定要有一个结束的条件，可使用 v-if="false"作为递归组件的结束条件。用到递归组件，数据格式是需要
满足的如下：

```
<template>
    <div>
        <ul>
            <li v-for="(item, index) in list" :key="index">
                <p>{{item.name}}</p>
                <tree-menu :list="list">
            </li>
        </ul>
    </div>
</template>
<script>
    export default {
        components: {
            treeMenu
        },
        data() {
            return {
                list: [
                    {
                        name: '全栈工程师',
                        cList: [
                            {name: 'vue'},
                            {
                                name: 'react',
                                cList: [
                                    {
                                        name: 'js',
                                        cList: [{name: 'css'}]
                                    }
                                ]
                            }
                        ]
                    },
                    {name: '高级工程师'}
                ]
            }
        }
    }
</script>

```

#### Vue 必会 api

1.数据相关

    Vue.set(vm.$set): 向响应式对象中追加属性，并希望这个属性也是响应式，且触发视图更新

    Vue.delete(vm.$delete): 删除对象的属性，对象如是响应式的，能确保触发试图更新

2.事件相关

    vm.$on: 监听当前实例上的自定义事件，可由vm.$emit触发。回调函数接收所有传入事件的参数
    并非在子组件派发，在父组件接受，父子组件为同一实例。
    转换成代码形式如下：

    ```
    vm.$on('test', function(msg) {
        console.log(msg)
    })
    ```

    vm.$emit: 触发当前实例上的事件，附加参数会传给监听器回调

    典型应用：事件总线

        通过在Vue原型上添加一个Vue实例作为事件总线，实现组件间相互通信，且不受组件关系的
        影响。（Vue.prototype.$bus = new Vue()）

3.节点引用

    ref和vm.$refs：ref被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的$refs对
    象上。如果在普通的DOM元素上使用，引用指向的就是DOM元素；如果用在子组件上，引用就指向组件

    注意：ref是作为渲染结果被创建的，在初始渲染时不能访问（至少在mounted之后访问）；$refs
    不是响应式的，不要试图用它在模板中做数据绑定；当v-for用于元素或组件时，引用信息将是包含
    DOM节点或组件实例的数组

---

#### 动画

    1.css过度和动画中自动应用class
    2.配合使用第三方css动画库，如animate.css
    3.在过渡钩子函数中使用js直接操作dom
    4.配合使用第三方js动画库，如velocity.js
    5.列表过度利用trasition-group可以对v-for渲染每个元素应用过度
        ```
            <transition-group name="fade">
                <div v-for="c in coures" :key="c.name">
                    {{c.name}} - ${{c.price}}
                    <button @click="addToCart(c)">加购</button>
                </div>
            </transition-group>
        ```

---

#### 过滤器

    可用于对传入的文本进行格式化：双花括号插值和v-bind表达式

    ```
        <!-- 双花括号 -->
        {{ message | capitalize }}

        <!-- v-bind -->
        <div v-bind:id="rawId | formatId"></div>

        filters: {
            capitalize(value, symbol = '¥') {
                return symbol + value
            }
        }
    ```

---

#### 自定义指令

    有复用的功能，并且需要操作dom元素时可使用自定义指令

    ```
        Vue.directive('permission', {
            inserted(el, binding) {
                console.log(binding)
                if (role !== binding.value) {
                    el.parentElement.removeChild()
                }
            }
        })

        <button v-permission="'admin'">
    ```

---

#### 渲染函数

Vue 推荐在绝大多数情况下使用模板创建 html，但在一些场景下需要用 js 的完全编程能力，此时需要用到渲染函数

```
  render: function (createElement) {
    // createElement函数返回的是VNode
    return {
      tag, // 标签名称 {String | Object | Function} 或者resolve上述任何一种async函数
      data, // 传递数据
      children // 子节点数据
    }
  }
```

例子：

```
  <heading :level="1" :title="title" icon="cart">{{title}}</heading>
  <h2 title="">
    <svg class="icon">
      <use xlink:href="#icon-cart"></use>
    </svg>
  </h2> //插值

  Vue.component('heading', {
    props: {
      level: {
        type: String,
        required: true
      },
      title: {
        type: String,
        default: ''
      },
      icon: {
        type: String
      }
    }
    render(h) { // h就是createElement
      // 子节点数组
      let children = []
      // icon属性处理逻辑
      if (this.icon) {
        // <svg class="icon"><use xlink="#icon-cart"></use></svg>
        children.push(h(
          'svg',
          {class: 'icon'},
          [h('use', {attrs: {'xlink:href': '#icon-' + this.icon}})]
        ))
      }

      // 拼接子节点
      children = children.concat(this.$slots.default)
      // snabbdom
      const vnode = h( // 必须返回
        'h' + this.level, // 参数1:tagname
        {attrs: {title: this.title}}, //参数2: {...}
        children, //参数3:子节点VNode数组
      )
      return vnode
    }
  })
```

---

#### 函数式组件

组件没有任何管理状态，也没有监听任何传递给他的状态，也没有生命周期方法时，可以将组件标记为 functional，
这意味着它无状态（没有响应式数据），也没有实例（没有 this 上下文）

```
  Vue.component('heading',{
    functional: true, // 函数式组件
    props: {...},
    render(h, context) {
      // 子节点数组
      let children = []

      // 属性获取
      const {icon, title, level} = context.props
      // icon属性处理逻辑
      if (icon) {
        // <svg class="icon"><use xlink="#icon-cart"></use></svg>
        children.push(h(
          'svg',
          {class: 'icon'},
          [h('use', {attrs: {'xlink:href': '#icon-' + icon}})]
        ))
      }

      // 拼接子节点
      children = children.concat(context.children)
      // snabbdom
      const vnode = h( // 必须返回
        'h' + level, // 参数1:tagname
        {attrs: {title}}, //参数2: {...}
        children, //参数3:子节点VNode数组
      )
      return vnode
    }
  })
```

---

#### 混入 mixin

提供了一种非常灵活的方式，来分发 vue 组件中的可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入
对象时，所有混入对象的选项将被“混合”进入该组件本身的选项（选项合并组件中的值优先级高，生命周期都保留）

    ```
      // 定义一个混入对象
      var myMixin = {
        created: function () {
          this.hello()
        },
        methods: {
          hello: function () {
            console.log('hello from mixin!')
          }
        }
      }

      Vue.component('comp', {
        mixins: [myMixin]
      })
    ```

---

#### 插件

用于为 Vue 添加全局功能范围如下：

    1.添加全局方法或属性，如：vue-custom-element

    2.添加全局资源：指令/过滤器/过渡等，如：vue-touch

    3.通过全局混入来添加一些组件选项，如：vue-router

    4.添加Vue实例方法，通过把他们添加到Vue.prototype上实现

    5.一个库，提供自己的Api，同时提供上面提到的一个或多个功能，如：vue-router

##### 插件声明

    Vue的插件应该暴露一个install方法，这个方法的第一个参数是Vue构造器第二个参数是一个可选的选项对象

    ```
      MyPlugin.install = function(Vue, options) {
        // 1.添加全局方法成属性
        Vue.myGlobalMethod = function () {}
        // 2.添加全局资源
        Vue.directive('my-directive', {})
        // 3.注入组件选项
        Vue.mixin({
          created: function() {
            // ...逻辑
          }
        })
        // 4.添加实例方法
        Vue.prototype.$myMethod = function (methodOptions) {}
      }

      eg:

      // 插件需要实现
      const MyPlugin = {
        install(Vue, options) {
          Vye.component('heading', {
            //
          })
        }
      }

      if(typeof window !== 'undefined' && window.Vue) {
        //使用插件使用Vue.use()
        window.Vue.use(MyPlugin)
      }

      // 引入
      <script src="./plugins/heading.js"></script>
    ```

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


##### 和react的key做对比

https://blog.csdn.net/newway007/article/details/104900608

https://www.zhoulujun.cn/html/webfront/ECMAScript/vue/8295.html


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

##### 和react的diff做比较

https://github.com/pfan123/Articles/issues/62


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

    activated: keep-alive组件激活时调用，类似created没有真正创建，只是激活

    deactivated: keep-alive组件停用时调用，类似destroyed没有真正移除，只是禁用

    在2.2.0及其更高版本中，activated和deactivated将会在树内的所有嵌套组件中触发

##### 父子组件生命周期顺序

https://m.html.cn/qa/vue-js/22535.html

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

    4.触发时机：同一事件循环中的数据变化后，dom完成更新，立即执行nextTick的回调

https://www.cnblogs.com/leiting/p/13174545.html

---

#### Vue 响应式原理

设计思想：观察者模式

Dep 对象：Dependency 依赖中心。依赖的简写。包含三个主要属性：id，subs，target 和四个主要函数 addSub，removeSub，depend，
notify，是观察者的依赖集合，负责在数据发生改变时，使用 notify 触发保存在 subs 下的订阅列表，依次更新数据和 DOM

Observer 对象：即观察者，包含两个主要属性 value，dep。做法是使用 getter、setter 方法覆盖默认的取值和赋值操作，将对象
封装为响应式对象，每一次调用时更新依赖列表，更新值时触发订阅者。绑定在对象的 ob 原型链属性上

Watcher： 订阅者，当某个观察者观察到数据发生变化的时候，这个变化经过消息调度中心，最终会传递到所有该观察者对应的订阅者身上，然后这些订阅者分别执行自身的业务回调即可

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

https://juejin.cn/post/6868085927685095437

---

#### vue双向绑定的原理

1）v-model 属性
针对于 input 的 v-model 双向数据绑定实际上就是通过子组件中的 $emit 方法派发 input 事件，父组件监听 input 事件中传递的 value 值，并存储在父组件 data 中；然后父组件再通过 prop 的形式传递给子组件 value 值，再子组件中绑定 input 的 value 属性即可。
其他元素使用 v-model 双向数据绑定实际上就是，通过监听 change 事件。以及$emit 方法派发，再通过 prop 的形式传递。

2）.sync 修饰符
父组件向子组件传递数据的方式有多种，props 是其中的一种，但是它的局限在于数据只能单向传递，子组件不能直接修改 prop 属性，但是碰到子组件需要修改父组件的情况怎么办呢？
父组件中并没有定义过 update 事件，但是却可以完成 prop 属性 page 的修改，这就是 sync 语法糖的作用。

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

#### vue插槽

https://juejin.cn/post/6844903920037281805#heading-2

---

#### keep-alive

用法 https://juejin.cn/post/6844903919273918477

原理 https://bbchin.com/archives/source-keep-alive

---

#### v-if和v-show

v-show（会导致回流）

v-show不管条件是真还是假，第一次渲染的时候都会编译出来，也就是标签都会添加到DOM中。之后切换的时候，通过display: none;样式来显示隐藏元素。可以说只是改变css的样式，几乎不会影响什么性能。

v-if（增删dom）

在首次渲染的时候，如果条件为假，什么也不操作，页面当作没有这些元素。当条件为真的时候，开始局部编译，动态的向DOM元素里面添加元素。当条件从真变为假的时候，开始局部编译，卸载这些元素，也就是删除。

性能方面
v-if绝对是更消耗性能的，因为v-if在显示隐藏过程中有DOM的添加和删除，v-show就简单多了，只是操作css。

使用场景
因为v-show无论如何都会渲染，如果在一些场景下很难出现，那么使用v-if。如果是一些固定的，条件内容都不怎么会改变的，频繁切换的，使用v-show会比较省性能。如果是子组件，每次切换子组件不执行生命周期，使用v-show，如果子组件需要重新执行生命周期，那么使用v-if才能触发。

---

#### vue的数组操作方法

https://juejin.cn/post/6941348666422919204

vue的set方法原理是splice()

---

#### vue数据变化视图不更新的情况

https://segmentfault.com/a/1190000022772025

---

#### vue更新数组时触发视图更新的方法

https://blog.csdn.net/weixin_57550930/article/details/120422295

---

#### Vue 常用修饰符

https://blog.csdn.net/m0_64969829/article/details/122881221

---

#### Vue在父组件中使用子组件生命周期

https://www.cnblogs.com/rainbowLover/p/13229003.html

---