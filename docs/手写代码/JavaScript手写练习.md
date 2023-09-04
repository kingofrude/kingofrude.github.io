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

### 7.手写promise方法

#### 手写Promise.all

`Promise.all(iterable)` 方法返回一个 `Promise` 实例，此实例在 `iterable` 参数内所有的 `promise` 都“完成（resolved）”或参数中不包含 `promise` 时回调完成（resolve）；如果参数中  `promise` 有一个失败（rejected），此实例回调失败（reject），失败原因的是第一个失败 `promise` 的结果

```js
function promiseAll(arr) {
    if(!Array.isArray(arr)) {
        return new TypeError('must array')
    }
    return new Promise((resolve, reject) => {
        let count = 0, res = Array(arr.length).fill(0)
        arr.forEach((item, index) => {
            Promise.resolve(item).then((value) => {
                count++
                res[index] = value
                if(count === arr.length) {
                    resolve(res)
                }
            }, (value) => {
                reject(value)
            })
        })
    })
}
function test(num, delay) {
        return new Promise((resolve, reject) => {
          setTimeout(() => {
            num == 4 ? reject(num) : resolve(num);
          }, delay);
        });
      }
      let p1 = test(1, 1000);
      let p2 = test(2, 2000);
      let p3 = test(3, 3000);
      let p4 = test(4, 4000);
      myPromiseAll([p1, p2, p3]).then((result) => {
        console.log(result);
      });
```

#### **手写promise.allSettled**

**Promise.allSettled()**方法返回一个`promise`，该`promise`在所有给定的`promise`已被解析或被拒绝后解析，并且每个对象都描述每个`promise`的结果。

`Promise.allSettled()` 可用于并行执行独立的异步操作，并收集这些操作的结果

该函数接受一个 `promise` 数组(通常是一个可迭代对象)作为参数:

```js
const statusesPromise = Promise.allSettled(promises);
```

当所有的输入 `promises` 都被 `fulfilled` 或 `rejected` 时，`statusesPromise` 会解析为一个具有它们状态的数组

1. `{ status: 'fulfilled', value: value }` — 如果对应的 promise 已经 `fulfilled`
2. 或者 `{status: 'rejected'， reason: reason}` 如果相应的 promise 已经被 `rejected`

```js
// 写法一
function promiseAllSettled(arr) {
    if (!Array.isArray(arr)) {
        return new TypeError('must array')
    }
    return new Promise((resolve, reject) => {
        let count = 0, res = Array(arr.length).fill(0)
        arr.forEach((item, index) => {
            Promise.resolve(item).then((value) => {
                count++
                res[index] = { status: 'fulfilled', value: value }
                if (count === arr.length) {
                    resolve(res)
                }
            }, (error) => {
                count++
                res[index] = { status: 'rejected', reason: error }
                if (count === arr.length) {
                    resolve(res)
                }
            })
        })
    })
}
// 写法二 推荐这种 另外， promise.then的第二个参数函数捕获错误的优先级高于.catch捕获错误的优先级，且第二个捕获错误之后，catch便不再捕获。
function PromiseAllSettled(list) {
    return new Promise((resolve) => {
        let result = []
        let count = 0
        list.forEach((item, index) => {
            item.then((res) => {
                result[index] = { status: 'fulfilled', value: res }
            }).catch((err) => {
                result[index] = { status: 'rejected', reason: err }
            }).finally(() => {
                count++
                count === list.length && resolve(result)
            })
        })
    })
}
const promiseList = [
    new Promise((resolve) => setTimeout(() => resolve('成功1'), 2000)),
    new Promise((resolve, reject) => setTimeout(() => reject('失败1'), 1000)),
    new Promise((resolve) => setTimeout(() => resolve('成功3'), 2600)),
    new Promise((resolve) => setTimeout(() => resolve('成功4'), 2300)),
]
PromiseAllSettled(promiseList).then((res) => console.log(res))
Promise.allSettled(promiseList).then((res) => console.log(res))
```

#### **手写promise.finally**

`finally()` 方法返回一个[`Promise`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise)。在promise结束时，无论结果是fulfilled或者是rejected，都会执行指定的回调函数。这为在`Promise`是否成功完成后都需要执行的代码提供了一种方式。

这避免了同样的语句需要在[`then()`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fthen)和[`catch()`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fcatch)中各写一次的情况。

**简单理解**：finally 的特点 无论如何都执行 ，但是如果返回的是一个promise需要等待这个promise之行完在继续向下执行

finally后面还可以.then .then 所以finally本质上就是一个`then`

