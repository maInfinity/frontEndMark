# v-for和v-if

不能连用，因为 `v-for` 优先级高于 `v-if`。导致会先循环出来再`v-if`判断。

用**computed**，提前将要循环的数组按条件筛选，再`v-for`遍历

# key的作用

## 虚拟DOM中的key

key是虚拟DOM对象的标识，当**数据发生变化**时，Vue会根据**【新数据】生成【新的虚拟DOM】**, 

随后Vue进行**【新虚拟DOM】与【旧虚拟DOM】的差异比较**，比较规则如下：

### 	对比规则

(1).旧虚拟DOM中**找到了**与新虚拟DOM**相同的key:**

​	①.若虚拟DOM中**内容没变, 直接使用之前的真实DOM**！

​	②.若虚拟DOM中**内容变了, 则生成新的真实DOM**，随后替换掉页面中之前的真实DOM。

(2).旧虚拟DOM中**未找到**与新虚拟DOM**相同的key:**

​		**创建新的真实DOM，随后渲染到到页面**。

### 	用index作为key可能会引发的问题

①若对数据进行：逆序添加、逆序删除等破坏顺序操作:

（意思就是如果页面只是文字等展示类元素，那么虽然表面看没影响，但是效率低下）

​	会产生没有必要的真实DOM更新 ==> 界面效果没问题, 但效率低。

②如果结构中还包含输入类的DOM：

​	会产生错误DOM更新 ==> 界面有问题。

## 作用

### 	①提高效率

**为了在diff算法执行时更快的找到对应的节点，`提高diff速度，更高效的更新虚拟DOM`;**

在vue的diff函数中，会根据新节点的key去对比旧节点数组中的key，从而找到相应旧节点。如果没找到就认为是

一个新增节点。而如果**没有key，那么就会采用遍历查找的方式去找到对应的旧节点。**

### 	②顺序变更导致渲染错误

**为了在数据变化时强制更新组件，以避免`“就地复用”`带来的副作用。**

当 Vue 用 `v-for` 更新已渲染过的元素列表时，它默认用“就地复用”策略。如果**数据项的顺序被改变**，Vue 将不

会移动 DOM 元素来匹配数据项的顺序，而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的

每个元素。重复的key会造成**渲染错误**。

# 组件通信

## 1.父子通信

### ①props / $emit

props：父 → 子

`$on`  / `$emit` ： 子 → 父

### ②parent / children

使用`$parent`可以让组件访问父组件的实例（访问的是上一级父组件的属性和方法），可以使用`$root`来访问根组件的实例

使用`$children`可以让组件访问子组件的实例，但是，`$children`并不能保证顺序，并且访问的数据也不是响应式的。

`$children` 的值是**数组**，数组的每一项是对象；而`$parent`是个**对象**

### ③ref

给子组件打ref，**可以调用子组件的方法**

### ④provide、inject：父→子（祖→孙也可以）

```javascript
// 父组件
provide() {
 return {
    num: this.num
  };
}
// 子组件
inject: ['num']

// 访问父组件所有属性
// 父
provide() {
 return {
    app: this
  };
}
data() {
 return {
    num: 1
  };
}
// 子
inject: ['app']
console.log(this.app.num)
```

## 2.任意通信

### ①全局事件总线（万能）

唯一缺点：如果项目过大，使用这种方式进行通信，后期维护起来会很困难。

### ②消息发布订阅（React）

### ③Vuex（万能）

# Vue DOM更新机制和$nextTick

## 1.原理

①DOM更新机制

Vue 在数据变化之后，视图不会立刻更新，而是等同一事件循环中所有的数据变化完成之后再进行统一是数据更新。即**vue是异步执行DOM更新的。**

②$nextTick

使用 $nextTick 之后，获取 DOM 值的代码会在 DOM 重新渲染结束之后执行

## 2.使用场景

