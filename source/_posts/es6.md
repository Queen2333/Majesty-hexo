---
title: es6 新特性
featured_image: ./images/blackpanther.jpg
---

#### 1.变量的新定义——let 和 const

##### var 的缺点

      可重复定义
      可随意修改
      没有块级作用域

##### let 和 const 特性

      在同一个块级作用域中，不允许重复定义
      const 定义的变量不允许二次修改（const定义的对象，里面的属性可修改）
      let 和 const 定义的变量会形成块级作用域
      它们定义的变量不存在变量提升，以及存在暂时性死区

##### 变量提升

    ```
      // var 的情况
      console.log(foo); // 输出undefined
      var foo = 2;
      // let 的情况
      console.log(bar); // 报错ReferenceError
      let bar = 2;
    ```

    （1）函数声明的提升优先于变量声明的提升；
    （2）重复的var声明会被忽略掉，但是重复的function声明会覆盖掉前面的声明。 在预处理阶段，
    声明的变量的初始值是undefined, 采用function声明的函数的初始内容就是函数体的内容。

##### 普通变量提升

    ```
      function test () {
        console.log(a);  //undefined
        var a = 123;
      };
      test();
       
      a = 1;
      var a;
      console.log(a); // 1
       
      console.log(v1);
      var v1 = 100;
      function foo() {
        console.log(v1);
        var v1 = 200;
        console.log(v1);
      }
      foo();
      console.log(v1);
      ​
      //undefined
      //undefined
      //200
      //100
    ```

##### 函数提升

    Javascript中不仅仅是变量声明有提升的现象，函数的声明也是一样；具名函数的声明有两种方式：
    函数声明式
    函数字面量式（函数表达式）

    ```
      //函数声明式
      function bar () {}
      //函数字面量式
      var foo = function () {}
    ```

    函数字面量式的声明和普通变量提升的结果是一样的，函数只是一个具体的值；
    但是函数声明式的提升现象和变量提升略有不同，函数提升是整个代码块提升到它所在的作用域的最开始执行。

    ```
      foo(); //1

      var foo;

      function foo () {
          console.log(1);
      }

      foo = function () {
          console.log(2);
      }
    ```

    这就是函数优先规则。
    下面这段代码，在低版本的浏览器中，函数提升不会被条件判断所控制，输出2；但是在高版本的浏览器中会报错，
    所以应该尽可能避免在块内部声明函数

    ```
      foo(); //低版本：2  //高版本： Uncaught TypeError: foo is not a function

      var a = true;

      if(a){
          function foo () { console.log(1); }
      }else{
          function foo () { console.log(2); }
      }
    ```

##### 小结

    ```
      var str; //这个属于变量声明
      str = "hhh"; //这个属于变量定义
      ​
      // 另
      var str2="fff";
      // 这样的其实是两个过程，可以看成
      var str2;
      str2="fff";
      // 而变量声明是会被提升的
    ```

    我们习惯将var a = 2;看做是一个声明，而实际上javascript引擎并不这么认为。
    它将var a和a = 2看做是两个单独的声明，第一个是编译阶段的任务，而第二个则是
    执行阶段的任务。这意味着无论作用域中的声明出现在什么地方，都将在代码本身被执
    行前首先进行处理，可以将这个过程形象地想象成所有的声明（变量和函数）都会被“移
    动”到各自作用域的最顶端，这个过程被称为提升。

---

#### 2.函数的变化——箭头函数，剩余参数，参数默认值

##### 箭头函数

    变量如果只有一个的时候，可以省略()
    如果是只有一句返回语句时，可以直接省略{return }这一部分
    因为它本身叫做 arrow，所以每次都必须带上=>符号

    缺点:

    1.箭头函数不能作为构造函数
    2.箭头函数没有它自己的 this 值，箭头函数内的 this 值继承自外围作用域
    3.箭头函数没有 arguments(注:arguments 对象是所有(非箭头)函数中都可用的局部变量。
    你可以使用 arguments 对象在函数中引用函数的参数。此对象包含传递给函数的每个 参数，
    第一个参数在索引 0 处，arguments 对象不是一个 Array 。它类似于 Array，但除了
    length 属性和索引元素之外没有任何 Array 属性)

##### 剩余参数

    ```
      const restParam = function(a, ...args){
          console.log(args);
      };
      restParam(1, 2, 3, 4);   //[2, 3, 4]
    ```

##### 默认参数

    ```
      function defaultParam(time = 1000){
        setTimeout(() => {
          //...
        }, time);
      }
    ```

---

