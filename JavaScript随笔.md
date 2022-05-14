# 函数的length

函数的length返回的值是 **参数的个数**

# Object的assign

`Object.assign(target, source.....)`

①目标对象和源对象有同名属性   /   有多个源对象： 则后面的永远覆盖前面的

②只有一个参数时：<1>参数是对象，返回对象；<2>参数不是对象，则转换为对象

③目标对象不能是undefined和null（因为这两者无法转换为对象）

# for in 和 for of

## ①for in

遍历的是**key**

**注意：不要用来遍历数组**，因为会遍历数组的原型和所有可枚举属性

**注意2：**可以遍历对象，但是也会遍历原型，所以可以在遍历的时候用hasOwnProperty判断

```javascript
for (var key in obj) {
　　if（obj.hasOwnProperty(key)){
　　　　console.log(key);
　　}
}
```

## ②for of

遍历的是**value**

**注意：**只会遍历数组内的元素，不包括数组的原型属性。所以**可以用来遍历数组**

**注意2：不能遍历对象**

# arguments

不是数组，没有数组的方法，只是类似数组。（但是有 length属性）

`Arrar.isArray(arguments) == false`

`arguments instanceof Array == false`

# 数组可以直接下标赋值

```javascript
let arr = []
arr[10] = 1
console.log(arr) // [empty * 10, 1]   未赋值的下标值是undefined
```

# 数组用toString

会将数组内的元素变成字符串拼接起来，自动去除`'['  ']'`

```javascript
let arr = [1, [2, [3, 4]]];
arr.toString() // "1,2,3,4"
```

# 浅拷贝

## ①Object.assign

```javascript
let target = {a:1}
let source1 = {b:2}
let source2 = {c:3}
Object.assign(target, source1, source2)
```

## ②...运算符

```javascript
let obj = {a:1, b:{c:2}}
let newObj = {...obj}
```

## ③slice（数组）

```javascript
let arr = [1,2,3,4]
let newArr = arr.slice() // arr.slice(0)效果一样
```

## ④concat（数组浅拷贝）

```javascript
let arr = [1,2,3,4]
let newArr = arr.concat()
```

## ⑤手写（遍历）

```javascript
function shallowClone(object) {
    if (!object || typeof object !== 'object') {
        return
    }
    let newObj = Array.isArray(object) ? [] : {}
    for (let key in object) {
        if (object.hasOwnProperty(key)) {
            newObj[key] = object[key]
        }
    }
    return newObj
}
```

# 深拷贝

## ①JSON.parse 和 JSON.stringify

存在的问题：当obj的属性有 函数、symbol、undefined时，JSON.stringify处理之后会消失

```javascript
let obj =  = {  a: 0,
              b: {
                  c: 0
              }
             };
let newObj = JSON.parse(JSON.stringify(obj))
```

## ②lodash库

```javascript
import cloneDeep from 'lodash/cloneDeep'
let obj = {......}
let newObj = cloneDeep(obj)
```

## ③手写（遍历+判断）

```javascript
function deepClone(object) {
    if (!object || typeof object !== 'object') {
        return
    }
    let newObj = Array.isArray(object) ? [] : {}
    for (let key in object) {
        if (object.hasOwnProperty(key)) {
            newObj[key] = typeof object[key] === 'object' ? deepClone(object[key]) : object[key]
        }
    }
    return newObj
}
```

# 手写call、apply、bind

## ①call

```javascript
Function.prototype.myCall = function (context) {
    if (typeof this !== 'function') return

    context = context || window
    context.fn = this
    let res = null
    let args = [...arguments].slice(1)
    res = context.fn(...args)
    delete context.fn
    return res
}
```

## ②apply

```javascript
Function.prototype.myApply = function(context) {
    if (typeof this !== 'function') return 
    context = context || window
    context.fn = this
    let res = null
    if (arguments[1]) {
        let args = arguments[1]
        res = context.fn(...args)
    } else {
        res = context.fn()
    }
    return res
}
```

## ③bind

利用**call**

```javascript
Function.prototype.myBind = function(context) {
    if (typeof this !== 'function') return
    context = context || window
    let that = this
    let args = [...arguments].slice(1)
    return function() {
        that.call(context, ...args)
    }
}
```

利用**apply**

```javascript
Function.prototype.myBind = function(context) {
    if (typeof this !== 'function') return
    context = context || window
    let that = this
    let args = [...arguments].slice(1)
    return function() {
        that.apply(context, args)
    }
}
```