- 在数据变化后执行的某个操作，而这个操作需要**使用随数据变化而变化的DOM结构**的时候，这个操作就需要方法在`nextTick()`的回调函数中。**也就是说数据修改后立即执行的方法要放入`nextTick`**
- 在vue生命周期中，如果在created()钩子进行DOM操作，也一定要放在`nextTick()`的回调函数中。因为在**created()钩子函数中，页面的DOM还未渲染**，这时候也没办法操作DOM，所以，**此时如果想要操作DOM，必须将操作的代码放在`nextTick()`的回调函数中。**

# keep-alive

## 1.原理

`Vue`内部将`DOM`节点抽象成了一个个的`VNode`节点，`keep-alive`组件的缓存也是基于`VNode`节点的。它将满足条件的组件在`cache`对象中缓存起来，在需要重新渲染的时候再将`vnode`节点从`cache`对象中取出并渲染。

用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 `deactivated` 钩子函数，命中缓存渲染后会执行 `activated` 钩子函数。

**注意：**keep-alive包裹的组件**不再具有destoryed和beforeDestory钩子**

## 2.使用场景

如果需要在组件切换的时候，保存一些组件的状态防止多次渲染，就可以使用 keep-alive 组件包裹需要保存的组件。

# v-model原理

## 1.普通标签

```javascript
<input v-model="sth" />  //这一行等于下一行
<input v-bind:value="sth" v-on:input="sth = $event.target.value" />
```

## 2.组件标签

```javascript
<currency-input v-model="price"></currentcy-input>
<!--上行代码是下行的语法糖d
:value绑定的是名为'value'的动态属性，子组件用props接收
@input绑定的是名为'input'的自定义事件，子组件用$emit调用
<currency-input :value="price" @input="price = arguments[0]"></currency-input>
    -->

<!-- 子组件定义 -->
<span>
    <input
ref="input"
:value="value"
@input="$emit('input', $event.target.value)"
>
    </span>
{
    xxxx
    props: ['value'],
})
```

# Vue事件绑定原理

## 1.原生事件

通过 addEventListener 绑定给真实元素

## 2.组件事件

通过 Vue 自定义的$on 实现的。如果要在组件上使用原生事件，需要加.native 修饰符，这样就相当于在父组件中把子组件当做普通 html 标签，然后加上原生事件。

# Vue性能优化

## 1.编码

- **对象层级**不要过深，否则性能就会差
- 尽量减少**data**中的数据
- **v-if 和 v-show** 区分使用场景
- v-for给每项元素绑定事件时，使用**事件代理**
- v-for 遍历**必须加 key**，key 最好是 id 值，且避免同时使用 v-if（Vue3中v-if优先级高于v-for了，应该可以用）
- 大数据列表和表格性能优化-虚拟列表/虚拟表格
- 防止内部泄漏，组件销毁后把全局变量和**事件销毁**：因为Vue 组件销毁时，会自动解绑它的全部指令及事件监听器，但是仅限于组件本身的事件。举例可以在beforeDestory中销毁定时器
- 图片**懒加载**
- 路由**懒加载**
- 第三方插件的**按需引入**
- 适当采用 **keep-alive** 缓存组件
- **防抖、节流**运用
- 使用**iconfont代替图片**图标：因为iconfont一般是SVG图，不会失真，而且生成的文件特别小

## 2.SEO优化

- 服务端渲染 SSR
- 预渲染

## 3.打包优化

- 打包文件中去掉map文件，找到config文件夹下index.js文件,把这个build里面的productionSourceMap改成false即可
- CDN引入第三方库

- JavaScript：UglifyPlugin
- CSS ：MiniCssExtractPlugin
- HTML：HtmlWebpackPlugin
- gzip压缩

# Vue响应式原理

## 1.**原理：**

通过**数据劫持结合发布者-订阅者模式**。具体的说，通过`Object.defineProperty`来**劫持所有属性的setter和getter**，在**数据变动的时候发布消息通知订阅者**，触发相应的**监听回调**。

## **2.总结版：**

