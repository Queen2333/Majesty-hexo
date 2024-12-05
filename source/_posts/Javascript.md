---
title: js面试知识点综合
featured_image: ./images/blackwidow.jpg
---


#### async/await

async 作为一个关键字放到函数前面，它的调用会返回一个 promise  对象。如果 async  函数中有返回值 ,当调用该函数时，内部会调用 Promise.solve()  方法把它转化成一个 promise  对象作为返回, 但如果 timeout  函数内部抛出错误，就会调用 Promise.reject()  返回一个 promise 对象，要想获取到 async  函数的执行结果，就要调用 promise 的 then  或 catch  来给它注册回调函数，如果只是 async,  和 promise  差不多，但有了 await 就不一样了， await  关键字只能放到 async  函数里面，await 是等待的意思，那么它等待什么呢，它后面跟着什么呢？其实它后面可以放任何表达式，不过我们更多的是放一个返回 promise  对象的表达式，它等待的是 promise  对象的执行完毕，并返回结果。

```js
  function getCallSettings() {
    return utils.ajax({
      url: '/dialer/dialerSetting',
      method: "GET",
    });
  }
  async function dealData() {
    try {
      let settingInfo = await getCallSettings(); // await会暂停在这，直到promise决议(请求返回)
      // do something……
    }
    catch(err) {
      console.log(err);
    }
  }
  dealData();
```

---

#### Promise

当出现第三个ajax(甚至更多)仍然依赖上一个请求时，我们的代码就变成了一场灾难。这场灾难，往往也被称为回调地狱。

因此我们需要一个叫做Promise的东西，来解决这个问题。

当然，除了回调地狱之外，还有一个非常重要的需求：为了代码更加具有可读性和可维护性，我们需要将数据请求与数据处理明确的区分开来。

一、 Promise对象有三种状态，他们分别是：

•pending: 等待中，或者进行中，表示还没有得到结果
•resolved(Fulfilled): 已经完成，表示得到了我们想要的结果，可以继续往下执行
•rejected: 也表示得到结果，但是由于结果并非我们所愿，因此拒绝执行

这三种状态不受外界影响，而且状态只能从pending改变为resolved或者rejected，并且不可逆。在Promise对象的构造函数中，将一个函数作为第一个参数。而这个函数，就是用来处理Promise的状态变化。

二、 Promise对象中的then方法，可以接收构造函数中处理的状态变化，并分别对应执行。then方法有2个参数，第一个函数接收resolved状态的执行，第二个参数接收reject状态的执行。

then方法的执行结果也会返回一个Promise对象。因此我们可以进行then的链式执行，这也是解决回调地狱的主要方式。

then(null, function() {}) 就等同于catch(function() {})

手动实现一个promise：

```js
// ①自动执行函数，②三个状态，③then
class Promise {
  constructor (fn) {
    // 三个状态
    this.state = 'pending'
    this.value = undefined
    this.reason = undefined
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled'
        this.value = value
      }
    }
    let reject = value => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        this.reason = value
      }
    }
    // 自动执行函数
    try {
      fn(resolve, reject)
    } catch (e) {
      reject(e)
    }
  }
  // then
  then(onFulfilled, onRejected) {
    switch (this.state) {
      case 'fulfilled':
        onFulfilled()
        break
      case 'rejected':
        onRejected()
        break
      default:
    }
  }
}
```

三、Promise中的数据传递

四、Promise.all

当有一个ajax请求，它的参数需要另外2个甚至更多请求都有返回结果之后才能确定，那么这个时候，就需要用到Promise.all来帮助我们应对这个场景。

Promise.all接收一个Promise对象组成的数组作为参数，当这个数组所有的Promise对象状态都变成resolved或者rejected的时候，它才会去调用then方法。（如果都成功，then接收到的是promise对象数组，否则走到catch）

五、 Promise.race

原理 https://juejin.cn/post/7004786857389064205
利用 Promise.race 提前终止多余异步请求 https://www.365seal.com/y/elnW5P6KVr.html

