---
sidebar_position: 1
---

# JavaScript手写练习

## 基础手写

### 1.手写防抖函数

函数防抖是指在事件被触发 n 秒后再执行回调，如果在这 n 秒内事件又被触发，则重新计时。这可以使用在一些点击请求的事件上，避免因为用户的多次点击向后端发送多次请求。应用场景：输入框，下拉

按钮提交场景：防⽌多次提交按钮，只执⾏最后提交的⼀次 服务端验证场景：表单验证需要服务端配合，只执⾏⼀段连续的输⼊事件的最后⼀次，还有搜索联想词功能类似⽣存环境请⽤

节流是在事件持续触发时以一定的时间间隔去定时执行回调

函数节流是指规定一个单位时间，在这个单位时间内，只能有一次触发事件的回调函数执行，如果在同一个单位时间内某事件被触发多次，只有一次能生效。节流可以使用在 scroll 函数的事件监听上，通过事件节流来降低事件调用的频率。

**防抖是每隔指定的时间发起请求** 用户滑动时 定时 / 定滑动的高度

拖拽场景：固定时间内只执⾏⼀次，防⽌超⾼频次触发位置变动 缩放场景：监控浏览器resize 动画场景：避免短时间内多次触发动画引起性能问题

**非立即防抖**

非立即执行函数： 多次触发事件，只会在最后一次触发事件后等待设定的wait时间结束时执行一次

```javascript
function debounce(fn,delay) {
  let timer = null
  return function(...args) {
    clearTimeout(timer)
    timer = null
    timer = setTimeout(() => {
      fn.apply(this,args)
    },delay)
  }
}
function task(){
    console.log('run task')
}
const debounceTask=debounce(task,1000)
window.addEventListener('scroll',debounceTask);
```

**立即防抖**

立即执行：即多次触发事件，第一次会立即执行函数，之后在设定wait事件内触犯的事件无效，不会执行。

设置clearTimeout为什么还要timer=null 设置延时器之前先清除下延时器，可以立即触发垃圾回收机制，不然每次事件触发都会多一个延时器，延时器之间互相干扰，造成紊乱。**清除定时器后变量仍然为数字**。

```javascript
function debounce(fn,wait){
  let timerId = null;
  let flag = true;
  return function(){
    clearTimeout(timerId);
    if(flag){
      fn.apply(this,arguments);
      flag = false
      }
    timerId = setTimeout(() => { flag = true},wait)
  }
}
//测试
function task(){
    console.log('run task')
}
const debounceTask=debounce(task,1000)
window.addEventListener('scroll',debounceTask);
```

### 2.手写节流函数

**非立即节流** 连续点击的话，每过 wait 秒执行一次

```javascript
function throttle(fn, time) {
    let timer = null
    return function (...args) {
        if (!timer) {
            timer = setTimeout(() => {
                fn.apply(this,args)
                timer = null
            }, time)
        }
    }
}
function task(){
    console.log('run task')
}
const throttleTask=throttle(task,1000);
window.addEventListener('scroll',throttleTask);
```

**立即节流** 连续点击的话，第一下点击会立即执行一次 然后每过 wait 秒执行一次

```js
function throttle(fn,delay){
    let last=Date.now();
    return function(...args){
        const now =Date.now();
        if(now-last>delay){
            last=now;
            fn.apply(this,args)
			console.log(this)
        }
    }
}
function task(){
    console.log('run task')
}
const throttleTask=throttle(task,1000);
window.addEventListener('scroll',throttleTask);
```

### 3.手写浅拷贝深拷贝

浅拷贝是指，一个新的对象对原始对象的属性值进行精确地拷贝，如果拷贝的是基本数据类型，拷贝的就是基本数据类型的值，如果是引用数据类型，拷贝的就是内存地址。如果其中一个对象的引用内存地址发生改变，另一个对象也会发生变化。

#### 浅拷贝

##### Object.assign()

