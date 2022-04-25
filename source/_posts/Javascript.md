---
title: js用法相关
featured_image: ./images/blackwidow.jpg
---


#### async/await

async 作为一个关键字放到函数前面，它的调用会返回一个 promise  对象。如果 async  函数中有返回值 ,当调用该函数时，内部会调用 Promise.solve()  方法把它转化成一个 promise  对象作为返回, 但如果 timeout  函数内部抛出错误，就会调用 Promise.reject()  返回一个 promise 对象，要想获取到 async  函数的执行结果，就要调用 promise 的 then  或 catch  来给它注册回调函数，如果只是 async,  和 promise  差不多，但有了 await 就不一样了， await  关键字只能放到 async  函数里面，await 是等待的意思，那么它等待什么呢，它后面跟着什么呢？其实它后面可以放任何表达式，不过我们更多的是放一个返回 promise  对象的表达式，它等待的是 promise  对象的执行完毕，并返回结果。

```
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

#### 防抖和节流

防抖(debounce):

就是指触发事件后在 n 秒内函数只能执行一次，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。
search 搜索联想，用户在不断输入值时，用防抖来节约请求资源。（喵喵电影项目中的搜索用到过）
window 触发 resize 的时候，不断的调整浏览器窗口大小会不断的触发这个事件，用防抖来让其只触发一次

节流(throttle):

就是指连续触发事件但是在 n 秒中只执行一次函数。节流会稀释函数的执行频率。（手指移动的时候，handleTouchMove 执行的频率是很高的）
鼠标不断点击触发，mousedown/mousemove(单位时间内只触发一次)
监听滚动事件，比如是否滑到底部自动加载更多，用 throttle 来判断 所谓防抖，就是指触发事件后在 n 秒内函数只能执行一次，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。

防抖函数分为非立即执行版和立即执行版。
非立即执行版的意思是触发事件后函数不会立即执行，而是在 n 秒后执行，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。
立即执行版的意思是触发事件后函数会立即执行，然后 n 秒内不触发事件才能继续执行函数的效果。

---

#### apply() call() bind()

apply/call 的区别

相同点：两个方法产生的作用是完全一样的，都用来改变当前函数调用的对象。

不同点：调用的参数不同，比较精辟的总结：

foo.call(this,arg1,arg2,arg3) == foo.apply(this, arguments)==this.foo(arg1, arg2, arg3)
bind()方法在这里再多说一下，bind 的时候传的参数会预先传给返回的方法，调用方法时就不用再传参数了。

---

#### 原型和原型链

https://github.com/mqyqingfeng/Blog/issues/2

每个对象都会在其内部初始化一个属性，就是 prototype(原型)，当我们访问一个对象的属性时，如果这个对象内部不存在这个属性，那么他就会去 prototype 里找这个属性，这个 prototype 又会有自己的 prototype，于是就这样一直找下去，也就是我们平时所说的原型链的概念。

关系：instance.constructor.prototype = instance.proto

特点：JavaScript 对象是通过引用来传递的，我们创建的每个新对象实体中并没有一份属于自己的原型副本，当我们修改原型时，与之相关的对象也会继承这一改变。当我们需要一个属性时，JavaScript 引擎会先看当前对象中是否有这个属性，如果没有的话，就会查找它的 prototype 对象是否有这个属性，如此递推下去，一致检索到 Object 内建对象。

![proto1](/images/protopic1.png)
![proto2](/images/protopic2.jpg)
![proto3](/images/protopic3.jpg)

---

#### 深拷贝和浅拷贝

它们俩只针对像 object/Array 这样的复杂对象的

简单的说：
浅拷贝只复制一层对象属性，只是将数据中存放的引用拷贝下来，但依旧指向同一个存放地址
深拷贝则递归复制了所有层级，将数据中所有的数据都拷贝下来，而不仅仅是引用。拷贝下来的数据的修改，并不会影响原数据。
javascript 存储对象都是存地址的，所以浅拷贝会导致 obj1 和 obj2 指向同一块内存地址，改变了其中一方的内容。会导致拷贝对象和源对象都发生改变；
而深拷贝是开辟一块新的内存地址，将原对象的各个属性逐个复制进去，对拷贝对象和源对象各自的操作互不影响。

实现方式

![deepcopy1](/images/deepcopy1.png)
![deepcopy2](/images/deepcopy2.png)

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

1、 通过 jsonp 跨域（只支持 get，不支持 post ）
2、 document.domain + iframe 跨域
3、 location.hash + iframe
4、 window.name + iframe 跨域
5、 postMessage 跨域
6、 跨域资源共享（CORS） 用得多
7、 nginx 代理跨域
8、 nodejs 中间件代理跨域
9、 WebSocket 协议跨域

---

#### 冒泡

冒泡型事件：事件按照从最特定的事件目标到最不特定的事件目标(document 对象)的顺序触发。
w3c 的方法是 e.stopPropagation()，IE 则是使用 e.cancelBubble = true。

![pop1](/images/pop1.png)

阻止默认事件

![pop2](/images/pop2.png)

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

#### Ajax

1.创建 Ajax 核心对象 XMLHttpRequest

![ajax1](/images/ajax1.png)

2.向服务器发送请求

![ajax2](/images/ajax2.png)
![ajax3](/images/ajax3.png)

3.服务器响应处理

![ajax4](/images/ajax4.png)
![ajax5](/images/ajax5.png)

---


#### 如何理解 this

this 的指向
this 表示当前对象，this 的指向是根据调用的上下文来决定的，默认指向 window 对象，指向 window 对象时可以省略不写，例如：

```
  this.alert() <=> window.alert()<=> alert();
