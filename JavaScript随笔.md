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

**await后面的语句可以当做new Promise，会立即执行**。**await下一行开始的语句相当于Promise.then，会加入微任务队列**（所以函数体**执行完await会立即跳出执行同步代码**）

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

子构造函数将原型指向父构造函数的一个实例

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

缺点：多个实例使用的是**同一个原型对象**。导致如果原型对象中**有引用型数据，则一个改了，另一个也改**

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
let child = Object.create(parent4)
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

# 栈和堆

栈：会**自动分配内存空间**，会**自动释放**。**存取速度快，但不灵活**，同时由于结构简单，在变量使用完成后就可以将其释放，内存回收容易实现。

堆：**动态分配的内存**，大小不定也**不会自动释放**。使用**灵活**，可以动态增加或删除空间，但是**存取比较慢**

基本类型：简单的数据段，存放在栈内存中，占据固大小的空间。

引用类型：对象等引用数据，保存在堆内存中，此时栈内存中保存的是引用数据的地址（即引用还是在栈中）

# ES6新

## 特性

箭头函数

Symbol

bigInt

解构赋值

...运算符

Map，WeakMap

Set，WeakSet

class

let，const

模板字符串

import，export

对象字面量缩写：即属性名和属性值一样时可以只写属性名(vue中很常用)

函数参数默认值

## 方法

includes

Array.from

Array.of

find

Object.assign

Object.keys， Object.values， Object.entries

Promise

# 事件委托（事件代理）

事件委托就是把原本需要绑定在子元素上面的响应事件委托给它们的父元素或者更外层元素

1.**实现原理：`事件委托是利用事件的冒泡原理来实现的`**，大致可以分为三个步骤：

①**确定**要添加事件元素的**父级**元素；

②**给父元素定义事件**，监听子元素的冒泡事件；

③使用 **`event.target`** 来**定位**触发事件冒泡的**子元素**。

注意：使用事件委托时，并不是说把事件委托给随意一个父元素就行。因为事件冒泡的过程也需要消耗时间，距离越远，所需的时间也就越长，所有**最好在直接父元素上使用事件委托**。

2.**优点：**

1>减少内存消耗

2>方便动态绑定事件：有时我们需要动态增加或移除页面中的元素，比如上面示例中动态的在 ul 标签中添加 li 标签，如果不使用事件委托，则需要手动为新增的元素绑定事件，同时为删除的元素解绑事件。而使用事件委托就没有这么麻烦了，无论是增加还是减少 ul 标签中的 li 标签，即不需要再为新增的元素绑定事件，也不需要为删除的元素解绑事件。

# 箭头函数

## 1.与普通函数的区别

①写法更简洁

②没有自己的this

③不能用call，apply，bind改变this指向

④this指向永远不会改变

⑤不能new一个箭头函数

⑥没有prototype

⑦没有arguments

## 2.this指向

就是定义时所在的对象

# 闭包

## 1.概念

闭包就是**一个函数**，这个函数**能访问另一个函数作用域中的变量**

## 2.创建方式

在一个A函数内部创建一个函数B，这个函数B能够访问函数A内部的局部变量

## 3.常见用途

循环中使用闭包解决var定义函数的问题

```javascript
for (var i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i)
    }, i * 1000)
}
// 解决
for (var i = 1; i <= 5; i++) {
    ;(function(j){
        setTimeout(function timer() {
            console.log(j)
        }, j * 1000)
    })(i)
}
```

# 事件循环

所有同步任务都在主线程上执行，形成一个执行栈。

当栈中的代码执行完毕，执行栈中的任务为空时，主线程会**先检查micro-task(微任务)队列中是否有任务**，如果**有，就将micro-task(微任务)队列中的所有任务依次执行**，直到micro-task(微任务)队列为空; **之后再检查macro-task(宏任务)队列中是否有**任务，如果**有，则**取出**第一个**macro-task(**宏任务**)**加入到执行栈**中，之后**再清空执行栈，检查micro-task(微任务)**，以此循环，直到全部的任务都执行完成。

**注意：**本次同步任务执行完毕，先清空微任务队列，然后再看宏任务队列，有的话就取**第一个**出来执行完再检查微任务队列，而**不是一次性把所有宏任务执行完**，因为最开始执行栈执行完后宏任务队列中可能有多个宏任务。

## 宏任务有哪些

`script`

`setTimeout`

`setInterval`

## 微任务有哪些

`Promise`

`Async，Await`

`process.nextTick`

# 原型

在js中使用构造函数来创建一个对象。

构造函数有一个prototype属性，并且只有函数有prototype属性。

这个prototype属性值是一个对象，对象中包含了可以由该构造函数创建的所有实例共享的属性和方法。

当使用构造函数创建一个实例后，实例中会有一个指针，该指针指向构造函数的prototype属性对应的值，这个指针可以叫做对象的原型，目前在浏览器中可以通过`__proto__`属性来访问。不过在ES6中提供了`Object.getPrototypeOf`方法获取对象的原型。

