---
title: ajax相关知识点综合
featured_image: ./images/spiderman1.jpeg
---

#### ajax，axios，和fetch

用法实现原理
https://zhuanlan.zhihu.com/p/364106739

优缺点
https://baijiahao.baidu.com/s?id=1709840036410376001&wfr=spider&for=pc

---

#### 如何取消ajax/axios

https://juejin.cn/post/6989262219007492110

---

#### axios拦截器

拦截器顾名思义就是对请求的拦截，分别为请求拦截器和响应拦截器， 执行顺序: 请求拦截器 -> api请求 -> 响应拦截器。
这其实就是一个顺序链表能实现的，把请求拦截器的函数推在前面， api请求的核心方法放在中间， 响应拦截器放在数组后面，遍历执行链表就实现了拦截器的顺序执行过程。

拦截器的作用

    a. 统计api从发起请求到返回数据需要的时间

    b. 配置公共的请求头，加载弹窗等

    c. 对响应状态码做拦截，比入后端返回400或500的状态码， 返回对应错误信息

拦截器的实现

1.引入axios模块，并创建实例

```
import axios from "axios";
 
const axios = axios.create({
  timeout: 3000,
  withCredentials: true
})
```

2.请求拦截器:在请求发送前统一执行某些操作，常用在请求头中处理token等        

```
axios.interceptors.request.use(
  (config) => {  
    Indicator.open({  //加载提示框
        text: '加载中...',
        spinnerType: 'fading-circle'
      });
 
    if (localStorage.token) {
      config.headers["Authorization"] = "Bearer " + localStorage.token;
    }
    return config;
  },
  (err) => {
    return Promise.reject(err);
  }
)
```

3.响应拦截器

```
axios.interceptors.response.use(
  (config) => {
    Indicator.close();  //关闭加载框
    return config;
  },
  (e) => {
	    // 401 一般是token 失效，403 是没有权限
    if (e.response?.status === 401) {
      window.location.reload();
      return Promise.reject("rmp.vehicle.Nopermission");
    }
    if (e.response?.status === 403) {
      return Promise.reject("rmp.vehicle.Nopermission");
    }
    if (e.response.status >= 500) {
      return Promise.reject("serverError");
    }
    return Promise.reject(e);
  }
);
 
export default axios;
```

axios 把用户注册的每个拦截器构造成一个 promise.then 所接受的参数，把相对应的拦截器数组进行调用链的头部和尾部组装，在运行时把所有的拦截器按照一个 promise 链的形式以此执行