Object.assign() 是ES6中对象的拷贝方法，接受的第一个参数是目标对象，其余参数是源对象，用法： Object.assign(target, source1, ···) ，该方法可以实现浅拷贝，也可以实现一维对象的深拷贝。

```js
let target = {a: 1};
let object2 = {b: 2};
let object3 = {c: 3};
Object.assign(target,object2,object3);
console.log(target); // {a: 1, b: 2, c: 3}
```

##### 扩展运算符

使用扩展运算符可以在构造字面量对象的时候，进行属性的拷贝。语法： let cloneObj = {...obj };

```js
let obj1 = {a:1,b:{c:1}
let obj2 = {...obj1};
obj1.a = 2;
console.log(obj1); 
//{a:2,b:{c:1}}
console.log(obj2);
//{a:1,b:{c:1}}
obj1.b.c = 2;
console.log(obj1); 
//{a:2,b:{c:2}}
console.log(obj2); 
//{a:1,b:{c:2}}
```

##### 数组方法实现数组浅拷贝

###### Array.prototype.slice()

array.slice(start, end)

从已有数组中返回选定的元素，该方法有两个参数，两个参数都可选，如果两个参数都不写，就可以实现一个数组的浅拷贝

###### Array.prototype.concat()

concat() 用于合并两个或多个数组 返回一个新数组

##### 手写浅拷贝

```js
function shallowCopy(object){
    if(!object||typeof object!=="object") return
    let newObject=Array.isArray(object)?[]:{}
    for(let key in object){
      if(object.hasOwnProperty(key)){
        newObject[key]=object[key]
      }
    }
  return newObject
}
```

##### 手写深拷贝

支持对象、数组、日期、正则的拷贝。

处理原始类型（原始类型直接返回，只有引用类型才有深拷贝这个概念）。

处理 Symbol 作为键名的情况。

处理函数（函数直接返回，拷贝函数没有意义，两个对象使用内存中同一个地址的函数，问题不大）。

处理 DOM 元素（DOM 元素直接返回，拷贝 DOM 元素没有意义，都是指向页面中同一个）。

额外开辟一个储存空间 WeakMap，解决循环引用递归爆栈问题（引入 WeakMap 的另一个意义，配合垃圾回收机制，防止内存泄漏）。

```js
function deepClone (target, hash = new WeakMap()) { // 额外开辟一个存储空间WeakMap来存储当前对象
  if (target === null) return target // 如果是 null 就不进行拷贝操作
  if (target instanceof Date) return new Date(target) // 处理日期
  if (target instanceof RegExp) return new RegExp(target) // 处理正则
  if (target instanceof HTMLElement) return target // 处理 DOM元素

  if (typeof target !== 'object') return target // 处理原始类型和函数 不需要深拷贝，直接返回

  // 是引用类型的话就要进行深拷贝
  if (hash.get(target)) return hash.get(target) // 当需要拷贝当前对象时，先去存储空间中找，如果有的话直接返回
  const cloneTarget = new target.constructor() // 创建一个新的克隆对象或克隆数组
  hash.set(target, cloneTarget) // 如果存储空间中没有就存进 hash 里

  Reflect.ownKeys(target).forEach(key => { // 引入 Reflect.ownKeys，处理 Symbol 作为键名的情况
    cloneTarget[key] = deepClone(target[key], hash) // 递归拷贝每一层
  })
  return cloneTarget // 返回克隆的对象
}


const obj = {
  a: true,
  b: 100,
  c: 'str',
  d: undefined,
  e: null,
  f: Symbol('f'),
  g: {
    g1: {} // 深层对象
  },
  h: [], // 数组
  i: new Date(), // Date
  j: /abc/, // 正则
  k: function () {}, // 函数
  l: [document.getElementById('foo')] // 引入 WeakMap 的意义，处理可能被清除的 DOM 元素
}
obj.obj = obj // 循环引用
const name = Symbol('name')
obj[name] = 'lin'  // Symbol 作为键
const newObj = deepClone(obj)
console.log(newObj)
```

### 4.手写ajax