```js
Promise.prototype.finally = function (cb) {
  return this.then((data) => {
    // 如何保证Promise.then能够执行完毕
    return Promise.resolve(cb()).then((n) => data);
  }, (err) => {
    // Promise.resolve 目的是等待cb()后的Promise执行完成
    return Promise.resolve(cb()).then((n) => { throw err });
  })
}
```

#### 手写promise.race

**Promise.race(iterable)** 方法返回一个 `promise`，一旦迭代器中的某个`promise`解决或拒绝，返回的 `promise`就会解决或拒绝。

```js
function PromiseRace(arrayList) {
        return new Promise((resolve) => {
          for (let i = 0; i < arrayList.length; i++) {
            Promise.resolve(arrayList[i]).then((result) => {
              resolve(result);
            });
          }
        });
      }
      function test(num, time) {
        return new Promise((resolve, reject) => {
          setTimeout(() => {
            num == 4 ? reject() : resolve(num);
          }, time);
        });
      }
      let p1 = test(1, 2000);
      let p2 = test(2, 1000);
      let p3 = test(3, 5000);
      PromiseRace([p3, p1, p2]).then((res) => {
        console.log(res);
      });
```

#### **手写promise.any**

全部状态失败才返回失败，其中有成功的状态就返回那个成功的

```js
function promiseAny(list) {
    return new Promise((resolve, reject) => {
        let count = 0
        list.forEach((item, index) => {
            item.then((res) => resolve(res)).catch((err) => {
                count++
                count === list.length && reject('都失败了')
            })
        })
    })
}

const promiseList = [
    new Promise((resolve) => setTimeout(() => resolve('成功1'), 3000)),
    new Promise((resolve, reject) => setTimeout(() => reject('失败1'), 1000)),
    new Promise((resolve) => setTimeout(() => resolve('成功3'), 2600)),
    new Promise((resolve) => setTimeout(() => resolve('成功4'), 2300)),
]

promiseAny(promiseList).then((res) => console.log(res)).catch((err) => console.log(err))

Promise.any(promiseList).then((res) => console.log(res)).catch((err) => console.log(err))
```

#### 实现 Promise.resolve

> 实现 resolve 静态方法有三个要点:

- 传参为一个 `Promise`, 则直接返回它。
- 传参为一个 `thenable` 对象，返回的 `Promise` 会跟随这个对象，采用它的最终状态作为自己的状态。
- 其他情况，直接返回以该值为成功状态的`promise`对象。

```text
Promise.resolve = (param) => {
  if(param instanceof Promise) return param;
  return new Promise((resolve, reject) => {
    if(param && param.then && typeof param.then === 'function') {
      // param 状态变为成功会调用resolve，将新 Promise 的状态变为成功，反之亦然
      param.then(resolve, reject);
    }else {
      resolve(param);
    }
  })
}
```

#### 实现 Promise.reject

> Promise.reject 中传入的参数会作为一个 reason 原封不动地往下传, 实现如下:

```js
Promise.reject = function (reason) {
    return new Promise((resolve, reject) => {
        reject(reason);
    });
}
```

### 8.实现函数柯里化

#### 1.固定参数

理解  arg数组是不断被扩展的 length指的是传入的test函数的参数长度

test实际上就是内部的 temp()

```js
 function curry(fn) {
        let length = fn.length;
        return function temp() {
          let arg = [...arguments];
          if (arg.length >= length) {
            return fn(...arg);
          } else {
            return function () {
              return temp(...arg, ...arguments);
            };
          }
        };
      }
      function add(a, b, c) {
        return a + b + c;
      }
      let test = curry(add);
      console.log(test(1)(2, 3));
```

#### 2.不固定参数

通过最后一个不传参来判断执行结束

```js
// 每次调用的传进来的参数做累计处理
function reduce (...args) {
    return args.reduce((a, b) => a + b)
}
function currying (fn) {
  // 存放每次调用的参数
  let args = []
  return function temp (...newArgs) {
    if (newArgs.length) {
      // 有参数就合并进去，然后返回自身
      args = [ ...args, ...newArgs ]
      return temp
    } else {
      // 没有参数了，也就是最后一个了，执行累计结果操作并返回结果
      let val = fn.apply(this, args)
      args = [] //保证再次调用时清空
      return val
    }
  }
}
let add = currying(reduce)
console.log(add(1)(2, 3, 4)(5))  //temp{ }
console.log(add(1)(2, 3, 4)(5)())  //15
console.log(add(1)(2, 3)(4, 5)())  //15
```



