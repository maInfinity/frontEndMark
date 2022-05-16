# 项目初始化：

1. 安装依赖

```
npm i vue-router@3  因为用的是vue2
npm i vuex@3
npm i axios
```

2. 配置跨域

新建vue.config.js

```javascript
module.exports = {
    devServer: {
        proxy: {
            '/api': {// 匹配所有以 '/api'开头的请求路径（根据实际写）
                target: 'xxxxxxx',// 代理目标的基础路径（根据实际接口写）
                changeOrigin: true,
            },
        }
    }
}
```

3. 配置@根路径

新建js.config.json（不用记，直接复制即可）

```
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            "@/*": [
                "src/*"
            ]
        }
    },
    "exclude": [
        "node_modules",
        "dist"
    ]
}
```

4. 初始化样式

```css
body,ul,li,p,h1,h2,h3,h4,h5 {
    margin: 0;
    padding: 0;
}
ul {
    list-style:none;
}
```



# 项目目录：

## **vue.config.js**：

里面配置一些东西，如下配置解决了跨域问题。还可以取消Eslint语法检查

```javascript
module.exports = {
    devServer: {
        proxy: {
            '/api': {// 匹配所有以 '/api'开头的请求路径
                target: 'http://39.98.123.211',// 代理目标的基础路径（根据实际接口写）
                changeOrigin: true,
            },
        }
    }
}
```

## **js.config.json**：

配置路径简写，不用记下来，只要这么配置了，**@字符就默认代表src目录**

```javascript
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            "@/*": [
                "src/*"
            ]
        }
    },
    "exclude": [
        "node_modules",
        "dist"
    ]
}
```

## **main.js**：

入口文件

①导入全局组件并使用

```javascript
import TypeNav from '@/components/TypeNav'
Vue.component('TypeNav', TypeNav)
```

②导入vue-router：导入的是router文件夹下的index.js文件

```javascript
import router from './router'
```

③导入vuex：导入的是store文件夹下的index.js文件

```javascript
import store from '@/store'
```

④导入mock：如果开发需要的话

```javascript
import '@/mock/mockServer'
```

⑤导入全局样式：

```javascript
import 'swiper/css/swiper.css'
```

## **store文件夹**：

①里面需要一个index.js文件，相当于一个总仓库，在这里引入vuex和其他的子仓库

```javascript
import Vue from'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
import home from './home'
import search from './search'
export default new Vuex.Store({
    modules: {
        home,
        search
    }
})
```

②为每个子仓库新建一个js文件

如 home.js

```javascript
//在这里从api文件夹中引入接口请求
import { reqCategoryList, reqBanners, reqFloors } from "@/api"
const state = {}
const actions = {}
const mutations = {}
const getters = {}
export default {
    state,
    actions,
    mutations,
    getters
}
```

## router文件夹

①**routes.js**：配置路由。暴露一个数组

```javascript
export default
    [
        xxxxx
    ]
```

②**index.js**：配置额外功能，如路由守卫

```javascript
import routes from './routes'
const router = new VueRouter({
    routes,
    scrollBehavior() {  //路由跳转后滚动到最上面
        return {
            y: 0
        }
    },
})

router.beforeEach(async (to, from, next) => {
    xxxxx
})
```

## **api文件夹**：

①一个**request.js**文件：在这里重新封装axios

```javascript
import axios from "axios";
import nprogress from "nprogress";		// 导入进度条包
import 'nprogress/nprogress.css'		// 导入进度条样式
const requests = axios.create({
    baseURL: '/api',  	// 配置请求路径前缀
    timeout: 5000		// 配置超时时间
})
import store from '@/store'
requests.interceptors.request.use((config) => {
    nprogress.start()    				// 配置请求拦截器——在这里加入进度条功能
    if (store.state.detail.uuid_token) {
        config.headers.userTempId = store.state.detail.uuid_token
    }
    if (localStorage.getItem('token')) {
        config.headers.token = localStorage.getItem('token')
    }
    return config
})

requests.interceptors.response.use(		// 配置请求拦截器
    (res) => {
        nprogress.done()	// 如果请求成功了 添加这条 ，实现进度条功能
        return res.data
    },
    (err) => {
        console.log(err)	// 这里console.log是因为eslint会对声明了未使用的变量报错
        return Promise.reject(new Error('fail'))
    }
)

export default requests
```

②一个**mockAjax.js**文件：（如果有需要的话，这是为了使用mock）

代码基本同 request.js文件