```

调用的上下文环境包括全局和局部

总结：

全局环境始终指向 window

局部环境：

    1.在全局作用域下直接调用函数，this指向window
    2.对象函数调用，哪个对象调用就指向哪个对象
    3.使用new实例化对象，在构造函数中的this指向实例化对象
    4.使用call或play改变this的指向

作用：用于区分全局变量和局部变量

---

#### 并行与并发

并行(parallel)：指在同一时刻，有多条指令在多个处理器上同时执行。所以无论从微观还是从宏观来看，二者都是一起执行的。
并发(concurrency)：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。

如果某个系统支持两个或者多个动作（Action）同时存在，那么这个系统就是一个并发系统。如果某个系统支持两个或者多个动作同时执行，那么这个系统就是一个并行系统。并发系统与并行系统这两个定义之间的关键差异在于“存在”这个词。

---

#### 浏览器输入一个 url 发生的事情

    1.根据域名到DNS中找到IP
    2.根据IP建立TCP连接(三次握手)
    3.连接建立成功发起http请求
    4.服务器响应http请求
    5.浏览器解析HTML代码并请求html中的静态资源（js,css）
    6.关闭TCP连接（四次挥手）
    7.浏览器渲染页面

---

#### v8 引擎--垃圾回收机制

v8 内存限制
node 在 32 位机器上只有 0.7GB 内存，64 位上有 1.4GB。为何限制内存的大小呢？是因为 v8 有垃圾回收机制。怎么才能突破限制，通过 JVM 来修改内存的大小。可以修改新生代和老生代的大小。

如何判断回收内容
如果一个对象不满足下面条件的，将会被回收
 （1）非根对象被其他对象引用
 （2）浏览器中对象被 window 引用
 （3）node 中对象被 global 对象引用

v8 的回收策略
内存被分为新生代和老生代。新生代存储一般是活跃对象，浏览器对新生代的回收也是比较频繁。新生代中对象在满足某些条件后，会晋升为老生代。
两个生代的大小和回收算法都是不同的。

新生代的回收策略
新生代的回收策略是通过 scavenge 中的 Cheney 算法实现的，这个算法主要是通过复制实现的。新生代内存主要分为两个区，From 区（被使用的）和 To 区（闲置的），这个两个区主要就是进行来回复制。新生代的回收主要的实现步骤如下：
 （1）对象首先被分到 From 区
 （2）检查对象是否是活跃对象，有的话直接复制到 To 区
 （3）To 区有两个指针：scanPtr 和 allocationPtr  scanPtr 主要是指向即将扫描的活跃对象   allocationPtr 是为即将成为新对象分配内存

新生代对象的晋升
 （1）从 From 区到 To 区，检查对象的内存地址是否经历过新生代的清理。
 （2）对象从 From 区到 To 区，发现 To 区空间已经被使用超过 25%，则对象将直接被分配到老生代

老生代的回收
老生代的回收策略主要是标记清除和标记整理。
标记清除：主要是两个步骤，标记和清除。首先遍历老生代的对象，判断对象是否被其他对象引用，有的话标记，没有的话进入清除阶段，把所有未被标记的对象清楚。
标记整理：由于在标记清除阶段很多清除了之后，对象的内存不是连续的，导致很多很大对象来了之后没有内存可以用。这时标记整理就发挥作用了。

---

#### cookies，sessionStorage 和 localStorage

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

```
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

    ```
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

    ```
      function getKey(key) {
        return JSON.parse("{\"" +document.cookie.replace(/;\s+/gim, "\",\"").replace(/=/gim, "\":\"") + "\"}")[key];
      }
    ```

清除 cookie

```
  var name = null;
  var pwd = null;
  var now = new Date();
  var path = "/";//可以是具体的网页
  document.cookie= "name=" + name + ";expires=" + now.toUTCString()+ ";path=" + path;//姓名
  document.cookie = "pwd=" + pwd + ";expires=" + now.toUTCString()+ ";path=" + path; //密码
```

---

#### 阻塞渲染

默认情况下，CSS 被视为阻塞渲染的资源。
我们可以通过媒体类型和媒体查询将一些 CSS 资源标记为不阻塞渲染。
浏览器会下载所有 CSS 资源，无论阻塞还是不阻塞。
CSS 是阻塞渲染的资源。需要将它尽早、尽快地下载到客户端，以便缩短首次渲染的时间。