**Observer**负责将数据转换成**getter/setter**形式；
**Dep**负责**管理数据的依赖**列表；是一个发布订阅模式，**上游对接Observer，下游对接Watcher**
**Watcher**是实际上的数据依赖，负责**将数据的变化转发到外界(渲染、回调)**；
首先将**data传入Observer转成getter/setter**形式；当**Watcher实例读取数据**时，会**触发getter**，被**收集到Dep仓库**中；当**数据更新**时，**触发setter**，**通知Dep仓库中的所有Watcher实例更新**，Watcher实例负责通知外界



**Dep和Watcher**：**Dep**是负责**存放**数据所绑定所有的**观察者的**对象的**容器**，只要数据发生改变，就会通过这个**Dep**来**通知**所有**观察者**进行修改数据。（每个数据都有独一无二的**Dep**）。而**Watcher**更偏向于一个动作，也就是规定的业务逻辑或者渲染函数，是一个执行者。

## **3.详细版：**

vue会将组件中所有的**data中的数据的每个属性通过Objcet.definProperty()将他们转换为getter和setter，并在内部追踪依赖**。

在vue中还有⼀个用来**记录这些被依赖的值的对象叫做watcher**，当我们**getter时vue会去watcher拿到数据并返回**，当我们**setter时，它会去通知watcher我们这个值变了**，然后新值是什么

**watcher接到通知修改值后，会去重新调用组件的render函数**

组件的**render函数会根据watcher的数据进行生成new virtual DOM tree（新虚拟dom树），然后vue丢掉旧的虚拟dom树，将新的虚拟dom树替换上去**，然后**虚拟dom树生成相对应的真实dom**，最终**页面重新渲染**完成，当然了**vue只会替换掉更改了的那部分虚拟dom树，其他的虚拟dom树不会变**，所以vue的效率比直接更改真实dom的效率高

<img src="https://s1.ax1x.com/2022/05/12/OD9a4S.png" alt="img" style="zoom: 67%;" />

# data和computed中的数据的区别

1.data属性的值，**不会随赋值变量的改动而改动**。如果要改变这个属性的值，则需要直接给data属性赋值，视图上对这个属性的显示才会变。

2.computed属性，属于**持续变化跟踪**。在computed属性定义的时候，这个computed属性就与给它赋值的变量绑定了。**改变这个赋值变量，computed属性值会随之改变**。

# computed和watch

①缓存：computed支持缓存，watch不支持缓存

②异步：computed不能处理异步，watch可以处理

③return：computed需要return，watch不需要

④初始化：computed默认第一次加载就会获取，watch除非指定immediate否则第一次加载不会监听

总结：

**computed——**有缓存，依赖其他属性值，仅当依赖的属性变化时computed才会变化

**watch——**无缓存，主要是监视作用

# 生命周期

## 1.beforeCreate

初始化还未开始，data，methods，watch，computed访问不到

## 2.created

初始化完成，data，methods，watch，computed能访问到，但此时节点还没挂载到dom上

## 3.beforeMount

render函数首次调用，完成的操作是——编译模板，data里面的数据和模板生成了html，但html还没挂载到页面

## 4.mounted

html渲染到页面

## 5.beforeUpdate

响应式数据更新时调用，此时数据已更新，但真实DOM还没渲染

## 6.updated

组件DOM已经更新

## 7.beforeDestory

实例销毁前触发，但此时this仍然能访问实例

## 8.destoryed

实例销毁

## 特殊的keep-alive声明周期

activated和deactivated生命周期 是 keep-alive独有的。

用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 `deactivated` 钩子函数，命中缓存渲染后会执行 `activated` 钩子函数。

**注意：**keep-alive包裹的组件**不再具有destoryed和beforeDestory钩子**

## 哪个周期异步请求

**created**， beforeMount， mounted里发请求，因为此时data已能访问

**最好：**在created周期发异步请求，原因是：

①能更快获取到服务端数据，减少页面加载时间，用户体验更好