```javascript
import axios from "axios";
const mockAjax = axios.create({
    baseURL: '/mock',			//这是和 request.js文件不同的地方
    timeout: 5000
})

mockAjax.interceptors.request.use((config) => {
    return config
})
mockAjax.interceptors.response.use(
    (res) => {
        return res.data
    },
    (err) => {
        console.log(err)
        return Promise.reject(new Error('fail'))
    }
)
export default mockAjax
```

③一个**index.js**文件：

引入封装好的axios文件（以及有需要的话引入配置好的mock文件），并在其中配置接口

```javascript
import requests from './request'
import mockAjax from './mockAjax'
export const reqCategoryList = () => requests({ url: '/product/getBaseCategoryList', method: 'get' })
export const reqBanners = () => mockAjax.get('/banners')
export const reqFloors = () => mockAjax.get('/floors')
// 配置的接口是 真正的接口 就用requests发起请求
// 配置的接口是 mock的接口， 就用mockAjax发起请求
```

上面是直接写在index.js中，也可以模块化管理

```javascript
import * as trademark from './product/tradeMark'
export default {
    trademark
}
```

## **mock文件夹**：（!!!注意，还要在API文件夹中配置mock，在main.js中引入mock， 这里都没写到，在上面找）

如果需要mock功能，需要在这里配置：（**注意，配置mock请求的地方是api文件夹，也就是上面**）

①数据格式：**json**文件（不需要暴露，默认json会暴露出去）

```json
[
    {
        "id": "1",
        "imgUrl": "/images/banner1.jpg"
    },
    {
        "id": "2",
        "imgUrl": "/images/banner2.jpg"
    },
    {
        "id": "3",
        "imgUrl": "/images/banner3.jpg"
    },
    {
        "id": "4",
        "imgUrl": "/images/banner4.jpg"
    }
]
```

②配置**mock.js**文件

在这里需要引用**mockjs**包，和，数据文件，以及配置接口

```javascript
import Mock from 'mockjs'
import banners from'./banners.json'
import floors from'./floors.json'
Mock.mock('/mock/banners',{
    code:200,		// 这里是返回的code
    data:banners	// 这里是返回的数据
})
Mock.mock('/mock/floors',{
    code:200,
    data:floors
})
```

# 组件请求vuex数据步骤：

1.先在**api**中**写接口**

2.在**store**中**配置**：

①state初始化数据 

②action中发送请求给接口（然后先打印结果：需要提前在组件中dispatch才会调用，判断请求是否成功发送）

成功发送后，commit数据给mutation 

③mutation修改state数据

3.在vue**开发者工具**中**查看 store**中 是否真的有数据

4.确定有数据后，在**组件**中：

①在mounted中，dispatch 刚刚配置的action（这一步其实**可以最开始做**）

②在computed中...mapState 捞取请求返回的数据

③然后在vue开发者工具中查看返回的数据是否挂载到组件上了

④确定挂载成功后，显示到页面中

# params占位问题

因为params必须占位，如果在路由跳转的时候params为空会出现问题，如下处理即可：

```javascript
this.$router.push({
    name: "search",   // 只要用params参数 路由就必须命名
    params: {
        search_val: this.search_val || undefined,   // 如果有params就传，没有就传undefined
    },
});
```

# 请求只发送一次，提高效率

在App.vue组件的mounted函数中发送请求（这里的请求是更新了store状态，是更新的一个全局的属性）

对于那种很多个页面都要用，数据又不会有变化的情况，直接在App.vue中发这个请求就可以了，在**App.vue中发**

**送的请求只会发送一次**，**不管跳转页面多少次**，因为App.vue是根组件。这样可以节省资源，提高效率。

```javascript
mounted() {
    this.$store.dispatch("categoryList");
},
```

# 合并参数

**params**：

```javascript
if (this.$route.query) {
    let location = {
        name: "search",
        params: {
            keyword: this.keyword || undefined,
        },
    };
    location.query = this.$route.query;
    this.$router.push(location);
}

```

**query**：

```javascript
let el = event.target;
let { categoryname, categoryid1, categoryid2, categoryid3 } = el.dataset;
if (categoryname) {
    let location = { name: "search" };          // 用name跳转路由
    let query = { categoryName: categoryname };
    if (categoryid1) {
        query.categoryId1 = categoryid1;
    } else if (categoryid2) {
        query.categoryId2 = categoryid2;
    } else {
        query.categoryId3 = categoryid3;
    }
    if (this.$route.params) {
        location.params = this.$route.params
        location.query = query;
        this.$router.push(location);
    }
}
```

# store中的getters

主要作用就是简化

# 随机生成ID

nanoid

uuid（自动安装）

# ElementUI

