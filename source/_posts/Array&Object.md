---
title: js--Array/Object/String
featured_image: ./images/deadpool.jpeg
---


#### 数组遍历的方法

https://juejin.cn/post/7017289929696739335

---

#### for…in和for…of的用法与区别

for in是遍历（object）键名，for of是遍历（array）键值。

for...in 循环只遍历可枚举属性（包括它的原型链上的可枚举属性）。像 Array和Object使用内置构造函数所创建的对象都会继承自Object.prototype和String.prototype的不可枚举属性，例如 String 的 indexOf()  方法或 Object的toString()方法。循环将遍历对象本身的所有可枚举属性，以及对象从其构造函数原型中继承的属性（更接近原型链中对象的属性覆盖原型属性）。

for...of语句在可迭代对象（包括Array，Map，Set，String，TypedArray，arguments 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句

for of不可以遍历普通对象，想要遍历对象的属性，可以用for in循环, 或内建的Object.keys()方法

无论是for...in还是for...of语句都是迭代一些东西。它们之间的主要区别在于它们的迭代方式。

for...in语句以任意顺序迭代对象的可枚举属性。

for...of 语句遍历可迭代对象定义要迭代的数据。

https://segmentfault.com/a/1190000022348279

---

#### Map()和Set()的区别

https://www.jb51.net/article/237610.htm

---

#### 数组去重

1.Set

```
function unique (arr) {
  return Array.from(new Set(arr))
}
```

2.双层for循环

```
function unique(arr){            
        for(var i=0; i<arr.length; i++){
            for(var j=i+1; j<arr.length; j++){
                if(arr[i]==arr[j]){         //第一个等同于第二个，splice方法删除第二个
                    arr.splice(j,1);
                    j--;
                }
            }
        }
return arr;
}
```

3.sort()排序后去重

```
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return;
    }
    arr = arr.sort()
    var arrry= [arr[0]];
    for (var i = 1; i < arr.length; i++) {
        if (arr[i] !== arr[i-1]) {
            arrry.push(arr[i]);
        }
    }
    return arrry;
}
```

4.includes

```
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var array =[];
    for(var i = 0; i < arr.length; i++) {
            if( !array.includes( arr[i]) ) {//includes 检测数组是否有某个值
                    array.push(arr[i]);
              }
    }
    return array
}
```

5.哈希

```
function arrayNonRepeatfy(arr) {
  let map = new Map();
  let array = new Array();  // 数组用于返回结果
  for (let i = 0; i < arr.length; i++) {
    if(map .has(arr[i])) {  // 如果有该key值
      map .set(arr[i], true); 
    } else { 
      map .set(arr[i], false);   // 如果没有该key值
      array .push(arr[i]);
    }
  } 
  return array ;
}
```
https://segmentfault.com/a/1190000016418021

---

#### 数组原生方法

https://blog.csdn.net/weixin_44024993/article/details/111667910

---

#### 字符串原生方法

https://juejin.cn/post/6844903854287355918

---