#### 3. 数组——解构赋值、二进制数组

    解构
    注:
    1. 必须保证有赋值的过程
    2. 左边内容部分的结构必须与右边保持一致

---

#### 4. 字符串——模版字符串、startsWith、endsWidth

    模板字符串: 由倒引号包裹``，然后使用${}来包裹变量。
    startsWith: 返回值为 boolean 型，然后去匹配字符串开头的部分。 
    endsWith: 返回值是 boolean 类型，然后去匹配字符串的结尾。

---

#### 5. Iterator 和 for...of

---

#### 6. Generator 和 Promise

##### promise

    promise是一个对象，对象和函数的区别就是对象可以保存状态，函数不可以（闭包除外）
    并未剥夺函数return的能力，因此无需层层传递callback，进行回调获取数据
    代码风格，容易理解，便于维护
    多个异步等待合并便于解决
    主要用于异步计算
    可以将异步操作队列化，按照期望的顺序执行，返回符合预期的结果
    可以在对象之间传递和操作promise，帮助我们处理队列

    promise是用来解决两个问题的：

    回调地狱，代码难以维护，常常第一个的函数的输出是第二个函数的输入这种现象
    promise可以支持多个并发的请求，获取并发请求中的数据
    这个promise可以解决异步的问题，本身不能说promise是异步的

    详解：

    resolve作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），
    在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；
    reject作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），
    在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

    promise有三个状态：

    1、pending[待定]初始状态
    2、fulfilled[实现]操作成功
    3、rejected[被否决]操作失败

    当promise状态发生改变，就会触发then()里的响应函数处理后续步骤；

    promise状态一经改变，不会再变。
    Promise对象的状态改变，只有两种可能：
    从pending变为fulfilled
    从pending变为rejected。
    这两种情况只要发生，状态就凝固了，不会再变了。

    错误处理

    Promise会自动捕获内部异常，并交给rejected响应函数处理。

    第一种：reject('错误信息').then(() => {}, () => {错误处理逻辑})
    第二种：throw new Error('错误信息').catch( () => {错误处理逻辑})
    推荐使用第二种方式，更加清晰好读，并且可以捕获前面所有的错误（可以捕获N个then回调错误）

##### Generator

    Generator(生成器):是一类特殊的函数，跟普通函数声明时的区别是加了一个*号，
    以下两种方式都可以得到一个生成器函数

    Iterator(迭代器):当我们实例化一个生成器函数之后，这个实例就是一个迭代器。
    可以通过next()方法去启动生成器以及控制生成器的是否往下执行。

    yield/next:这是控制代码执行顺序的一对好基友。
    通过yield语句可以在生成器函数内部暂停代码的执行使其挂起，此时生成器函数仍然是运行并且是活跃的，
    其内部资源都会保留下来，只不过是处在暂停状态。
    在迭代器上调用next()方法可以使代码从暂停的位置开始继续往下执行。

    ```
      function getCallSettings() {
        utils.ajax({
          url: '/dialer/dialerSetting',
          method: "GET",
          success: (res) => {
            it.next(res.dialerSetting); // 将res.dialerSetting传给yield表达式
          },
          error: (err) => {
            it.throw(err); // 抛出错误
          }
        });
      }
      function *dealData() {
        try{
          let settingInfo = yield getCallSettings();
          // do something……
        }
        catch(err) {
          console.log(err); // 接收错误
        }
      }
      let it = dealData();
      it.next(); // 启动生成器
    ```

    Generator+Promise实现完美异步(现用async+await代替更简便)

    ```
      function getCallSettings() {
        // utils.ajax方法支持返回promise对象，把得到的promise return出去
        return utils.ajax({
          url: '/dialer/dialerSetting',
          method: "GET",
        });
      }
      function *dealData() {
        try {
          let settingInfo = yield getCallSettings();
          // do something……
        }
        catch(err) {
          console.log(err); // 接收错误
        }
      }
      ​
      let it = dealData();
      let promise = it.next().value; // 注意，这里拿到yield出来的promise
      promise.then(
        (info) => {
          it.next(info); // 拿到info传给yield表达式
        },
        (err) => {
          it.throw(err); // 抛出错误
        }
      );
    ```

---

#### 7. Class 和 extends

    es6新增的Class其实也是语法糖，js底层其实没有class的概念的，其实也是原型继承的封装。

---

#### 8.map()

    map() 方法返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。
    map() 方法按照原始数组元素顺序依次处理元素。
    注意： map() 不会对空数组进行检测。
    注意： map() 不会改变原始数组。

---