```html
<el-pagination layout="prev, pager, next, jumper,->, total, sizes">
    -> 可以让右边俩东西跑到最后边
</el-pagination>
```

# lodash

## 深拷贝：

```javascript
import cloneDeep from 'lodash/cloneDeep'
```

## 防抖（还没写）：

## 节流：

```javascript
import thorttle from "lodash/throttle";
addIndex: thorttle(function (index) {   // 方法名：thorttle(function(参数) {方法具体功能})
    this.currentIndex = index;
}, 50),
```

# nextTick

有的dom渲染需要时间，所以读取的时候可能是undefined，把对其的操作放在nextTick回调中解决这个问题

如：

轮播图读不到数据问题：如果**在一个使用轮播图的当前组件里发请求，并在mounted中挂载轮播图数据**，由于请求是异步的，会导致数据更新了但是dom还没渲染。又由于轮播图必须先获得结构，而此时vfor的结构还没有生成，所以轮播图**会出现错误**。但是如果不是在这个组件里发送请求，比如在该组件的父组件发送请求，再将数据传送给使用轮播图的组件，那么在mounted中直接挂载不会出错，因为此时数据是传过来的，vfor结构已经生成。所以如果是前面的情况，需要使用nextTick方法。

# moment/dayjs

日期相关包

# 导出接口（方便直接在页面文件中调用，不走vuex）

**main.js**

```javascript
import * as API from '@/api'
beforeCreate() {
    Vue.prototype.$API = API
  },
```

**使用（在 xxx.vue 中）**

```javascript
// 没有模块化管理，直接在index.js中定义了接口
this.$API.reqxxxxx()
// 模块化管理，在index中暴露的是模块，而不是接口
this.$API.模块名.reqxxxxx()
```

# 图片懒加载

**main.js**

```javascript
import VueLazyload from 'vue-lazyload'
import atm from '@/assets/atm.gif'
Vue.use(VueLazyload, {
  loading: atm,
})
```

# Swiper使用

xxx.vue

```vue
<template>
<div class="swiper-container" ref="list">
    <div class="swiper-wrapper">
      <div
        class="swiper-slide"
        v-for="carousel in list"
        :key="carousel.id"
      >
        <img :src="carousel.imgUrl" />
      </div>
    </div>
    <!-- 如果需要分页器 -->
    <div class="swiper-pagination"></div>

    <!-- 如果需要导航按钮 -->
    <div class="swiper-button-prev"></div>
    <div class="swiper-button-next"></div>
  </div>
</template>

<script>
import Swiper from "swiper";
new Swiper(this.$refs.list, {
    loop: true, // 循环模式选项
    // 如果需要分页器
    pagination: {
        el: ".swiper-pagination",
    },
    // 如果需要前进后退按钮
    navigation: {
        nextEl: ".swiper-button-next",
        prevEl: ".swiper-button-prev",
    },
});
</script>
```

# mapState（模块化写法）

vuex模块化时，写法

```javascript
computed: {
    ...mapState({
        categoryList: (state) => state.模块名.categoryList,
    }),
},
```

# 自定义属性

```html
<a :data-自定义属性名="cate1.categoryName">
```

# class / style 动态绑定

**class：**要绑定的类名为key，value是布尔值true / fasle。true就绑定，否则反之

**style：**要绑定的样式名为key（样式名是css存在的，不能瞎编，而且text-align这种短横线连接的要改成驼峰textAlign），value为样式值

```html
<h3 :class="{ cur: currentIndex == index }">绑定class</h3>
<h3 :style="{ display: currentIndex == index ? 'block' : 'none' }">绑定style</h3>
```

# 默认暴露

json文件默认暴露，不需要再手动暴露，可以直接import（图片文件也是）

# Mock

mock.js文件只需要在main.js引入即可，不需要使用，引入的那次即执行了一次（项目中mockjs是在一个文件夹中的一个js文件中使用的，所以实际引入的是那个js文件）

```javascript
import '@/mock/mockServer'
```

# 安装依赖重新运行

在安装完新的包之后，**一定要重新运行web**（不然依赖更新会导致报错）

# 路由懒加载

```javascript
{
    path: '/detail/:skuId',
    component: () => import('@/pages/Detail'),  // 核心
},
```

# 生成随机ID

```javascript
import {v4 as uuidv4} from 'uuid'
export const get_uuid = () => {
    let uuid_token = localStorage.getItem('uuid_token')
    if (!uuid_token) {
        uuid_token = uuidv4()
        localStorage.setItem('uuid_token', uuid_token)
    }
    return uuid_token
}
```

# 生成随机二维码

