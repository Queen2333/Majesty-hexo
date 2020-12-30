---
title: 前端架构--Vue源码分析
---

基于面试的 Vue 相关知识点

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