```js
function curry(fn) {
        let arg = [...arguments].slice(1);
        let temp = function temp() {
          arg = [...arg, ...arguments];
          return curry(fn, ...arg);
        };
        temp.toString = function () {
          return fn.apply(null, arg);
        };
        return temp;
      }

      function fn() {
        return [...arguments].reduce((pre, cur) => {
          return pre + cur;
        }, 0);
      }
      let test = curry(fn);
			console.log(test())
      console.log(test(1)(2, 3)(4)(5);  //temp
      console.log(test(1)(2, 3)(4)(5).toString());
```

#### 3.固定参数的add函数

```js
function add(x) {
      let sum = x;
      let temp = function (y) {
        sum += y;
        return temp;
      };
      temp.toString = function () {
        return sum;
      };
      return temp;
    }
    let a = add(1)(2)(4);
    console.log(a.toString());
```



#### 4.不固定参数的add函数

```js
 function add() {
        let arg = [...arguments];
        let add = function () {
          arg.push(...arguments);
          return add;
        };
        add.toString= function () {
          return arg.reduce((pre, cru) => {
            return pre + cru;
          });
        };
        return add;
      }
      console.log(add(1)(2)(3)(4)(5).toString());
```



```js
add(1); 			// 1
add(1)(2);  	// 3
add(1)(2)(3)；// 6
add(1)(2, 3); // 6
add(1, 2)(3); // 6
add(1, 2, 3); // 6
function add() {
  let args = [].slice.call(arguments);

  let fn = function(){
   let fn_args = [].slice.call(arguments)
   return add.apply(null,args.concat(fn_args))
  }

  fn.toString = function(){
    return args.reduce((a,b)=>a+b)
  }
  return fn
}
```

### 9.实现new

1. 创建了一个全新的对象。
2. 新对象的`__proto__`属性指向构造函数的`prototype`属性。
3. 生成的新对象会绑定到函数调用的`this`。
4. 通过`new`创建的每个对象将最终被链接到这个函数的`prototype`对象上
5. 如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，那么`new`表达式中的函数调用会自动返回这个新的对象

```js
function myNew(Fn) {
    if (typeof Fn !== 'function') throw new TypeError('This is not a constructor') // Fn校验
    var args = Array.from(arguments).slice(1) // 取入参
    var obj = {} // 1.创建一个空的简单JavaScript对象（即`  {}  `）
    obj.__proto__ = Fn.prototype // 2.  为步骤1新创建的对象添加属性`  __proto__  `，将该属性链接至构造函数的原型对象
    var res = Fn.call(obj, ...args) // 3.  将步骤1新创建的对象作为this的上下文并传入参数；
    return Object(res) === res ? res : obj // 4.  如果该函数没有返回对象，则返回this。
}
```

### 10.模拟 Object.create

> Object.create()`方法创建一个新对象，使用现有的对象来提供新创建的对象的 `__proto__

```js
// 模拟 Object.create
function create(proto) {
  function F() {}
  F.prototype = proto;
  return new F();
}
```

### 11.实现Instanceof

```js
function myInstanceof(left, right) {
        let proto = Object.getPrototypeOf(left),
          prototype = right.prototype;

        while (1) {
          if (!proto) return false;
          if (proto === prototype) return true;

          proto = Object.getPrototypeOf(proto);
        }
      }
      console.log(myInstanceof([1, 3, 4], Array));
      console.log(myInstanceof({}, Array));
      console.log(myInstanceof(new Date(), Object));
      console.log(myInstanceof(2, Array));
```

### 12.手写promise