# 数组扁平化

## ①递归

```javascript
function flatten(arr) {
    let res = []
    for (let i = 0; i < arr.length; ++i) {
        if (Array.isArray(arr[i])) {
            res = res.concat(flatten(arr[i]))
        } else{
            res.push(arr[i])
        }
    }
    return res
}
let arr = [1, [2, [3, 4]]]
arr = flatten(arr)
```

## ②...运算符

```javascript
function flatten(arr) {
    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr)
    }
    return arr
}
```

## ③toString + split

```javascript
function flatten(arr) {
    return arr.toString().split(',');
}
```

## ④ES6的flat

```javascript
function flatten(arr) {
    return arr.flat(Infinity) // 默认是1，即只拉一层，填Infinity就是不论维度全部拉成一维
}
```

# 手写push

```javascript
Array.prototype.push = function() {
    for (let val of arguments) {
        this[this.length] = val
    }
}
```

# 手写filter

```javascript
Array.prototype.filter = function(fn) {
    if (typeof fn !== 'function') {
        throw Error('error')
    }
    let res = []
    for (let i = 0; i < this.length; ++i) {
        if (fn(this[i])) res.push(this[i])
    }
    return res
}
```

# 手写map

```javascript
Array.prototype.map = function (fn) {
    let res = []
    if (typeof fn !== 'function') {
        throw Error('fuck')
    }
    for (let val of this) {
        res.push(fn(val))
    }
    return res
}
```

# 类数组转数组

①slice

`Array.prototype.slice.call(arguments)`

②splice

`Array.prototype.splice.call(arguments, 0)`

③Array.from

`Array.from(arguments)`

④concat

`Array.prototype.concat.apply([], arguments)`

⑤...操作符

`[...arguments]  `

# Promise

## **Promise的状态在发生变化之后，就不会再发生变化**

```javascript
const promise = new Promise((resolve, reject) => {
    resolve('success1');
    reject('error');
    resolve('success2');
});
promise.then((res) => {
    console.log('then:', res);
}).catch((err) => {
    console.log('catch:', err);
})

// 输出then：success1
```

## Promise.resolve传递的参数

Promise.resolve方法的参数如果是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为resolved，Promise.resolve方法的参数，会同时传给回调函数。

then方法接受的参数是函数，而如果传递的并非是一个函数，它实际上会将其解释为then(null)，这就会导致前一个Promise的结果会传递下面。

```javascript
Promise.resolve(1)
    .then(2)
    .then(Promise.resolve(3))
    .then(console.log)

// 输出
// 1
// Promise {<fulfilled>: undefined}
```

一个原则：`.then` 或`.catch` 的参数期望是函数，传入非函数则会发生**值透传**。

## then返回非promise

**返回任意一个非 promise 的值都会被包裹成 promise 对象**

①

`return 2`会被包装成`resolve(2)`

```javascript
Promise.resolve(1)
    .then(res => {
    console.log(res);
    return 2;
})
    .catch(err => {
    return 3;
})
    .then(res => {
    console.log(res);
});

// 输出
// 1
// 2
```

②

`return new Error('error!!!')`也被包裹成了`return Promise.resolve(new Error('error!!!'))`

```javascript
Promise.resolve().then(() => {
    return new Error('error!!!')
}).then(res => {
    console.log("then: ", res)
}).catch(err => {
    console.log("catch: ", err)
})
// 输出："then: " "Error: error!!!"
```

## 失败函数和catch函数捕获

①直接走了失败函数

```javascript
Promise.reject('err!!!')
    .then((res) => {
    console.log('success', res)
}, (err) => {
    console.log('error', err)
}).catch(err => {
    console.log('catch', err)
})
// 输出：error err   
```

②走catch函数

```javascript
Promise.resolve()
    .then(function success (res) {
    throw new Error('error!!!')
}, function fail1 (err) {
    console.log('fail1', err)
}).catch(function fail2 (err) {
    console.log('fail2', err)
})
```

## 已经resolve后再resolve

如果Promise已经之前resolve了，后面再调用resolve不会执行

## 没有resolve的promise

会永远处于pending状态，await 这个promise则无法进行下去，导致这个await后面的代码永远不会执行

## throw error

只要throw了错误，就会直接被catch捕获（即直接跳转到catch语句处）。

只要没有throw 错误，就会继续then， 如