```js
/**
 *  解决异步操作冲突的问题
 *  多次触发后，丢弃未响应的结果，只有最新的一次会进入 then回调中
 *  @template T
 *  @param {T & function} fn 需要包装的异步操作
 *  @return T
 */
export function useLatestCall(fn) {
  //  “假” promise 中的 reject 方法
  let prePromiseReject
  return function() {
    // 如果存在，就行终止
    prePromiseReject?.(new Error('新的promise执行，旧promise被丢弃'))
    // 创建“假” promise，并将reject赋值给闭包变量
    const fakePromise = new Promise((resolve, reject) => (prePromiseReject = reject))
    return Promise.race([fakePromise, fn(...arguments)])
        .finally(() => (prePromiseReject = null))
  }
}

const fetchData = useLatestCall(getXXXXX)
const reloadPage = () => {
  // 只有最近触发的一次会进入 then 回调
  fetchData(params).then(res => {
    // xxx
  })
}
```

与Promise.all相似的是，Promise.race都是以一个Promise对象组成的数组作为参数，不同的是，只要当数组中的其中一个Promsie状态变成resolved或者rejected时，就可以调用.then方法了。而传递给then方法的值也会有所不同。（then接收到的参数为第一个成功的promise对象）


##### Promise并发控制数量的方法

```js
async function asyncPool(poolLimit, array, iteratorFn) {
  const ret = []; // 用于存放所有的promise实例
  const executing = []; // 用于存放目前正在执行的promise
  for (const item of array) {
    const p = Promise.resolve(iteratorFn(item)); // 防止回调函数返回的不是promise，使用Promise.resolve进行包裹
    ret.push(p);
    if (poolLimit <= array.length) {
      // then回调中，当这个promise状态变为fulfilled后，将其从正在执行的promise列表executing中删除
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e);
      if (executing.length >= poolLimit) {
        // 一旦正在执行的promise列表数量等于限制数，就使用Promise.race等待某一个promise状态发生变更，
        // 状态变更后，就会执行上面then的回调，将该promise从executing中删除，
        // 然后再进入到下一次for循环，生成新的promise进行补充
        await Promise.race(executing);
      }
    }
  }
  return Promise.all(ret);
}
```
原文 https://www.qetool.com/scripts/view/13313.html

---

#### 防抖和节流

##### 防抖(debounce):

就是指触发事件后在 n 秒内函数只能执行一次，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。
search 搜索联想，用户在不断输入值时，用防抖来节约请求资源。（喵喵电影项目中的搜索用到过）
window 触发 resize 的时候，不断的调整浏览器窗口大小会不断的触发这个事件，用防抖来让其只触发一次

防抖函数分为非立即执行版和立即执行版。
非立即执行版的意思是触发事件后函数不会立即执行，而是在 n 秒后执行，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。
立即执行版的意思是触发事件后函数会立即执行，然后 n 秒内不触发事件才能继续执行函数的效果。

```js
function debounce(fn, delay, immediate = true) {
    let timer = null
    return function (...args) {
        if (timer) clearTimeout(timer)
        immediate && !timer && fn.apply(this, args)
        timer = setTimeout(() => {
            timer = null
            !immediate && fn.apply(this, args)
        }, delay)
    }
}

function fn () {
  console.log('防抖')
}
addEventListener('scroll', debounce(fn, 1000))
```
##### 节流(throttle):

就是指连续触发事件但是在 n 秒中只执行一次函数。节流会稀释函数的执行频率。（手指移动的时候，handleTouchMove 执行的频率是很高的）
鼠标不断点击触发，mousedown/mousemove(单位时间内只触发一次)
监听滚动事件，比如是否滑到底部自动加载更多，用 throttle 来判断 所谓防抖，就是指触发事件后在 n 秒内函数只能执行一次，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。

```js
function throttle(fn, delay, immediate = true) {
    let timer = null
    return function (...args) {
        if (!timer) {
            timer = setTimeout(() => {
                timer = null
                !immediate && fn.apply(this, args)
            }, delay)
            immediate && fn.apply(this, args)
        }
    }
}

function fn () {
  console.log('节流')
}
addEventListener('scroll', throttle(fn, 1000))
```

---

#### apply() call() bind()

apply/call/bind 的区别

相同点：两个方法产生的作用是完全一样的，都用来改变当前函数调用的对象。

不同点：调用的参数不同，比较精辟的总结：

foo.call(this,arg1,arg2,arg3) == foo.apply(this, arguments)==this.foo(arg1, arg2, arg3)
bind() 传参和call一致。

call 、bind 、 apply 这三个函数的第一个参数都是 this 的指向对象，第二个参数差别就来了：