[学习文章](https://juejin.cn/post/6945319439772434469)

#### 链式调用原理

我门常常用到 new Promise().then().then() ,这就是链式调用，用来解决回调地狱

1、为了达成链式，**我们默认在第一个then里返回一个promise**。规定了一种方法，就是**在then里面返回一个新的promise**,称为promise2： promise2 = new Promise((resolve, reject)=>{})

- **将这个promise2返回的值传递到下一个then中**
- 如果返回一个普通的值，则将**普通的值**传递给下一个then中

2、当我们在第一个then中 return 了一个参数（参数未知，需判断）。这个return出来的新的promise就是onFulfilled()或onRejected()的值

**规定onFulfilled()或onRejected()的值，即第一个then返回的值，叫做x，判断x的函数叫做resolvePromise**

```js
then(onFulfilled,onRejected) {
    // 声明返回的promise2
    let promise2 = new Promise((resolve, reject)=>{
      if (this.state === 'fulfilled') {
        let x = onFulfilled(this.value);
        resolvePromise(promise2, x, resolve, reject);
      };
      if (this.state === 'rejected') {
        let x = onRejected(this.reason);
        resolvePromise(promise2, x, resolve, reject);
      };
      if (this.state === 'pending') {
        this.onResolvedCallbacks.push(()=>{
          let x = onFulfilled(this.value);
          resolvePromise(promise2, x, resolve, reject);
        })
        this.onRejectedCallbacks.push(()=>{
          let x = onRejected(this.value);
          resolvePromise(promise2, x, resolve, reject);
        })
      }
    });
    // 返回promise，完成链式
    return promise2;
  }
```

规定了一段代码，让不同的promise代码互相套用，叫做resolvePromise
如果 x === promise2，则是会造成循环引用，自己等待自己完成，则报“循环引用”错误

```js
let p = new Promise(resolve => {
  resolve(0);
});
var p2 = p.then(data => {
  // 循环引用，自己等待自己完成，一辈子完不成
  return p2;
})
```

1、判断x

- x 不能是null
- x 是普通值 直接resolve(x)
- x 是对象或者函数（包括promise）， let then = x.then 

2、当x是对象或者函数（默认promise）

- 声明了then
- 如果取then报错，则走reject()
- 如果then是个函数，则用call执行then，第一个参数是this，后面是成功的回调和失败的回调
- 如果成功的回调还是pormise，就递归继续解析

成功和失败只能调用一个 所以**设定一个called来防止多次调用**

```js
function resolvePromise(promise2, x, resolve, reject){
  // 循环引用报错
  if(x === promise2){
    // reject报错
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
  // 防止多次调用
  let called;
  // x不是null 且x是对象或者函数
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      // A+规定，声明then = x的then方法
      let then = x.then;
      // 如果then是函数，就默认是promise了
      if (typeof then === 'function') { 
        // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
        then.call(x, y => {
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          // resolve的结果依旧是promise 那就继续解析
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          reject(err);// 失败了就失败了
        })
      } else {
        resolve(x); // 直接成功即可
      }
    } catch (e) {
      // 也属于失败
      if (called) return;
      called = true;
      // 取then出错了那就不要在继续执行了
      reject(e); 
    }
  } else {
    resolve(x);
  }
}
```

#### 不完全实现（没有完全实现链式调用）

```javascript
/*  Promise有三个状态,包括Pending,resolved,rejected */
      //状态定义
      const PENDING = "pending";
      const RESOLVED = "resolved";
      const REJECTED = "rejected";

      function MyPromise() {
        //保存初始化状态
        var self = this;
        //初始化状态
        this.state = PENDING;
        //用于保存resolve或者rejected传入的值
        this.value = null;
        //用于保存resolve的回调函数
        this.resolvedCallbacks = [];
        //用于保存reject的回调函数
        this.rejectedCallbacks = [];

        //状态转变为resolved方法
        function resolved(value) {
          if (value instanceof MyPromise) {
            return value.then(resolved, rejected);
          }

          //保证代码的执行顺序为本轮事件循环的末尾
          setTimeout(() => {
            //   只有状态pending才改变
            if (self.state === PENDING) {
              //改变状态
              self.status = RESOLVED;
              //设置传入的值
              self.value = value;
              //执行回调函数
              self.resolvedCallbacks.forEach((callback) => {
                callback(back);
              });
            }
          }, 0);
        }

        //状态转变为rejected方法
        function rejected(value) {
          //保证代码的执行顺序为本轮事件循环的末尾
          setTimeout(() => {
            //   只有状态pending才改变
            if (self.state === PENDING) {
              //改变状态
              self.status = REJECTED;
              //设置传入的值
              self.value = value;
              //执行回调函数
              self.rejectedCallbacks.forEach((callback) => {
                callback(back);
              });
            }
          }, 0);
        }

        //将两个方法传入函数执行
        try {
          fn(resolve, reject);
        } catch (e) {
          //遇到错误时,捕获错误,执行reject函数
          reject(e);
        }

        MyPromise.prototype.then = function (onResolved, onRejected) {
          //首先判断两个参数是否为函数类型,因为这两个参数是可选参数
          onResolved =
            typeof onResolved === "function"
              ? onResolved
              : function (value) {
                  return value;
                };
          onRejected =
            typeof onRejected === "function"
              ? onRejected
              : function (error) {
                  throw error;
                };

          //如果是等待状态,则将函数加入对应列表中
          if (this.status === PENDING) {
            this.resolvedCallbacks.push(onResolved);
            this.rejectedCallbacks.push(onRejected);
          }
          if (this.state === RESOLVED) {
            onResolved(this.value);
          }
          if (this.state === REJECTED) {
            onRejected(this.value);
          }
        };
      }
```

#### 完全规范实现

```js
class Promise{
  constructor(executor){
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.onResolvedCallbacks.forEach(fn=>fn());
      }
    };
    let reject = reason => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.onRejectedCallbacks.forEach(fn=>fn());
      }
    };
    try{
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled,onRejected) {
    // onFulfilled如果不是函数，就忽略onFulfilled，直接返回value
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    // onRejected如果不是函数，就忽略onRejected，直接扔出错误
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
    let promise2 = new Promise((resolve, reject) => {
      if (this.state === 'fulfilled') {
        // 异步
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'rejected') {
        // 异步
        setTimeout(() => {
          // 如果报错
          try {
            let x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'pending') {
        this.onResolvedCallbacks.push(() => {
          // 异步
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0);
        });
        this.onRejectedCallbacks.push(() => {
          // 异步
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0)
        });
      };
    });
    // 返回promise，完成链式
    return promise2;
  }
catch(fn) {
		return this.then(null, fn);
	}
    
}
function resolvePromise(promise2, x, resolve, reject) {
	if (x === promise2) {
		return reject(new TypeError('Chaining cycle detected for promise'));
	}
	let called;
	if (x != null && (typeof x === 'object' || typeof x === 'function')) {
		try {
			let then = x.then;
			if (typeof then === 'function') {
				then.call(x, y => {
					if (called) return;
					called = true;
					resolvePromise(promise2, y, resolve, reject);
				}, err => {
					if (called) return;
					called = true;
					reject(err);
				})
			} else {
				resolve(x);
			}
		} catch (e) {
			if (called) return;
			called = true;
			reject(e);
		}
	} else {
		resolve(x);
	}
}
```

#### 全部方法实现

```js
class Promise{
  constructor(executor){
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.onResolvedCallbacks.forEach(fn=>fn());
      }
    };
    let reject = reason => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.onRejectedCallbacks.forEach(fn=>fn());
      }
    };
    try{
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled,onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
    let promise2 = new Promise((resolve, reject) => {
      if (this.state === 'fulfilled') {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'rejected') {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'pending') {
        this.onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0);
        });
        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0)
        });
      };
    });
    return promise2;
  }
  catch(fn){
    return this.then(null,fn);
  }
}
function resolvePromise(promise2, x, resolve, reject){
  if(x === promise2){
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
  let called;
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      let then = x.then;
      if (typeof then === 'function') { 
        then.call(x, y => {
          if(called)return;
          called = true;
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          if(called)return;
          called = true;
          reject(err);
        })
      } else {
        resolve(x);
      }
    } catch (e) {
      if(called)return;
      called = true;
      reject(e); 
    }
  } else {
    resolve(x);
  }
}
//resolve方法
Promise.resolve = function(val){
  return new Promise((resolve,reject)=>{
    resolve(val)
  });
}
//reject方法
Promise.reject = function(val){
  return new Promise((resolve,reject)=>{
    reject(val)
  });
}
//race方法 
Promise.race = function(promises){
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(resolve,reject)
    };
  })
}
//all方法(获取所有的promise，都执行then，把结果放到数组，一起返回)
Promise.all = function(promises){
  let arr = [];
  let i = 0;
  function processData(index,data){
    arr[index] = data;
    i++;
    if(i == promises.length){
      resolve(arr);
    };
  };
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(data=>{
        processData(i,data);
      },reject);
    };
  });
}
```

#### 通过A+测试

```js
// 先定义三个常量表示状态
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

// 新建 MyPromise 类
class MyPromise {
	constructor(executor) {
		// executor 是一个执行器，进入会立即执行
		// 并传入resolve和reject方法
		try {
			executor(this.resolve, this.reject)
		} catch (error) {
			this.reject(error)
		}
	}

	// 储存状态的变量，初始值是 pending
	status = PENDING;

	// resolve和reject为什么要用箭头函数？
	// 如果直接调用的话，普通函数this指向的是window或者undefined
	// 用箭头函数就可以让this指向当前实例对象
	// 成功之后的值
	value = null;
	// 失败之后的原因
	reason = null;

	// 存储成功回调函数
	onFulfilledCallbacks = [];
	// 存储失败回调函数
	onRejectedCallbacks = [];

	// 更改成功后的状态
	resolve = (value) => {
		// 只有状态是等待，才执行状态修改
		if (this.status === PENDING) {
			// 状态修改为成功
			this.status = FULFILLED;
			// 保存成功之后的值
			this.value = value;
			// resolve里面将所有成功的回调拿出来执行
			while (this.onFulfilledCallbacks.length) {
				// Array.shift() 取出数组第一个元素，然后（）调用，shift不是纯函数，取出后，数组将失去该元素，直到数组为空
				this.onFulfilledCallbacks.shift()(value)
			}
		}
	}

	// 更改失败后的状态
	reject = (reason) => {
		// 只有状态是等待，才执行状态修改
		if (this.status === PENDING) {
			// 状态成功为失败
			this.status = REJECTED;
			// 保存失败后的原因
			this.reason = reason;
			// resolve里面将所有失败的回调拿出来执行
			while (this.onRejectedCallbacks.length) {
				this.onRejectedCallbacks.shift()(reason)
			}
		}
	}

	then(onFulfilled, onRejected) {
		const realOnFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
		const realOnRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };

		// 为了链式调用这里直接创建一个 MyPromise，并在后面 return 出去
		const promise2 = new MyPromise((resolve, reject) => {

			const fulfilledMicrotask = () => {
				// 创建一个微任务等待 promise2 完成初始化
				queueMicrotask(() => {
					try {
						// 获取成功回调函数的执行结果
						const x = realOnFulfilled(this.value);
						// 传入 resolvePromise 集中处理
						resolvePromise(promise2, x, resolve, reject);
					} catch (error) {
						reject(error)
					}
				})
			}

			const rejectedMicrotask = () => {
				// 创建一个微任务等待 promise2 完成初始化
				queueMicrotask(() => {
					try {
						// 调用失败回调，并且把原因返回
						const x = realOnRejected(this.reason);
						// 传入 resolvePromise 集中处理
						resolvePromise(promise2, x, resolve, reject);
					} catch (error) {
						reject(error)
					}
				})
			}
			// 判断状态
			if (this.status === FULFILLED) {
				fulfilledMicrotask()
			} else if (this.status === REJECTED) {
				rejectedMicrotask()
			} else if (this.status === PENDING) {
				// 等待
				// 因为不知道后面状态的变化情况，所以将成功回调和失败回调存储起来
				// 等到执行成功失败函数的时候再传递
				this.onFulfilledCallbacks.push(fulfilledMicrotask);
				this.onRejectedCallbacks.push(rejectedMicrotask);
			}
		})

		return promise2;
	}

	catch(onRejected) {
		// 只需要进行错误处理
		this.then(undefined, onRejected);
	}

	finally(fn) {
		return this.then((value) => {
			return MyPromise.resolve(fn()).then(() => {
				return value;
			});
		}, (error) => {
			return MyPromise.resolve(fn()).then(() => {
				throw error
			});
		});
	}

	// resolve 静态方法
	static resolve(parameter) {
		// 如果传入 MyPromise 就直接返回
		if (parameter instanceof MyPromise) {
			return parameter;
		}

		// 转成常规方式
		return new MyPromise(resolve => {
			resolve(parameter);
		});
	}

	// reject 静态方法
	static reject(reason) {
		return new MyPromise((resolve, reject) => {
			reject(reason);
		});
	}

	static all(promiseList) {
		return new MyPromise((resolve, reject) => {
			const result = [];
			const length = promiseList.length;
			let count = 0;

			if (length === 0) {
				return resolve(result);
			}

			promiseList.forEach((promise, index) => {
				MyPromise.resolve(promise).then((value) => {
					count++;
					result[index] = value;
					if (count === length) {
						resolve(result);
					}
				}, (reason) => {
					reject(reason);
				});
			});
		});

	}

	static allSettled = (promiseList) => {
		return new MyPromise((resolve) => {
			const length = promiseList.length;
			const result = [];
			let count = 0;

			if (length === 0) {
				return resolve(result);
			} else {
				for (let i = 0; i < length; i++) {
					const currentPromise = MyPromise.resolve(promiseList[i]);
					currentPromise.then((value) => {
						count++;
						result[i] = {
							status: 'fulfilled',
							value: value
						}
						if (count === length) {
							return resolve(result);
						}
					}, (reason) => {
						count++;
						result[i] = {
							status: 'rejected',
							reason: reason
						}
						if (count === length) {
							return resolve(result);
						}
					});
				}
			}
		});
	}

	static race(promiseList) {
		return new MyPromise((resolve, reject) => {
			const length = promiseList.length;

			if (length === 0) {
				return resolve();
			} else {
				for (let i = 0; i < length; i++) {
					MyPromise.resolve(promiseList[i]).then((value) => {
						return resolve(value);
					}, (reason) => {
						return reject(reason);
					});
				}
			}
		});
	}
}

function resolvePromise(promise, x, resolve, reject) {
	// 如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise
	// 这是为了防止死循环
	if (promise === x) {
		return reject(new TypeError('The promise and the return value are the same'));
	}

	if (typeof x === 'object' || typeof x === 'function') {
		// 这个坑是跑测试的时候发现的，如果x是null，应该直接resolve
		if (x === null) {
			return resolve(x);
		}

		let then;
		try {
			// 把 x.then 赋值给 then 
			then = x.then;
		} catch (error) {
			// 如果取 x.then 的值时抛出错误 e ，则以 e 为据因拒绝 promise
			return reject(error);
		}

		// 如果 then 是函数
		if (typeof then === 'function') {
			let called = false;
			// 将 x 作为函数的作用域 this 调用之
			// 传递两个回调函数作为参数，第一个参数叫做 resolvePromise ，第二个参数叫做 rejectPromise
			// 名字重名了，我直接用匿名函数了
			try {
				then.call(
					x,
					// 如果 resolvePromise 以值 y 为参数被调用，则运行 [[Resolve]](promise, y)
					y => {
						// 如果 resolvePromise 和 rejectPromise 均被调用，
						// 或者被同一参数调用了多次，则优先采用首次调用并忽略剩下的调用
						// 实现这条需要前面加一个变量called
						if (called) return;
						called = true;
						resolvePromise(promise, y, resolve, reject);
					},
					// 如果 rejectPromise 以据因 r 为参数被调用，则以据因 r 拒绝 promise
					r => {
						if (called) return;
						called = true;
						reject(r);
					});
			} catch (error) {
				// 如果调用 then 方法抛出了异常 e：
				// 如果 resolvePromise 或 rejectPromise 已经被调用，则忽略之
				if (called) return;

				// 否则以 e 为据因拒绝 promise
				reject(error);
			}
		} else {
			// 如果 then 不是函数，以 x 为参数执行 promise
			resolve(x);
		}
	} else {
		// 如果 x 不为对象或者函数，以 x 为参数执行 promise
		resolve(x);
	}
}

MyPromise.deferred = function () {
	var result = {};
	result.promise = new MyPromise(function (resolve, reject) {
		result.resolve = resolve;
		result.reject = reject;
	});

	return result;
}
module.exports = MyPromise
```



### 13.手写数组方法

#### 手写reduce

```js
Array.prototype.myReduce=function (cb,initialValue){
	const array=this;
	let pre=initialValue||array[0]
	const startIndex=initialValue?0:1
	for(let i=startIndex;i<array.length;i++){
		const cur=array[i]
		pre=cb(pre,cur,i,array)
	}
	return pre
}
```

#### 手写forEach

forEach() 方法用于调用数组的每个元素，并将元素传递给回调函数。

```js
Array.prototype.myForEach = function (fn) {
  let context = arguments[1] || window;
  
  if (typeof fn === 'function') {
    for (let i = 0; i < this.length; i++) {
      fn.call(context, this[i], i, this)
    }
  } else {
    throw new Error('parameter1 is not a function')
  }
}
let arr = [1, 3, 5, 7]
arr.myforEach((item, i , arr) =>{
    console.log('item:' + item + 'i:' + i)
})
```

#### 手写map

```js
Array.prototype.myMap = function (callback, thisArg) {
  let length = this.length
  let res = []
  if (!Array.isArray(this)) throw new TypeError('this is not an array')
  if (typeof callback !== 'function') throw new TypeError(callback + 'is not a function')
  if (length === 0) {
    return res
  }
  for (let i = 0; i < length; i++) {
    res[i] = callback.call(thisArg, this[i], i, this)
  }
  return res
}
    let arr = [1,3,5,7]
    let newArr = arr.mymap((item)=>{
        return item*3
    })
    console.log(newArr)
```

#### 手写flat

```js
Array.prototype.myflat = function (num = 1) {
        if (!Number(num) || Number(num) < 0) {
            return this;
        }
        var arr = []
        this.forEach((item) => {
            if (Array.isArray(item)) {
                arr = arr.concat(item.flat(--num))
            } else {
                arr.push(item)
            }
        })
        return arr
    }
const arr = [1, [2, [3, 'a', [4]]]]

console.log(arr.myflat('dsdsadf'));  // [1, [2, [3, 'a', [4]]]]
console.log(arr.myflat(-32)); // [1, [2, [3, 'a', [4]]]]
console.log(arr.myflat(0));   // [1, [2, [3, 'a', [4]]]]
console.log(arr.myflat('1'));   // [1, 2, [3, 'a', [4]]]
console.log(arr.myflat('2'));    // [1, 2, 3, 'a', [4]]
console.log(arr.myflat(3));       // [1, 2, 3, 'a', 4]
console.log(arr.myflat(Infinity));     // [1, 2, 3, 'a', 4]
console.log(arr.myflat('Infinity'));   // [1, 2, 3, 'a', 4]
```

#### 实现 filter 方法

```js
Array.prototype.myFilter=function(callback, context=window){

  let len = this.length
      newArr = [],
      i=0

  for(; i < len; i++){
    if(callback.apply(context, [this[i], i , this])){
      newArr.push(this[i]);
    }
  }
  return newArr;
}
```

#### 实现 some 方法

**语法** 

```js
array.some(function(currentValue,index,arr),thisValue)
```

**参数说明**

```vb
currentValue: 必须。当前元素的值
index: 可选。当前元素的索引值
arr: 可选。当前元素属于的数组对象

thisValue: 可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。\
如果省略了 thisValue ，"this" 的值为 undefined

返回值： 布尔值。如果数组中有元素满足条件返回 true，否则返回 false。
```



```js
Array.prototype.mySome = function (fn, thisValue) {
    if (typeof fn !== 'function') {
        throw new Error(`${fn} 不是一个函数`)
    }
    if ([null, undefined].includes(this)) {
        throw new Error(`this 是null 或者 undefined`)
    }
    const arr = Object(this)
    let flag = false
    for (let i = 0; i < arr.length; i++) {
        const res = fn.call(thisValue, arr[i], i, arr) // call方法如果传null或者undefined为参数，会将this指向window
        if (res) {
            return true
        }
    }
    return flag
}
const arr = [13]
console.log(arr.myEvery((item) => item > 15, [12, 15, 45]))
```

#### 实现 every 方法

```js
Array.prototype.myEvery = function (fn, thisValue) {
    if (typeof fn !== 'function') {
        throw new Error(`${fn} 不是一个函数`)
    }
    if ([null, undefined].includes(this)) {
        throw new Error(`this 是null 或者 undefined`)
    }
    const arr = Object(this)
    let flag = true
    for (let i = 0; i < arr.length; i++) {
        const res = fn.call(thisValue, arr[i], i, arr)
        if (!res) {
            return false
        }
    }
    return flag
}
```

#### reduce 实现一个 map

```js
Array.prototype.mapByreduce=function(fn,context=null){
	let arr=this;
	if(typeof fn!=='function'){
		throw new TypeError('is not a function');
	}
	return arr.reduce((pre,cur,index,array)=>{
		let res=fn.call(context,cur,index,array);
			return [...pre,res]
	},[])
}
```

### 14.手写promisify

```js
function promisify(fn) {
    console.log(fn,"fn"); // 保存的是原始函数(add)
    return function (...args) {
        console.log(...args,"...args"); // 2 6 保存的是调用时的参数
        //返回promise对象
        return new Promise(function (resolve, reject) {
            // 将callback放到参数末尾,并执行callback函数
            args.push(function (err, ...args) {
                console.log(...args,"12"); // 2 6 callback,
                if (err) {
                    reject(err);
                    return;
                }
                resolve(...args);
            });

            fn.apply(null, args);
        });
    }
}

// 示例
let add = (a,b, callback) => {
    let result = a+b;
    if(typeof result === 'number') {
        callback(null,result)
    }else {
        callback("请输入正确数字")
    }
}

const addCall = promisify(add);
addCall(2,6).then((res) => {
    console.log(res);
})

```



手动实现一个promisify函数的意思是：我们把一个异步请求的函数，封装成一个可以具有 `then`方法的函数，并且在`then`方法中返回异步方法执行结果的这么一个函数

1. 具有 then 方法
2. then 方法里返回异步接口执行结果

```js
// 首先定一个需要进行 promisify 的函数
function asyncFn(a, b, callback) {
        // 异步操作，使用 setTimeout 模拟
        console.log('异步请求参数', a, b)
        setTimeout(function() {
                callback('异步请求结果')
        }, 3000)
}

// 我们希望调用的方式是
const proxy = promisify(asyncFn)
proxy(11,22).then(res => {
        // 此处输出异步函数执行结果
        console.log(res)
})

// 定义一个方法， 需要针对异步方法做封装，所以需要一个入参，既需要promisify的原异步方法
function promisify( asyncFn ) {
        // 方法内部我们需要调用asyncFn方法，并传递原始参数，所以需要返回一个方法来接收参数
        return function(...args) { // 由于需要接收参数，所以参数我们可以写为...args
                // 我们需要执行异步操作，并返回一个结果，所以返回一个 promise实例
                return new Promise(resolve => {
                        // asyncFn 需要执行一个回调，所以定义一个回调方法
                        const callback = function(...args) {
                              resolve(args)
                         }
                        args.push(callback)
                        asyncFn.apply(null, args)
                })
        }
}

```

```js
function promisify(fn) {
    return function (...args) {
        return new Promise(function (resolve, reject) {
            args.push(function (err, ...arg) {
                if (err) {
                    reject(err);
                    return;
                }
                resolve(...arg);
            });

            fn.apply(null, args);
        });
    }
}
```



### 15.用setTimeout实现setInterval