```javascript
Promise.resolve().then(() => {
    console.log('1');
    throw 'Error';
    // 直接跳到第一个catch
}).then(() => {
    console.log('2');
}).catch(() => {
    console.log('3');
    throw 'Error';
    // 跳到第二个catch
}).then(() => {
    console.log('4');
}).catch(() => {
    console.log('5');
    //没有throw error了，所以继续下一个then
}).then(() => {
    console.log('6');
});
```

## 多个setTimeout

此时不再按入队顺序执行，而是按定时器设定的时间执行。

# async await

## await后面的语句

await后面的语句可以当做new Promise，会立即执行。await下一行开始的语句相当于Promise.then，会加入微任务队列（所以函数体执行完await会立即跳出执行同步代码）

# 匿名函数

匿名函数的this指向window

**注意：**虽然this指向window，但是在**其中声明的变量仍然属于局部变量**，即使用var声明。除非不用声明符直接声明全局变量。

# call函数不指定参数

如果是`xx.call()`括号内不指定参数，则默认this指向window

# this绑定优先级

**new绑定 > 显式绑定 > 隐式绑定 > 默认绑定。**

默认绑定：在非严格模式下是指向的全局，在严格模式下是undefined。

隐式绑定：是指谁调用了函数，函数的指向就是谁，如果存在多个调用的话，this就是指向最近的一个调用。

显示绑定：通过call apply bind方法改变this的行为就是显示绑定。

new绑定：通过构造函数 new来调用的普通函数。

# 全局变量

## ①使用var

在**方法内**部使用var声明变量是**局部**变量；在**方法外**部声明变量是**全局**变量

## ②不使用var

不管在 方法内还是方法外，**都是全局**变量。

**注意：**在方法**内部声明的全局变量**，需要**先调用一下方法**才会生效。否则仍然读不到全局变量。

## 污染全局属性

声明的**全局变量**和**全局函数**会变成window上的属性，也即

```javascript
var a = 1
// window.a = 1
b = 2
// window.b = 2
function haha(){}
// window.haha = f(){}
var haha = function(){}		// 同上
ha = function(){}			// 同上
```

但是let、const、class等声明，**即使在全局作用域下声明的变量或函数，也不会变为顶层对象（window）的属性**

# 声明提升

## ①var声明

var的声明会提升到顶部，但是赋值不会提升

## ②函数声明

函数的声明有两种：

`function haha() {console.log('haha')}`这种声明会直接全部提升到顶部（因为不存在赋值这一说）

`var a = function() {console.log('haha')}`这种声明还是属于var声明提升的范畴

**注意：**

①声明提升到顶部的顺序是按声明的顺序来的，如下：

后声明的函数a会覆盖之前声明的函数a

```javascript
function a() {
    var temp = 10;
    function b() {
        console.log(temp); // 10
    }
    b();
}
function a() {
    temp = 10;
    b();
}
```

②全局变量声明，**没有提升**

# 原型

## prototype上的方法

```javascript
function Foo() {
    Foo.a = function () {
        console.log(1);
    }
    this.a = function () {
        console.log(2)
    }
}

Foo.prototype.a = function () {
    console.log(3);
}
```

prototype上绑定的方法**不会覆盖**构造函数中声明的内部方法。这两个方法会**同时存在**。

# 模块化

 ①ES6不能动态导入，CommonJS可以动态导入

```javascript
require(`${path}/xx.js`)
```

②ES6是异步导入，CommonJS是同步导入

# 防抖

## 普通版

```javascript
function debounce(func, wait) {
    let timer = null
    return function () {
        if (timer) clearTimeout(timer)
        timer = setTimeout(() => {
            func.apply(this, arguments)
        }, wait);
    }
}
```

## 带立即执行版

```javascript
function debounce(func, wait, immediate = true) {
    let timer = null
    return function () {
        if (timer) clearTimeout(timer)
        if (immediate) {
            if (!timer) func.apply(this, arguments)
            timer = setTimeout(() => {
                timer = null
            }, wait);
        }
        else {
            timer = setTimeout(() => {
                func.apply(this, arguments)
            }, wait);
        }
    }
}
```

**绑定：**

`content.onmousemove = debounce(count, 1000);`

**注意：**

①绑定时是执行了debounce函数，该函数会返回一个函数并声明一个变量timer。也就是页面刚初始化就会执行上述操作，即使没触发事件，所以从始至终只会有一个timer变量

②debounce函数执行后返回一个函数，事件绑定的是这个函数。

