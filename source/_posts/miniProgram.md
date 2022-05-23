---
title: 微信小程序面试知识点综合
featured_image: ./images/falcon.jpeg
---

#### 在小程序中如何获取用户信息？

    （1）小程序为升级前：可使用wx.getUserInfo直接获取用户信息，目前逐渐不能使用该方法了

    （2）升级后，可使用以下方法获取用户的账户信息：

    （1）使用button，设置其属性：open-type="getUserInfo"

    ```<button open-type="getUserInfo">获取用户信息</button>```

---

#### 小程序中如何实现分享功能，微信有什么限制？

    发送给朋友：onShareAppMessage(Object object)；

    分享到朋友圈： onShareTimeline()；

    限制：

    “单页模式”下，一些组件或接口存在一定限制：
    1、页面无登录态，与登录相关的接口，入wx.login()均不可用；
    2、不允许跳转其他页面，包括跳转小程序页面，跳转其他小程序，跳转微信原生页面；
    3、不允许横屏，页面包含的tabbar不会渲染，包括自定的tabbar;
    4、 本地储存与小程序普通模式不共用；

---

#### 你的小程序是如何上线的，审核大概需要多久？

    （1）在微信web开发者工具里找到项目，并且设置好服务器的域名，如果你的小程序没有用到外网请求，可以不用配置服务器。配置好服务器，先预览一下，看看有没有问题，如果没有问题的话，点击上传。
    （2）上传代码之后，在微信公众号平台登录微信小程序后台，点击开发管理，就可以看到刚刚上传的代码，点击提交审核，就可以了。接下来就是等待微信官方的审核。
    （3）一般都是1~3天左右

---

#### 小程序中如何用像Vant一样的第三方组件？

    （1）打开cmd，进入你的项目中，在cmd中执行：npm init，初始化项目
    （2)然后安装Vant
    （3）打开小程序客户端，选择【工具】菜单 -> 选择【构建 npm】命令

---

#### 小程序如何实现父子组件传参?

1、父组件给子组件传值

在子组件中定义属性
```properties: {    // 复杂定义    name:{      type: String,      value:'张三丰'    },    // 简单定义    name2:String},```

父组件在引用子组件的时候通过设置属性的方式给子组件传值

2、 子组件给父组件传值

在组件中绑定一个自定义事件

```// 引用了自定义的组件， 绑定了myevent事件，这个事件对应的是parentEvent方法<test-button name="张无忌" bindmyevent="parentEvent"></test-button>```

在子组件中触发这个事件，可以给父组件传值。
子组件中通过triggerEvent触发事件
 ```
 methods: {    
    方法名字: function(){   
    var myEventDetail = {} // detail对象，提供给事件监听函数
    var myEventOption = {} // 触发事件的选项
    this.triggerEvent('myevent', myEventDetail, myEventOption)
    }
}
```

---

#### 小程序中APP的生命周期有哪些？

onLaunch(options)
小程序被加载完毕的时候调用。这个方法一般用来做一些初始化的事情。比如获取用户 信息、获取历史缓存信息、获取小程序打开来源等。

onShow(options)
小程序启动，或从后台进入前台显示时调用。如果想要在小程序每次进入到前台的时候 都执行一些事情，那么可以把代码放在这个里面。比如一些实时动态更改的数据，用户每次进来都要从服务器更新，那么我们就可以在这个里面做。

onHide()
小程序被切换到后台（包括微信自身被切换到后台或者小程序暂时被切换到后台时)。可以在这个方法中做一些数据的保存。

onError(String error)
小程序发生脚本错误，或者 api 调用失败时触发。在小程序发生错误的时候，会把错误 信息发送到这个函数中，所以可以在这个函数中做一些错误收集。

onPageNotFound(Object)
小程序要打开的页面不存在时触发。一般在代码更新的时候，有些页面被删除了，但是 其他地方没有改过来的情况下会发生这种情况，或者一些活动页面，活动结束后被关掉了。也可以 在这个里面做一些错误的收集和页面的重新跳转。

getApp()
获取当前的 app 对象。一般在app.js外的地方调用。在app.js内部可以使用this获得当前的大对象；在外面要用定义在app.js的全局数据时，要用getApp()。

---

#### 小程序中Page的生命周期有哪些？

• onLoad() 页面加载时触发，只会调用一次，可获取当前页面路径中的参数。
• onShow() 页面显示/切入前台时触发，一般用来发送数据请求；
• onReady() 页面初次渲染完成时触发, 只会调用一次，代表页面已可和视图层进行交互。
• onHide() 页面隐藏/切入后台时触发, 如底部 tab 切换到其他页面或小程序切入后台等。
• onUnload() 页面卸载时触发，如redirectTo或navigateBack到其他页面时。

---

#### 小程序如何定义事件？

在小程序中绑定事件可以以bind开头然后跟上事件的类型，如bindtap绑定一个点击事件，对应的值是一个字符串，需要在page构造器中定义同名函数，每次触发事件之后就会执行对应函数的内容。

---

#### 如何阻止小程序的事件冒泡？

在小程序中除了通过bind之外，还可以通过catch进行事件绑定，通过catch绑定的事件不会触发事件冒泡。

---

#### 如何让事件在捕获阶段触发？

事件的触发分为两个阶段，首先是捕获阶段，其次是冒泡阶段。默认情况下事件都是在冒泡阶段触发。如果希望事件可以在捕获阶段触发，可以通过capture-bind进行事件绑定。

---

#### 传递数据

使用全局遍历实现数据传递
页面跳转或重定向时，使用url带参数传递数据
使用组件模板 template传递参数
使用缓存传递参数
使用数据库传递参数
或
给html元素添加data-*属性来传递值，然后通过e.currentTarget.dataset或onload的param参数获取（data- 名称不能有大写字母，不可以存放对象）
设置id 的方法标识来传值，通过e.currentTarget.id获取设置的id值，然后通过设置全局对象的方式来传递数据
在navigator中添加参数数值

---

#### wx.navigateTo和 wx.redirectTo区别

使用wx.navigateTo每新开一个页面，页面栈大小加1，使用wx.navigateTo重复打页面也会增加页面栈
使用wx.redirectTo会关闭当前页面打开新页面，页面栈大小不变
1.对于可逆操作，使用wx.navigateTo,比如从首页跳转到二级页面，从二级页面返回是不需要重新渲染首页
2.对于不可逆操作，使用wx.redirectTo,比如用户登录成功后，关闭登录页面，不能返回到登录界面。
3.不要在首页使用wx.redirectTo，这样会导致应用无法返回首页

---

#### 原理相关

https://blog.csdn.net/weixin_53150999/article/details/122486262?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-4.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-4.pc_relevant_default&utm_relevant_index=7

---