②SSR不支持后两个钩子

# Vuex原理

每一个 Vuex 应用的**核心就是 store**（仓库）。“store” 基本上就是一个容器，它包含着你的应用中大部分的状态 ( state )。

- Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
- 改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样可以方便地跟踪每一个状态的变化。

# Vuex数据持久化

## 1.用localStorage存

```javascript
state:{
    xxx:localStorage.getItem('key')
}
mutations:{
    SETXXX(state) {
        state.xxx = state.xxx + ...... // 修改数据
        localStorage.setItem('key', state.xxx)
    }
}
```

## 2.安装插件

如`vuex-persist`

# Vue-router

## 1.前端路由和后端路由

前端路由的核心，就在于 —— **改变视图的同时不会向后端发出请求，不刷新页面，前后端分离。**

后端路由：是**url**地址**映射**到**服务器**上的某些资源

前端路由：是**url**地址**映射**到**浏览器**上的某些资源

**前端路由的优点：**

①单页面只有一个页面,在**第一次加载**时,就已经将**所有资源从服务下载**

②在通过前端路由切换页面时,不是像服务器发送的请求,我们只是**通过url决定哪些资源显示**

③因为不用向服务器发送请求,所以**请求/响应造成的等待时间就会大大减少**,提高了响应速度.

**前端路由的缺点：**

①**不利于SEO优化**(单页面应用,只有一个页面会被百度收录,其他的页面都是虚拟的)

②使用浏览器的**前进后退键的时候会重新发请求**,没有合理的利用缓存

③单页面**无法记住之前滚动条的位置**,无法在前进,后退的时候记住滚动的位置

## 2.hash模式和history模式

### ①hash模式

**形式：**`http:www.xxx.com/#/haha`，`#/haha`就是hash值

**特点：**hash值会出现在URL中，但不会出现在http中，即**不会向后端发请求**，所以改变hash值不会重新加载页面，对浏览器的支持也比较好；hash值变化对应的URL也会被记录，所以浏览器可以前进和后退

### ②history模式

**形式：**`http:www.xxx.com/haha/nihao`

**特点：**比较好看，不怕前进不怕后退，但是很怕刷新，后端如果没配置就会报错

**API：**利用新的API来实现前端路由

**修改历史状态**：包括了 HTML5 History Interface 中新增的 `pushState()` 和 `replaceState()` 方法，这两个方法应用于浏览器的历史记录栈，提供了对历史记录进行修改的功能。只是当他们进行修改时，虽然修改了url，但浏览器不会立即向后端发送请求。如果要做到改变url但又不刷新页面的效果，就需要前端用上这两个API。

**切换历史状态：** 包括`forward()`、`back()`、`go()`三个方法，对应浏览器的前进，后退，跳转操作。

### ③比较

> 参数传递

hash的**传参是基于url**的，如果要**传递复杂的数据，会有体积的限制**

而history模式不仅可以在**url里放参数**，还可以将**数据存放在一个特定的对象中**。

> 请求后端

hash不请求后端

history请求后端，后端没配置路由的话，请求会404

> 总结

- hash模式带#号比较丑，history模式比较优雅；
- pushState设置的新的URL可以是与当前URL同源的任意URL；而hash只可修改#后面的部分，故只可设置与当前同文档的URL；
- pushState设置的新URL可以与当前URL一模一样，这样也会把记录添加到栈中；而hash设置的新值必须与原来不一样才会触发记录添加到栈中；
- pushState通过stateObject可以添加任意类型的数据到记录中；而hash只可添加短字符串；
- pushState可额外设置title属性供后续使用；
- hash兼容IE8以上，history兼容IE10以上；
- history模式需要后端配合将所有访问都指向index.html，否则用户**刷新页面，会导致404错误**。

# 路由守卫

## 1.全局守卫

`beforeEach(to, from, next)`， `afterEach(to, from)`，`beforeResolve(to, from, next)`

## 2.独享守卫