call 的参数是直接放进去的，第二第三第 n 个参数全都用逗号分隔，直接放到后面 obj.myFun.call(db,'成都', ... ,'string' )。

apply 的所有参数都必须放在一个数组里面传进去 obj.myFun.apply(db,['成都', ..., 'string' ])。

bind 除了返回是函数以外，它 的参数和 call 一样。

当然，三者的参数不限定是 string 类型，允许是各种类型，包括函数 、 object 等等！

call：
```js
// 将要改变this指向的方法挂到目标this上执行并返回
Function.prototype.mycall = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('not funciton')
  }
  context = context || window
  context.fn = this
  let arg = [...arguments].slice(1)
  let result = context.fn(...arg)
  delete context.fn
  return result
}
```

apply:
```js
Function.prototype.myapply = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('not funciton')
  }
  context = context || window
  context.fn = this
  let result
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```

bind:
```js
Function.prototype.mybind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  let _this = this
  let arg = [...arguments].slice(1)
  return function F() {
    // 处理函数使用new的情况
    if (this instanceof F) {
      return new _this(...arg, ...arguments)
    } else {
      return _this.apply(context, arg.concat(...arguments))
    }
  }
}
```

---

#### 原型和原型
##### 原型

在对实例的封装中，我们将所有实例共享的方法抽离出来，挂载在一个对象里，这个对象就是原型对象。

构造函数中，使用 prototype 指向原型对象。

实例对象中，使用 __proto__ 指向原型对象。

原型对象中，使用 constructor 指向构造函数。

##### 原型链

在上例中，Person 的原型对象也是一个对象。因此，原型对象也是别人的实例。

我们会发现，Person 的原型对象中，也有一个 __proto__ 属性，用于指向自己的原型对象。而该原型对象的构造函数， constructor 指向 Object。

![prototype1](/images/article/prototype1.png)

对于实例对象 p1 而言，它的原型对象为 p1.__proto__

对于 p1 的原型对象而言，它的原型对象为 p1.__proto__.__proto__

此时我们发现，一个单向的链表模型存在于对象的关联关系中，这就是原型链。

https://xiaozhuanlan.com/advance/0831942675

https://github.com/mqyqingfeng/Blog/issues/2


---

#### 深拷贝和浅拷贝

它们俩只针对像 object/Array 这样的复杂对象的

简单的说：
浅拷贝只复制一层对象属性，只是将数据中存放的引用拷贝下来，但依旧指向同一个存放地址
深拷贝则递归复制了所有层级，将数据中所有的数据都拷贝下来，而不仅仅是引用。拷贝下来的数据的修改，并不会影响原数据。
javascript 存储对象都是存地址的，所以浅拷贝会导致 obj1 和 obj2 指向同一块内存地址，改变了其中一方的内容。会导致拷贝对象和源对象都发生改变；
而深拷贝是开辟一块新的内存地址，将原对象的各个属性逐个复制进去，对拷贝对象和源对象各自的操作互不影响。

实现方式

```js
// 浅拷贝
// 1. ...实现
let copy1 = {...{x:1}}

// 2. Object.assign实现

let copy2 = Object.assign({}, {x:1})
```

```js
// 深拷贝
// 1. JOSN.stringify()/JSON.parse()，无法拷贝函数、循环引用
let obj = {a: 1, b: {x: 3}}
JSON.parse(JSON.stringify(obj))

// 2. 递归拷贝
function deepClone(obj) {
  let copy = obj instanceof Array ? [] : {}
  for (let i in obj) {
    if (obj.hasOwnProperty(i)) {
      copy[i] = typeof obj[i] === 'object' ? deepClone(obj[i]) : obj[i]
    }
  }
  return copy
}

function deepClone(obj, hash = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (hash.has(obj)) return hash.get(obj);

  const copy = Array.isArray(obj) ? [] : {};
  hash.set(obj, copy);

  for (const key in obj) {
    copy[key] = deepClone(obj[key], hash);
  }
  return copy;
}

```

---

#### 跨域

跨域需要针对浏览器的同源策略来理解，同源策略指的是请求必须是同一个端口，同一个协议，同一个域名，不同源的客户端脚本在没有明确授权的情况下，不能读写对方资源。
受浏览器同源策略的影响，不是同源的脚本不能操作其他源下面的对象。想要操作另一个源下的对象是就需要跨域。
当一个请求 url 的协议、域名、端口三者之间任意一个与当前页面 url 不同即为跨域

