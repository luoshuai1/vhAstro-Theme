---
title: call, apply, bind 用法和区别
date: 2021-11-11 15:00:14
categories: Code
tags:
  - JavaScript

id: "js-this"
recommend: true
---

:::note
本文详细介绍了 JavaScript 中 call、apply 和 bind 三种方法的作用和区别，它们主要用于改变函数执行时的上下文（this）。call 和 apply 都用于立即调用函数，call 接受参数为单独的值，apply 则接受一个包含参数的数组。bind 则返回一个新的函数，保留指定的 this 值和参数。文中通过实例展示了如何利用这些方法实现构造函数的继承、借用其他对象的方法，以及处理伪数组等场景
:::

### call, apply, bind 用法和区别

:::note
call() 和 apply()
作用：调用函数，并修改函数运行时的 this 指向
:::

```js
fn.call(thisArg, arg1, arg2, ...)
fn.apply(thisArg, [arg1, arg2, ...])
// thisArg 当前调用函数this的指向对象
// arg1, arg2... 传递的参数

```

### 调用函数

```js
this.name = "张三";
function fn(x, y) {
  console.log(this);
  console.log(this.name);
  console.log(x + y);
}
fn(3, 4); // window对象; "张三"; 7
fn.call(); // window对象; "张三"; NaN
fn.call(null, 2, 4); // window对象; "张三"; 6
fn.bind(null, [3, 4]); //  // window对象; "张三"; 7
```

### 修改函数运行时的this指向



```js
const obj = {
    name: "李四"
}
fn.call(obj, 5, 4) // obj对象; "李四"; 9
fn.bind(obj, [3, 4]) // obj对象; "李四"; 7
```

### 场景

:::note
**1.借用构造函数，继承父类的属性；**
**2.借用其他对象的方法**
:::

```js 
// 例：伪数组借用数组的方法添加成员
var fakeArr = {
    0:'a',
    1:'b',
    length: 2
};
// 数组的push()方法内部的this原本指向数组，改为指向fakeArr
Array.prototype.push.call(fakeArr, 'newItme');
// fakeArr现在的结果为
fakeArr = {
    0:'a',
    1:'b',
    2:'newItem',
    length: 3
}
```

### 区别
:::note
call(thisArg, [val1], [val2], [val3])
第一个参数是改变后的 this 指向，其后参数都是要传入函数的元素；
apply(thisArg, [val1, val2, val3])
第一个参数是改变后的 this 指向，第二个参数必须是数组，apply 可以把数组每一项展开传入函数。
:::

```js 
var arr = [5,1,3,6];
// 使用apply()调用Math.max方法，不需要改变该方法的this指向，所以第一个参数是Math或者null
// 需要把数组里每一项展开，所以第二个参数传入数组
Math.max.apply(Math, arr);
```

### bind()

> 作用:不会调用函数，会返回一个新的函数。该函数的第一个参数是绑定的 this 指向，原函数中的参数通过第二个参数传递。
```js
fn.bind(thisArg, val)

var obj = {
    name: '张三',
    age: 18,
    say: function say(){
        setTimeout(
        	// function(){}.bind(this) 生成一个新的函数，该函数内部this指向原函数的this，原函数是obj调用的，所以this指向obj
        	function(){console.log(this)}.bind(this)
        ,1000)
    }
};
obj.say();

```