`beforeEnter(to, from, next)`

## 3.组件内守卫

`beforeRouteEnter(to, from, next)` ，` beforeRouteUpdate(to, from, next)`，`beforeRouteLeave(to, from, next)`

# 单向数据流

Vue提倡单向数据流，即**父级 props 的更新会流向子组件，但是反过来则不行**。这是**为了防止意外的改变父组件状态，使得应用的数据流变得难以理解**，导致数据流混乱。如果破坏了单向数据流，当应用复杂时，debug 的成本会非常高。

**只能通过 `$emit` 派发一个自定义事件，父组件接收到后，由父组件修改。**

# Vue怎么监听数组变化

重写方法在实现时除了将数组方法名对应的**原始方法调用一遍**并将执行结果返回外，**还**通过**执行ob.dep.notify()将当前数组的变更通知给其订阅者，这样当使用重写后方法改变数组后，数组订阅者会将这边变化更新到页面中。**

# 自定义指令

## 1.使用场景

- 普通**DOM元素进行底层操作**的时候，可以使用自定义指令
- 自定义指令是用来操作DOM的。尽管Vue推崇数据驱动视图的理念，但并非所有情况都适合数据驱动。自定义指令就是一种有效的补充和扩展，不仅可用于定义任何的DOM操作，并且是可复用的。

## 2.种类

全局：`Vue.directive("指令名", {钩子函数})`

局部：

```javascript
new Vue({
    directives:{
        // 简写
        指令名(钩子函数参数) {}
        // 完整写
    	指令名:{
			钩子函数(钩子函数参数){}        
    	}
    }
})
```

## 3.钩子函数

`bind`：指令**第一次绑定到元素**时调用，用于初始化设置

`inSerted`：被绑定**元素插入父节点**时

`update`：所在组件**VNode更新**时调用

`ComponentUpdate`：所在**组件的Vnode及子VNode**全部更新后调用

`unbind`：只**调用一次**，指令与元素**解绑时调用**

## 4.钩子函数参数

`el`：绑定元素

`binding`：描述指令全部信息

`vnode`：虚拟节点

`oldVnode`：上一个虚拟节点（更新钩子函数中才有用）

# defineProperty和proxy

## 1.defineProperty

**缺点：**

①添加或删除对象的属性时，Vue 检测不到。因为添加或删除的对象没有在初始化进行响应式处理，只能通过`$set` 来调用`Object.defineProperty()`处理。

②不能监测到数组下标和长度的变化。

## 2.proxy

**优点：**

①代理的是整个对象而不是对象的属性，导致只需做一层代理就可以监听同级结构下的所有属性变化，包括新增和删除。

②能监测数组的变化

## 3.为什么用proxy

defineProperty会**改变原始数据**

proxy是创建对象的虚拟表示，有以下特点：

①不需要使用`Vue.$set()`等触发响应式

②全方位监测数组变化

③支持`Map`， `WeakMap`， `Set`， `WeakSet`

# scoped的处理

给HTML的DOM节点**加一个不重复data属性**(形如：data-v-2311c06a)来表示他的唯一性
在每句css选择器的末尾（编译后的生成的css语句）加一个**当前组件的data属性选择器（如[data-v-2311c06a]）**来私有化样式

# scoped穿透

当引入第三方组件库时(如使用vue-awesome-swiper实现移动端轮播)，**需要在局部组件中修改第三方组件库的样式，而又不想去除scoped属性造成组件之间的样式覆盖**。这时我们可以通过特殊的方式穿透scoped。

**普通css**

```css
.wrapper >>> .swiper-pagination-bullet-active{
    background: #fff
}
```

**less/sass**

```less
.wrapper /deep/ .swiper-pagination-bullet-active{
  background: #fff;
}
```

# 虚拟DOM

## 1.概念