非同源限制

【1】无法读取非同源网页的 Cookie、LocalStorage 和 IndexedDB
【2】无法接触非同源网页的 DOM
【3】无法向非同源地址发送 AJAX 请求

跨域解决方法

1、 通过 jsonp 跨域（只支持 get，不支持 post ）：

  核心思想：网页通过添加一个```<script>```元素，向服务器请求 JSON 数据，服务器收到请求后，将数据放在一个指定名字的回调函数的参数位置传回来。

  原理：
  客户端利用 script 标签的 src 属性，去请求一个接口，因为 src 属性不受跨域影响。
  服务端响应一个字符串
  客户端接收到字符串，然后把它当做 JS 代码运行。
  
2、 document.domain + iframe 跨域
3、 location.hash + iframe
4、 window.name + iframe 跨域
5、 postMessage 跨域
6、 跨域资源共享（CORS）：

  CORS 是跨域资源分享（Cross-Origin Resource Sharing）的缩写。它是 W3C 标准，属于跨源 AJAX 请求的根本解决方法
  实际上就是在响应头添加允许跨域的源
  Access-Control-Allow-Origin: 字段和值(意思就是允许去哪些源地址去请求这个服务器)

7、 nginx 代理跨域
8、 nodejs 中间件代理跨域
9、 WebSocket 协议跨域
10、vue-cli代理转发

  在前端服务和后端接口服务之间 架设一个中间代理服务，它的地址保持和前端服务一致，那么：
  代理服务和前端服务之间由于协议域名端口三者统一不存在跨域问题，可以直接发送请求
  代理服务和后端服务之间由于并不经过浏览器没有同源策略的限制，可以直接发送请求
  这样，我们就可以通过中间这台服务器做接口转发，在开发环境下解决跨域问题，看起来好像挺复杂，其实vue-cli已经为我们内置了该技术，我们只需要按照要求配置一下即可。

  ```js
  devServer: {
    proxy: {
      // http://c.m.163.com/nc/article/headline/T1348647853363/0-40.html
      '/api': { // 请求相对路径以/api开头的, 才会走这里的配置
        target: 'http://c.m.163.com', // 后台接口域名 我们要代理的真实的接口地址
        changeOrigin: true, // 改变请求来源(欺骗后台你的请求是从http://c.m.163.com)
        pathRewrite: {
          '^/api': '' // 因为真实路径中并没有/api这段, 所以要去掉这段才能拼接正确地址转发请求
        }
      }
    }
  }
  ```

---

#### 冒泡
##### 1. 事件冒泡与事件捕获

DOM事件流（event flow）存在三个阶段：事件捕获阶段、处于目标阶段、事件冒泡阶段。

事件捕获（event capturing）：通俗的理解就是，当鼠标点击或者触发dom事件时，浏览器会从根节点开始由外到内进行事件传播，即点击了子元素，如果父元素通过事件捕获方式注册了对应的事件的话，会先触发父

元素绑定的事件。在事件捕获的概念下发生click事件的顺序应该是：document -> html -> body -> div -> p

事件冒泡（dubbed bubbling）：与事件捕获恰恰相反，事件冒泡顺序是由内到外进行事件传播，直到根节点。在事件冒泡的概念下发生click事件的顺序应该是：p -> div -> body -> html -> document

##### 2. addEventListener 的第三个参数

element.addEventListener(event, function, useCapture)
##### 3. 事件代理

在实际的开发当中，利用事件流的特性，我们可以使用一种叫做事件代理的方法。使用事件代理的好处不仅在于将多个事件处理函数减为一个，而且对于不同的元素可以有不同的处理方法。假如上述列表元素当中添加了其他的元素节点（如：a、span等），我们不必再一次循环给每一个元素绑定事件，直接修改事件代理的事件处理函数即可。
##### 阻止事件冒泡

    1. 给子级加 event.stopPropagation()
    ```
    $("#div1").mousedown(function(e){
        var e=event||window.event;
        event.stopPropagation();
    });
    ```
    2. 在事件处理函数中返回 false
    ```
    $("#div1").mousedown(function(event){
        var e=e||window.event;
        return false;
    });
    ```
