---
title: js原理coding题
featured_image: ./images/hawkeye.webp
---

#### instanceof的原理

```js
// 右边变量的原型存在于左边变量的原型链上
function instanceOf(left, right) {
  let leftValue = left.__proto__
  let rightValue = right.prototype
  while (true) {
    if (leftValue === null) {
      return false
    }
    if (leftValue === right) {
      return true
    }
    leftValue = rightValue.__proto__
  }
}
```

---

#### Event Bus

```js
// 组件通信，一个触发与监听的过程
class EventEmitter {
  constructor () {
    // 存储事件
    this.events = this.events || new Map()
  }
  // 监听事件
  addListener (type, fn) {
    if (!this.events.get(type)) {
      this.events.set(type, fn)
    }
  }
  // 触发事件
  emit (type) {
    let handle = this.events.get(type)
    handle.apply(this, [...arguments].slice(1))
  }
}

// 测试
let emitter = new EventEmitter()
// 监听事件
emitter.addListener('ages', age => {
  console.log(age)
})
// 触发事件
emitter.emit('ages', 18)  // 18
```

---

#### 双向数据绑定

```js
let obj = {}
let input = document.getElementById('input')
let span = document.getElementById('span')
Object.defineProperty(obj, 'text', {
  configurable: true,
  enumerable: true,
  get() {
    console.log('获取数据了')
    return obj.text
  },
  set(newVal) {
    console.log('数据更新了')
    input.value = newVal
    span.innerHTML = newVal
  }
})
input.addEventListener('keyup', function(e) {
  obj.text = e.target.value
})
```

---

#### 实现一个简单路由

```js
class Route{
  constructor(){
    // 路由存储对象
    this.routes = {}
    // 当前hash
    this.currentHash = ''
    // 绑定this，避免监听时this指向改变
    this.freshRoute = this.freshRoute.bind(this)
    // 监听
    window.addEventListener('load', this.freshRoute, false)
    window.addEventListener('hashchange', this.freshRoute, false)
  }
  // 存储
  storeRoute (path, cb) {
    this.routes[path] = cb || function () {}
  }
  // 更新
  freshRoute () {
    this.currentHash = location.hash.slice(1) || '/'
    this.routes[this.currentHash]()
  }
}
```

---

#### 实现懒加载

```html

<ul>
  <li><img src="./imgs/default.png" data="./imgs/1.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/2.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/3.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/4.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/5.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/6.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/7.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/8.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/9.png" alt=""></li>
  <li><img src="./imgs/default.png" data="./imgs/10.png" alt=""></li>
</ul>
```
```js
let imgs =  document.querySelectorAll('img')
// 可视区高度
let clientHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight
function lazyLoad () {
  // 滚动卷去的高度
  let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
  for (let i = 0; i < imgs.length; i ++) {
    // 得到图片顶部距离可视区顶部的距离
    let x = clientHeight + scrollTop - imgs[i].offsetTop
    // 图片在可视区内
    if (x > 0 && x < clientHeight+imgs[i].height) {
      imgs[i].src = imgs[i].getAttribute('data')
    }
  }
}
setInterval(lazyLoad, 1000)
```

---

#### rem实现原理

```js
function setRem () {
  let doc = document.documentElement
  let width = doc.getBoundingClientRect().width
  // 假设设计稿为宽750，则rem为10px
  let rem = width / 75
  doc.style.fontSize = rem + 'px'
}
```

---

#### 实现AJAX

```js

// 1. 简单实现

// 实例化
let xhr = new XMLHttpRequest()
// 初始化
xhr.open(method, url, async)
// 发送请求
xhr.send(data)
// 设置状态变化回调处理请求结果
xhr.onreadystatechange = () => {
  if (xhr.readyStatus === 4 && xhr.status === 200) {
    console.log(xhr.responseText)
  }
}

// 2. 基于promise实现

function ajax (options) {
  // 请求地址
  const url = options.url
  // 请求方法
  const method = options.method.toLocaleLowerCase() || 'get'
  // 默认为异步true
  const async = options.async
  // 请求参数
  const data = options.data
  // 实例化
  const xhr = new XMLHttpRequest()
  // 请求超时
  if (options.timeout && options.timeout > 0) {
    xhr.timeout = options.timeout
  }
  // 返回一个Promise实例
  return new Promise ((resolve, reject) => {
    xhr.ontimeout = () => reject && reject('请求超时')
    // 监听状态变化回调
    xhr.onreadystatechange = () => {
      if (xhr.readyState == 4) {
        // 200-300 之间表示请求成功，304资源未变，取缓存
        if (xhr.status >= 200 && xhr.status < 300 || xhr.status == 304) {
          resolve && resolve(xhr.responseText)
        } else {
          reject && reject()
        }
      }
    }
    // 错误回调
    xhr.onerror = err => reject && reject(err)
    let paramArr = []
    let encodeData
    // 处理请求参数
    if (data instanceof Object) {
      for (let key in data) {
        // 参数拼接需要通过 encodeURIComponent 进行编码
        paramArr.push(encodeURIComponent(key) + '=' + encodeURIComponent(data[key]))
      }
      encodeData = paramArr.join('&')
    }
    // get请求拼接参数
    if (method === 'get') {
      // 检测url中是否已存在 ? 及其位置
      const index = url.indexOf('?')
      if (index === -1) url += '?'
      else if (index !== url.length -1) url += '&'
      // 拼接url
      url += encodeData
    }
    // 初始化
    xhr.open(method, url, async)
    // 发送请求
    if (method === 'get') xhr.send(null)
    else {
      // post 方式需要设置请求头
      xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded;charset=UTF-8')
      xhr.send(encodeData)
    }
  })
}
```
---

#### 实现拖拽

```js
window.onload = function () {
  // drag处于绝对定位状态
  let drag = document.getElementById('box')
  drag.onmousedown = function(e) {
    var e = e || window.event
    // 鼠标与拖拽元素边界的距离 = 鼠标与可视区边界的距离 - 拖拽元素与边界的距离
    let diffX = e.clientX - drag.offsetLeft
    let diffY = e.clientY - drag.offsetTop
    drag.onmousemove = function (e) {
      // 拖拽元素移动的距离 = 鼠标与可视区边界的距离 - 鼠标与拖拽元素边界的距离
      let left = e.clientX - diffX
      let top = e.clientY - diffY
      // 避免拖拽出可视区
      if (left < 0) {
        left = 0
      } else if (left > window.innerWidth - drag.offsetWidth) {
        left = window.innerWidth - drag.offsetWidth
      }
      if (top < 0) {
        top = 0
      } else if (top > window.innerHeight - drag.offsetHeight) {
        top = window.innerHeight - drag.offsetHeight
      }
      drag.style.left = left + 'px'
      drag.style.top = top + 'px'
    }
    drag.onmouseup = function (e) {
      this.onmousemove = null
      this.onmouseup = null
    }
  }
}
```