**是**一个**JavaScript对象**，通过**对象的方式来表示DOM结构**。将页面的状态抽象为JS对象的形式，配合不同的渲染工具，使**跨平台渲染成为可能**。通过事务处理机制，将**多次DOM修改的结果一次性的更新到页面上**，从而有效的**减少页面渲染的次数**，**减少修改DOM的重绘重排次数**，**提高渲染性能**。

在vue中，**每个组件都有一个render函数，每个render函数都会返回一个虚拟dom树**，这也就意味着**每个组件都对应一棵虚拟DOM树**

```html
<div>
    <p>123</p>
</div>
```

```javascript
var Vnode = {
    tag: 'div',
    children: [
        { tag: 'p', text: '123' }
    ]
};
```

## 2.为什么操作真实dom开销大

①dom树的**实现模块 和 js 模块 是分开**的，这些**跨模块的通讯**增加了成本。

②**dom 操作引起的浏览器的回流和重绘**，使得性能开销巨大。

③备注：在 pc 端是没有性能问题的（因为 pc 的计算能力强），但随着移动端的发展，越来越多的网页在智能手机上运行，而手机的性能参差不齐，会有性能问题。

## 3.为什么用虚拟dom

### ①保证性能下限，在不进行手动优化的情况下，提供过得去的性能

在vue中，**渲染视图会调用render函数**，这种渲染不仅发生在组件创建时，同时发生在视图依赖的数据更新时。如果在渲染时，**直接使用真实DOM，由于真实DOM的创建、更新、插入等操作会带来大量的性能损耗，从而就会极大的降低渲染效率**。

因此，vue在渲染时，使**用虚拟dom来替代真实dom，主要为解决渲染效率**的问题。

并且虚拟DOM不会立即操作DOM，而是将这10次更新的diff内容保存到本地一个JS对象中，最终将这个JS对象**一次性**attch到DOM树上，再进行后续操作，**避免大量无谓的计算量**。

### ②跨平台

Virtual DOM本质上是JavaScript的对象，它可以很方便的跨平台操作，比如服务端渲染、uniapp等

## 4.虚拟dom怎么转为真实dom

- 首先对将要插入到文档中的 **DOM 树结构进行分析，使用 js 对象将其表示出来**，比如一个元素对象，包含 TagName、props 和 Children 这些属性。然后将这个 js 对象树给保存下来，最后再将 DOM 片段插入到文档中。
- 当页面的状态发生改变，需要对页面的 DOM 的结构进行调整的时候，首先**根据变更的状态，重新构建起一棵对象树，然后将这棵新的对象树和旧的对象树进行比较，记录下两棵树的的差异**。
- 最后**将记录的有差异的地方应用到真正的 DOM 树中去**，这样视图就更新了。

# diff算法

- 首先，对比**节点本身**，判断是否为同一节点，如果**不为相同节点**，则**删除该节点重新创建节点进行替换**
- 如果**为相同节点**，进行**patchVnode**：
  - 找到对应的真实dom，称为`el`
  - 判断`Vnode`和`oldVnode`**是否指向同一个对象**，如果是，那么直接`return`
  - 如果他们**都有文本节点并且不相等**，那么将`el`的文本节点设置为`Vnode`的文本节点。
  - 如果**`oldVnode`有子节点而`Vnode`没有**，则删除`el`的子节点
  - 如果**`oldVnode`没有子节点而`Vnode`有**，则将`Vnode`的子节点真实化之后添加到`el`
  - 如果两者**都有子节点，则执行`updateChildren`函数比较子节点**，这一步很重要
- 比较如果都有子节点，则进行updateChildren，判断如何对这些新老节点的子节点进行操作（diff核心）。
- 匹配时，找到相同的子节点，递归比较子节点

------

在采取diff算法比较新旧节点的时候，比较只会在同层级进行, 不会跨层级比较。

<img src="https://s1.ax1x.com/2022/05/14/OybzMq.png" alt="img" style="zoom: 67%;" />

<img src="https://s1.ax1x.com/2022/05/14/O6CF2T.png" alt="img" style="zoom: 67%;" />