```javascript
import QRcode from "qrcode";
let url = await QRcode.toDataURL(this.payInfo.codeUrl);
this.$alert(`<img src=${url} />`, { // element-ui功能
    dangerouslyUseHTMLString: true,
    center: true,
    showClose: false,
    showCancelButton: true,
    cancelButtonText: "支付遇到问题",
    confirmButtonText: "支付成功",
});
```

# 初始化CSS文件的引入

①在App.vue中

```vue
<style>
@import url('./assets/css/base.css');
</style>
```

②在main.js中

```javascript
import './assets/css/base.css'
```

# 异步请求导致的问题

异步请求数据时，可能导致dom渲染的时候还没有数据，则调用数据会出现undefined。但是数据请求回来之后由于数据改变了所以dom会更新，从而**页面能够正常显示但是控制台会报错**非常烦人。用以下方法消除控制台错误。

```javascript
this.desc || {}  
//为了避免异步请求出现数据还没加载出来的情况，每次获取异步的数据都要 || {}/[]/'' 等情况
```

# v-if和v-for同时使用

**用computed解决**

可以先用计算属性filter出要的数据，这样就避免两者同时使用

# vant使用轮播图问题

图片使用懒加载后，在最开始底下会有一大片空白。不使用懒加载就好了

# img之间会有留白（div可能也会，不确定）

设置父容器 `font-size:0` 

# Mock的安装

`npm i mockjs`，  不是 `npm i mock`！！！

# UI组件库样式改动 及 css scoped属性

①对于**组件的样式尽量都使用scoped属性**，因为组件间类名命名极有可能重复，避免对其他组件造成预想不到的影响。

②当需要修改UI组件库的样式时，**直接打class是获取不到的！（因为此时的class不是类名，而是传递了一个自定义属性）**。所以**需要用控制台去抓 UI组件库 提供的类名**。

但是有一个很重要的事就是，**UI组件库是全局注册**的，所以**修改它的样式不能在 scoped中** ，这样改不了。

不过由于可以同时写多个style标签，所以可以再写一个没有scoped的style标签`<style>xxxx</style>`，然后把要设置的样式放到这里面。

**！总结：**①`<style scoped></style>`组件样式要写scoped②UI组件库的样式修改专门放在一个新的`<style></style>`中，该标签没有scoped

# 登录后回跳到刚刚浏览的页面

**登录方法中：**

登录时判断路由参数中是否有`redirect`，有就跳转到`redirect`，没有跳到首页

**路由守卫中：**

当token没有时（即没登录时），当要跳转到  必须登录才能查看的页面时（如购物车，订单页），先跳转到登录页，同时在路由参数中加入`redirect = to.path`，比如要跳转到购物车，则`to.path = 'shopCart'`

这样当在登录页登录时，会触发上面的登录方法，此时发现路由参数中有redirect，则跳转回刚刚的页面。

# 分页器封装

1、分页器需要**4个参数**

```javascript
pageNo  	// 当前页
pageSize	// 每页多少条数据
total		// 总数据条数
continues	// 连续区间的长度
```

2、需要计算**startPageNo**，是连续区间的开始页数

```javascript
// 假设共有 31页， 则有以下情况
// 1 2 3 4 5 ... 31
// 1 ... 27 28 29 30 31

// 1 ... 3 4 5 6 7... 31
// 1 ... 25 26 27 28 29 ... 31

// 当 连续区间的中位数 - 2 ≥ 1 就应该出现 ... ，但因为 1 ... 2 无意义
// 比如1 ... 2 3 4 5 6 ... 31， 所以 当 连续区间的开始页数≤连续区间长度-1时，startPageNo 都 = 1
// 反之同理  开始页数≥totalPage - （连续区间个数 - 1） 则
// startPageNo = this.totalPage - this.continues + 1;

// 其他时候，即两边都有 ... 则， startPageNo = this.pageNo - Math.floor(this.continues / 2);
```

# 权限控制

## 路由权限

**核心：**通过**addRoutes**配置动态路由

登录后，根据服务端返回的 用户个人信息 数据中 携带的 routes（这里需要后台配置好返回来），这里面包含了可以查看的路由（即权限），然后考虑到可能有公开路由（即任何人都能访问的路由，如首页），再将其和公开路由拼接在一起，再通过`router.addRoutes(fullRoutes)`即可

```javascript
let openRoutes = [
    {
        path:'/',
        name:'Home',
        component:Home
    }
]
fullRoutes = openRoutes.concat(userRoutes)
router.addRoutes(fullRoutes)
```

## 按钮权限

将登录用户的按钮权限信息存储到store中，在按钮上通过v-if判断按钮可见与否