1. 创建 XMLHttpRequest 实例
2. 发出 HTTP 请求
3. 接收服务器传回的数据
4. 更新网页数据

```js
function handleGet(url) {
        var response = "";
        //1.创建对象
        var xhr = new XMLHttpRequest();
        //2.设置方法
        xhr.open("GET", url);
        //3.发送请求
        xhr.send();
        //4.返回结果
        xhr.onreadystatechange = function () {
          if (xhr.readyState == 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
              response = xhr.responseText;
              console.log(response, "1");
            } else {
              return "Error" + xhr.status;
            }
          }
        };
      }
```

### 5.使用promise实现ajax

```js
// promise 封装实现：
function getJSON(url) {
  // 创建一个 promise 对象
  let promise = new Promise(function(resolve, reject) {
    let xhr = new XMLHttpRequest();
    // 新建一个 http 请求
    xhr.open("GET", url, true);
    // 设置状态的监听函数
    xhr.onreadystatechange = function() {
      if (this.readyState !== 4) return;
      // 当请求成功或失败时，改变 promise 的状态
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    // 设置错误监听函数
    xhr.onerror = function() {
      reject(new Error(this.statusText));
    };
    // 设置响应的数据类型
    xhr.responseType = "json";
    // 设置请求头信息
    xhr.setRequestHeader("Accept", "application/json");
    // 发送 http 请求
    xhr.send(null);
  });
  return promise;
}
```

### 6.手写 apply call bind

#### apply实现

判断调用对象是否为函数，即使我们是定义在函数的原型上的，但是可能出现使用 call 等方式调用的情况。

判断传入上下文对象是否存在，如果不存在，则设置为 window 。

将函数作为上下文对象的一个属性。

判断参数值是否传入

使用上下文对象来调用这个方法，并保存返回结果。

删除刚才新增的属性

返回结果

```js
Function.prototype.myApply=function (context,args){
    context = context || window;

    if(typeof this !=="function"){
        return new Error('typeError')
    }
    context.fn=this
    if(args){
        result=context.fn(...args)
    }else{
        result=context.fn()
    }
    delete context.fn
    return result
}
function bar(age,sex) {
  return {
    name: this.name,
    age,
    sex
  }
}
const obj = {
    name: 'cyx'
}
bar.myApply( obj, [22, "男"] )   // {name: "cyx", age: 22, sex: "男"}
```

#### call实现

判断调用对象是否为函数，即使我们是定义在函数的原型上的，但是可能出现使用 call 等方式调用的情况。

判断传入上下文对象是否存在，如果不存在，则设置为 window 。

处理传入的参数，截取第一个参数后的所有参数。

将函数作为上下文对象的一个属性。

使用上下文对象来调用这个方法，并保存返回结果。

删除刚才新增的属性。

返回结果。

```js
 Function.prototype.myCall = function (context) {
        context = context || window;
        if (typeof this !== "function") {
          throw new Error("typeError");
        }
        let result;
        context.fn = this;
        let args = [...arguments].slice(1);
        if (args) {
          result = context.fn(...args);
        } else {
          result = context.fn();
        }
        delete context.fn;
        return result;
      };
      var obj = {
        n: 15,
      };
      function sum(n, m) {
        console.log(this);
        return n + m;
      }
      console.log(sum.myCall(obj, 10, 20));
```

#### bind实现

```js
// bind 函数实现
Function.prototype.myBind = function(context) {
  // 判断调用对象是否为函数
  if (typeof this !== "function") {
    throw new TypeError("Error");
  }
  // 获取参数
  var args = [...arguments].slice(1),
     fn = this;
  return function Fn() {
    // 根据调用方式，传入不同绑定值
    return fn.apply(
      this instanceof Fn ? this : context,
      args.concat(...arguments)
    );
  };
};
var obj = {
        n: 15,
      };
function sum(n, m) {
        console.log(this);
        return n + m;
      }
var ans=sum.myBind(obj, 10, 20)
console.log(ans());
```