## 原型链

当一个对象访问的属性或方法不在自己身上时，会去原型对象上找这个属性或方法，而这个原型对象也有自己的原型对象，顺着这个链条找下去就是所说的原型链。原型链的尽头是null

# 作用域

## 1.全局作用域

①最外层函数，最外层函数外定义的变量  都具有全局作用域。即**最外层声明的函数具有全局作用域**，**最外层声明的变量有全局作用域**

②没有声明符号声明的变量具有全局作用域，是个全局变量

③window对象的属性拥有全局作用域

## 2.函数作用域

函数内声明的变量具有函数作用域

## 3.块级作用域

ES6新增，let和const创建的变量

## 4.作用域链

**Tips：**凡是跨了自己的作用域的变量都叫自由变量

**①解释**

在当前作用域中查找所需变量，如果在自己的作用域没有找到变量，就往上层作用域找，直到找到window对象停止。这一层层的关系就叫作用域链

**②作用**

**保证对执行环境有权访问的所有变量和函数的有序访问，通过作用域链，可以访问到外层环境的变量和函数。**

**③本质**

指向变量对象的指针列表。变量对象是一个包含了执行环境中所有变量和函数的对象。作用域链的前端始终都是当前执行上下文的变量对象。全局执行上下文的变量对象（也就是全局对象）始终是作用域链的最后一个对象。

当查找一个变量时，如果当前执行环境中没有找到，可以沿着作用域链向后查找。

# 执行上下文

## 1.执行上下文类型

**①全局执行上下文**

不在函数内部的都是全局执行上下文，一个程序只有一个

**②函数执行上下文**

函数调用时就会创建一个新的执行上下文，函数的上下文可以有很多个

**③eval函数执行上下文**

执行eval中的代码会有一个上下文

## 2.执行上下文栈

- JavaScript引擎使用执行上下文栈来管理执行上下文
- 当JavaScript执行代码时，首先遇到全局代码，会创建一个全局执行上下文并且压入执行栈中，每当遇到一个函数调用，就会为该函数创建一个新的执行上下文并压入栈顶，引擎会执行位于执行上下文栈顶的函数，当函数执行完成之后，执行上下文从栈中弹出，继续执行下一个上下文。当所有的代码都执行完毕之后，从栈中弹出全局执行上下文

## 3.创建执行上下文

①this绑定

- 在全局执行上下文中，**this指向全局对象**（window对象）
- 在函数执行上下文中，this指向取决于函数如何调用。如果它**被一个引用对象调用**，那么 **this 会被设置成那个对象**，**否则** this 的值被设置为**全局对象或者 undefined**

②创建词法环境组件

③创建变量环境组件

④完成变量的分配，再执行代码

## 4.总结

在执行一点JS代码之前，需要先解析代码。解析的时候会先创建一个全局执行上下文环境，先把代码中即将执行的变量、函数声明都拿出来，变量先赋值为undefined，函数先声明好可使用。这一步执行完了，才开始正式的执行程序。

在一个函数执行之前，也会创建一个函数执行上下文环境，跟全局执行上下文类似，不过函数执行上下文会多出this、arguments和函数的参数。

# 垃圾回收

## 1.标记清除

- 垃圾收集器在运行时会给内存中所有的变量都加上一个标记，假设内存中所有的对象全部是垃圾，全部标记为0
- 然后从各个根对象开始遍历，把不是垃圾的节点改成1
- 清理所有标记为0的垃圾，销毁并回收它们所占用的内存空间
- 最后把所有内存中对象标记修改为0，等待下一轮的垃圾回收

在清除垃圾之后，剩余对象的内存位置是不变的，就会导致空闲内存空间不连续。这样就出现了内存碎片，并且由于剩余空间不是整块，就需要内存分配的问题。

## 2.引用计数清除

- 跟踪记录每个变量值被使用的次数
- 当声明一个变量并且将一个引用类型数据赋值给这个变量的时候，这个引用类型数据的引用次数就标记为 1
- 如果当这个引用类型数据又赋值给另一个变量，那么引用次数就+1
- 如果变量被其他的值覆盖（即赋值给的某个变量被重新赋值为其他的值，不再是这个引用类型数据了），则引用次数-1
- 当这个引用类型数据的引用次数变为0的时候,这个变量就没有被使用了，也无法访问，垃圾回收器就会在执行时，销毁引用次数为0的引用类型数据，回收其所占用的内存空间。

**缺点：**无法回收循环引用的对象

## 3.减少垃圾回收

- **对数组进行优化：**在清空一个数组时，可以将数组的**长度设置为0**，以此来达到清空数组的目的。
- **对`object`进行优化：**对象**尽量复用**，对于不再使用的对象，就将其**设置为null**，尽快被回收。
- **对函数进行优化：**在循环中的函数表达式，如果可以复用，尽量放在函数的外面。

## 4.内存泄露

①全局变量

②定时器

③获取dom的引用，后来这个dom被删除，但是引用仍存在

④闭包
