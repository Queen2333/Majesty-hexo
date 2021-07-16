---
title: 前端架构--Vue高级用法
featured_image: ./images/ironman3.jpg
---

#### Vue 中组件之间的通信

阐述过程（1.分几类；2.使用；3.选用）

方法：

1.props(父子组件简单数据传递)

2.$emit/$on 事件总线

3.vuex(需要保存状态)

4.$parent/$children

5.$attrs/$listeners/$root

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

####

<!--
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

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html) -->