# 节流

## 时间戳版：（立即调用）

```javascript
function throttle(func, wait) {
    let previous = 0
    return function() {
        let now = Date.now()
        if(now - previous > wait) {
            func.apply(this, arguments)
            previous = now
        }
    }
}
```

## 定时器版：（延迟调用）

```javascript
function throttle(func, wait) {
    let timer = null
    return function() {
        if(!timer) {
            timer = setTimeout(() => {
                timer = null
                func.apply(this,arguments)
            }, wait);
        }
    }
}
```

# 让0.1 + 0.2 == 0.3

```javascript
function isEqual(a, b) {
    return Math.abs(a - b) < Number.EPSILON;
}

console.log(isEqual(0.1 + 0.2, 0.3)); // true
```

# for循环独特之处

设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

# Object.create

`Object.create()`方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。

# 继承

## ---构造函数的继承---

## ①原型链继承

```javascript
function Parent() {
    this.name = 'parent'
    this.play = [1, 2, 3]
}
Parent.prototype.getName = function () {
    return this.name
}
function Child() {
    this.type = 'child'
}
Child.prototype = new Parent()   // 继承在这里
```

缺点：多个实例使用的是**同一个原型对象**。导致如果原型对象中有引用型数据，则一个改了，另一个也改

## ②构造函数继承（借助call）

```javascript
function Parent() {
    this.name = 'parent'
    this.play = [1, 2, 3]
}
Parent.prototype.getName = function () {
    return this.name
}
function Child() {
    Parent.call(this)  // 继承在这里
    this.type = 'child'
}
```

优点：解决了**原型链继承中引用型数据内存共享**的问题

缺点：父类**原型对象上定义的方法，子类拿不到**

## ③组合继承（组合以上两种）

```javascript
function Parent() {
    this.name = 'parent'
    this.play = [1, 2, 3]
}
Parent.prototype.getName = function () {
    return this.name
}
function Child() {
    Parent.call(this)		// 继承1
    this.type = 'child'
}
Child.prototype = new Parent() 			// 继承2
Child.prototype.constructor = Child		// 继承2
```

优点：解决了①和②中的缺点，即：<1>解决父类**引用型数据内存共享**问题；<2>解决**父类原型对象定义的方法**子类拿不到问题

## ---普通对象的继承---

## ④原型式继承

```javascript
let parent4 = {
    name: "parent4",
    friends: ["p1", "p2", "p3"],
    getName: function() {
        return this.name;
    }
};
```

优点：父类的**属性和方法都能继承**

缺点：存在父类**引用型数据内存共享问题**

## ⑤寄生式继承

```javascript
let parent5 = {
    name: "parent5",
    friends: ["p1", "p2", "p3"],
    getName: function() {
        return this.name;
    }
};
function clone(original) {
    let clone = Object.create(original);
    clone.getFriends = function() {
        return this.friends
    };
    return clone;
}
let person5 = clone(parent5);
```

优缺点：和④一模一样，只是能**额外添加方法**。

## ⑥继承的最终版本

```javascript
function clone (parent, child) {
    // 这里改用 Object.create 就可以减少组合继承中多进行一次构造的过程
    child.prototype = Object.create(parent.prototype);		// 3继承 + 4继承
    child.prototype.constructor = child;					// 3继承
}

function Parent6() {
    this.name = 'parent6';
    this.play = [1, 2, 3];
}
Parent6.prototype.getName = function () {
    return this.name;
}
function Child6() {
    Parent6.call(this);				// 3继承
    this.friends = 'child5';
}
clone(Parent6, Child6);
Child6.prototype.getFriends = function () {
    return this.friends;
}
let person6 = new Child6();
```

## ⑦ES6---extends继承

```javascript
class Person {
    constructor(name) {
        this.name = name
    }
    // 原型方法
    // 即 Person.prototype.getName = function() { }
    // 下面可以简写为 getName() {...}
    getName = function () {
        console.log('Person:', this.name)
    }
    // 简写形式
    // getName() {
    //     console.log('Person:', this.name)
    // }
}

class Gamer extends Person {
    constructor(name, age) {
        // 子类中存在构造函数，则需要在使用“this”之前首先调用 super()。
        super(name)
        this.age = age
    }
}

const asuna = new Gamer('Asuna', 20)
asuna.getName() // 成功访问到父类的方法
```

# NaN

`NaN == NaN  // false`

用`Number.isNaN(num)`