但是这两种方式是有区别的。return false 不仅阻止了事件往上冒泡，而且阻止了事件本身(默认事件)。event.stopPropagation()则只阻止事件往上冒泡，不阻止事件本身。

    3. event.target==event.currentTarget，让触发事件的元素等于绑定事件的元素，也可以阻止事件冒泡；
   
##### 阻止默认事件

    (1）event.preventDefault()
    (2）return false
   
---

#### js 里的==和===区别

==  先检查两个数据类型，如果相同， 则进行===比较， 如果不同， 则进行一次类型转换， 转换成相同类型后再进行比较， 
===如果类型不同，直接 false.

双等号==： 
　　（1）如果两个值类型相同，再进行三个等号(===)的比较
　　（2）如果两个值类型不同，也有可能相等，需根据以下规则进行类型转换在比较：
　　　　 1）如果一个是 null，一个是 undefined，那么相等
　　　　 2）如果一个是字符串，一个是数值，把字符串转换成数值之后再进行比较

三等号===：

（1）如果类型不同，就一定不相等
　　（2）如果两个都是数值，并且是同一个值，那么相等；如果其中至少一个是 NaN，那么不相等。（判断一个值是否是 NaN，只能使用 isNaN( ) 来判断）
　　（3）如果两个都是字符串，每个位置的字符都一样，那么相等，否则不相等。
　　（4）如果两个值都是 true，或是 false，那么相等
　　（5）如果两个值都引用同一个对象或是函数，那么相等，否则不相等
　　（6）如果两个值都是 null，或是 undefined，那么相等

注意： null 除了和 undefined   和其它任何值比都是不等的，undefined 也是一样

---

#### js判断类型

typeof

instanceof

prototype.toString.call()

---

#### Ajax

1.创建 Ajax 核心对象 XMLHttpRequest

2.向服务器发送请求

3.服务器响应处理

---

#### 并行与并发

并行(parallel)：指在同一时刻，有多条指令在多个处理器上同时执行。所以无论从微观还是从宏观来看，二者都是一起执行的。
并发(concurrency)：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。

如果某个系统支持两个或者多个动作（Action）同时存在，那么这个系统就是一个并发系统。如果某个系统支持两个或者多个动作同时执行，那么这个系统就是一个并行系统。并发系统与并行系统这两个定义之间的关键差异在于“存在”这个词。

---

#### 页面渲染html的过程

    1.解析HTML文件，创建DOM树

    浏览器解析html源码，然后创建一个 DOM树。并行请求 css/image/js在DOM树中，每一个HTML标签都有一个对应的节点，并且每一个文本也都会有一个对应的文本节点。DOM树的根节点就是 documentElement，对应的是html标签。

    2.解析CSS,形成CSS对象模型

    浏览器解析CSS代码，计算出最终的样式数据。构建CSSOM树。对CSS代码中非法的语法它会直接忽略掉。解析CSS的时候会按照如下顺序来定义优先级：
    浏览器默认设置 < 用户设置 < 外链样式 < 内联样式 < html中的style。

    3.将CSS与DOM合并，构建渲染树（renderingtree）

    DOM Tree + CSSOM –> 渲染树（rendering tree）。渲染树和DOM树有点像，但是是有区别的。DOM树完全和html标签一一对应，但是渲染树会忽略掉不需要渲染的元素，比如head、display:none的元素等。而且一大段文本中的每一个行在渲染树中都是独立的一个节点。渲染树中的每一个节点都存储有对应的css属性。

    4.布局和绘制

    一旦渲染树创建好了，浏览器就可以根据渲染树直接把页面绘制到屏幕上。
    以上四个步骤并不是一次性顺序完成的。如果DOM或者CSSOM被修改，以上过程会被重复执行。实际上，CSS和JavaScript往往会多次修改DOM或者CSSOM。

    Repaint(重绘)

    重绘是改变不影响元素在网页中的位置的元素样式时，譬如background-color(背景色)， border-color(边框色)， visibility(可见性)，浏览器会根据元素的新属性重新绘制一次(这就是重绘，或者说重新构造样式)，使元素呈现新的外观。重绘不会带来重新布局，所以并不一定伴随重排。

    Reflow（重排）

    渲染对象在创建完成并添加到渲染树时，并不包含位置和大小信息。计算这些值的过程称为布局或重排。

    "重绘"不一定需要"重排"，比如改变某个网页元素的颜色，就只会触发"重绘"，不会触发"重排"，因为布局没有改变。
    但是，"重排"必然导致"重绘"，比如改变一个网页元素的位置，就会同时触发"重排"和"重绘"，因为布局改变了。

##### 浏览器如何优化渲染？

    （1）将多次改变样式属性的操作合并成一次操作

    （2）将需要多次重排的元素，position属性设为absolute或fixed，

    这样此元素就脱离了文档流，它的变化不会影响到其他元素。例如有动画效果的元素就最好设置为绝对定位。

    （3）由于display属性为none的元素不在渲染树中，对隐藏的元素操作不会引发其他元素的重排。

    如果要对一个元素进行复杂的操作时，可以先隐藏它，操作完成后再显示。这样只在隐藏和显示时触发2次重排。

---
#### v8 引擎--垃圾回收机制

https://xiaozhuanlan.com/advance/4158792360

https://juejin.cn/post/6844904016325902344

---

#### 前端缓存
##### http 缓存

缓存过程：

浏览器第一次加载资源，服务器返回200，浏览器将资源文件从服务器上请求下载下来，并把response header及该请求的返回时间(要与Cache-Control和Expires对比)一并缓存；

下一次加载资源时，先比较当前时间和上一次返回200时的时间差，如果没有超过Cache-Control设置的max-age，则没有过期，命中强缓存，不发请求直接从本地缓存读取该文件（如果浏览器不支持HTTP1.1，则用Expires判断是否过期）；

如果时间过期，服务器则查看header里的If-None-Match和If-Modified-Since ；

服务器优先根据Etag的值判断被请求的文件有没有做修改，Etag值一致则没有修改，命中协商缓存，返回304；如果不一致则有改动，直接返回新的资源文件带上新的Etag值并返回 200；

如果服务器收到的请求没有Etag值，则将If-Modified-Since和被请求文件的最后修改时间做比对，一致则命中协商缓存，返回304；不一致则返回新的last-modified和文件并返回 200；

使用协商缓存主要是为了进一步降低数据传输量，如果数据没有变，就不必要再传一遍

强缓存

    强缓存主要是采用响应头中的Cache-Control和Expires两个字段进行控制的

    Cache-Control 值理解:
    max-age 指定设置缓存最大的有效时间(单位为s)
    public 指定响应会被缓存，并且在多用户间共享
    private 响应只作为私有的缓存，不能在用户间共享
    no-cache 指定不缓存响应，表明资源不进行缓存
    no-store 绝对禁止缓存

    Expires 理解:
    缓存过期时间，用来指定资源到期的时间，是服务器端的具体的时间点, 需要和Last-modified结合使用

    Last-modified 理解
    服务器端文件的最后修改时间，需要和cache-control共同使用，是检查服务器端资源是否更新的一种方式

    ETag 理解
    根据实体内容生成一段hash字符串，标识资源的状态，由服务端产生。浏览器会将这串字符串传回服务器，验证资源是否已经修改

协商缓存(304)

    协商缓存是指当强缓存机制如果检测到缓存失效，就需要进行服务器再验证

##### 浏览器缓存： cookies，sessionStorage 和 localStorage

    Cookie

    LocalStorage

    SessionStorage

    Service Worker:
    主要是为了提高web app的用户体验，可以实现离线应用消息推送等等一系列的功能, 可以看做是一个独立于浏览器的Javascript代理脚本, 在离线状态下也能提供基本的功能。 出于安全性的考虑，Service Worker 只能在https协议下使用

    相同点：都存储在客户端

    不同点： 1.存储大小
    cookie 数据大小不能超过 4k。
    sessionStorage 和 localStorage 虽然也有存储大小的限制，但比 cookie 大得多，可以达到 5M 或更大。

    2.有效时间
    localStorage    存储持久数据，浏览器关闭后数据不丢失除非主动删除数据；
    sessionStorage  数据在当前浏览器窗口关闭后自动删除。
    cookie          设置的 cookie 过期时间之前一直有效，即使窗口或浏览器关闭

    3.数据与服务器之间的交互方式
    cookie 的数据会自动的传递到服务器，服务器端也可以写 cookie 到客户端
    sessionStorage 和 localStorage 不会自动把数据发给服务器，仅在本地保存。

设置 cookie

```js
  var name = "jack";
  var pwd = "123";
  var now = new Date();
  now.setTime(now.getTime() +1 * 24 * 60 * 60 * 1000);//转毫秒
  var path = "/";//可以是具体的网页
  document.cookie = "name=" + name + ";expires=" + now.toUTCString() + ";path=" + path;//姓名
  document.cookie= "pwd=" + pwd + ";expires=" + now.toUTCString()+ ";path=" + path; //密码
```

读取 cookie

```
  var data=document.cookie;//获取对应页面的cookie
```

解析 cookie

    1.截取字符串

    ```js
      function getKey(key) {
        var data = document.cookie;
        var findStr = key + "=";
        //找到key的位置
        var index = data.indexOf(findStr);
        if (index == -1)
            return null;
        var subStr = data.substring(index +findStr.length);
        var lastIndex = subStr.indexOf(";");
        if (lastIndex == -1) {
          return subStr;
        } else {
          return subStr.substring(0,lastIndex);
        }
      }
    ```

    2.使用正则表达式+JSON

    ```js
      function getKey(key) {
        return JSON.parse("{\"" +document.cookie.replace(/;\s+/gim, "\",\"").replace(/=/gim, "\":\"") + "\"}")[key];
      }
    ```

清除 cookie

```js
  var name = null;
  var pwd = null;
  var now = new Date();
  var path = "/";//可以是具体的网页
  document.cookie= "name=" + name + ";expires=" + now.toUTCString()+ ";path=" + path;//姓名
  document.cookie = "pwd=" + pwd + ";expires=" + now.toUTCString()+ ";path=" + path; //密码
```

---

#### 阻塞渲染

https://juejin.cn/post/6844903497599549453


---

#### defer和async

https://segmentfault.com/q/1010000000640869/a-1020000000641029


---
#### new

•创建一个新的对象；

•将构造函数的this指向这个新对象(将其隐式原型指向构造函数原型)；

•指向构造函数的代码，为这个对象添加属性，方法等；

•返回新对象。

实现：

```js
function myNew (fun) {
  return function () {
    // 创建一个新对象且将其隐式原型指向构造函数原型
    let obj = {
      __proto__ : fun.prototype
    }
    // 执行构造函数
    fun.call(obj, ...arguments)
    // 返回该对象
    return obj
  }
}

function person(name, age) {
  this.name = name
  this.age = age
}
let obj = myNew(person)('chen', 18) // {name: "chen", age: 18}
```

---

#### history与hash路由的区别

https://blog.51cto.com/u_13953650/4752840


#### import 和 require的区别

区别1：模块加载的时间

require：运行时加载
import：编译时加载（效率更高）【由于是编译时加载，所以import命令会提升到整个模块的头部】

区别2：模块的本质

require：模块就是对象，输入时必须查找对象属性
import：ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，再通过 import 命令输入（这也导致了没法引用 ES6 模块本身，因为它不是对象）。由于 ES6 模块是编译时加载，使得静态分析成为可能。有了它，就能进一步拓宽 JavaScript 的语法，比如引入宏（macro）和类型检验（type system）这些只能靠静态分析实现的功能。

```js
// CommonJS模块
let { exists, readFile } = require('fs');
// 等同于
let fs = require('fs');
let exists = fs.exists;
let readfile = fs.readfile;

```
上面CommonJs模块中，实质上整体加载了fs对象（fs模块），然后再从fs对象上读取方法

```js
// ES6模块
import { exists, readFile } from 'fs';
```
上面ES6模块，实质上从fs模块加载2个对应的方法，其他方法不加载

区别3：严格模式
CommonJs模块和ES6模块的区别：
（1）CommonJs模块默认采用非严格模式
（2）ES6 的模块自动采用严格模式，不管你有没有在模块头部加上 “use strict”;
（3）CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用，举例如下

```js
// m1.js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
// m2.js
import {foo} from './m1.js';
console.log(foo); //bar
setTimeout(() => console.log(foo), 500); //baz
```

ES6 模块之中，顶层的 this 指向 undefined ，即不应该在顶层代码使用 this

---

#### Object.defineProperty()

https://www.jianshu.com/p/8fe1382ba135

---

#### i++与++i的区别

https://www.jianshu.com/p/3d3481e3d99e

---

#### 继承

https://github.com/mqyqingfeng/Blog/issues/16

---

#### 前端优化策略

https://blog.csdn.net/weixin_44485276/article/details/119975366