# Vue Router原理实现
-----------------------------

## Vue 中的 History 和 Hash 模式的区别

在实现 Vue Router 之前，我们来看看 Vue Router 的两种模式，前端路由的表现形式有两种，一种是 Hash 模式，一种是 History 模式。无论是哪一种都是客户端路由实现的方式，也就是说当路径发生变化时，不会向服务器发送请求，是由 JavaScript 监听路由的变化，根据不同的路由地址渲染不同的内容，如果需要服务器的内容，会发送 ajax 请求来获取。

### 表现形式的区别

- Hash 模式

```
https://music.163.com/#/playlist?id=3102961863
```
上面这种表现形式就是 Hash 模式，"#"后面的内容代表路由地址，"?"后面的内容就是路由传递的参数，这种模式中带有"#"和"?"这样的符号，会显得比较"丑"。

- History 模式

```
https://music.163.com/playlist/3102961863
```
上面这种表现形式就是 History 模式，这种模式就是正常的 url，但是用好 History 模式还需要服务器端的配置支持。

### 原理上的区别

- Hash 模式

Hash 模式是基于锚点，锚点的值作为路由地址，当路由地址发生变化的时候，会触发 onhashchange 事件，在这个函数里可以做一些根据不同路径渲染页面的逻辑。

- History 模式

History 模式是基于 HTML5 中的 History API，也就是 history.pushState() 和 history.replaceState()，history.push() 方法会向服务器发送请求，而 history.pushState() 不会向服务器发送请求，只会改变浏览器中的地址，并且将这个地址记录到历史记录当中，所以通过 history.pushState() 可以去实现客户端路由，但是 history.pushState() 这个方法是 IE10 之后才支持，也就是使用 History 这种模式会有兼容性问题。在路由发生变化时，可以通过 history.popstate() 这个事件来监听路由的变化，根据当前路由地址去找到对应的组件去重新渲染。

## Vue Router 模拟实现

我们已经了解了 Vue Router 有哪两种模式，下面我们来自己实现的一个 Vue Router。下面我们先来看看在 Vue 中路由是如何使用的，通过 Vue Router 的使用我们就可以快速分析出如何去实现
```
// router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'
// 1. 注册路由插件
Vue.use(VueRouter)

// 路由规则
const routes = [
  {
    path: 'home',
    name: 'Home',
    component: Home
  },
]
// 2. 创建 router 对象
const router = new VueRouter({
  routes
})

// main.js
// 创建 Vue 实例，注册 router 对象
new Vue({
  router,
  render: h => h(app)
}).$mount('#app')
```
首先我们需要引入一个 vue-router 插件，然后在 Vue 中注册它，Vue 的强大之处就跟它的插件机制有很大关系。Vue.use() 这个方法可以传入函数或者对象，如果传入一个函数，那么 Vue.use() 会直接调用这个函数，如果传入的是一个对象，那么 Vue.use() 就会调用这个对象中的 install 方法，我们使用的是传入对象的方式。接下来就会创建 vue-router 实例，所以 vue-router 应该是一个构造函数或者一个类，我们使用类的形式，而且这个类中应该有 install 方法，因为这个实例过后的 VueRouter 会直接传给 Vue.use() 这个方法。

### hash模式的路由

```
import Vue from 'vue'
// import VueRouter from 'vue-router'
import Home from './views/Home.vue'


class VueRouter {

    constructor(options) {
        this.$options = options
        this.routeMap = {} // 路由对应组件的 hashMap

        // 路由响应式
        this.app = new Vue({
            data: {
                current: '/'
            }
        })
    }

    init() {
        this.bindEvents() // 监听url变化
        this.createRouteMap() // 解析路由配置
        this.initComponent() // 实现两个组件
    }

    bindEvents() {
        window.addEventListener('load', this.onHashChange.bind(this))
        window.addEventListener('hashchange', this.onHashChange.bind(this))
    }
    onHashChange() {
        this.app.current = window.location.hash.slice(1) || '/'
    }
    createRouteMap(options) { // 将路径作为 key 对应组件作为 value 存入 hashMap
        options.routes.forEach(item => {
            this.routeMap[item.path] = item.component
        })
    }
    initComponent() {
        Vue.component('route-link', { // 创建 route-link 组件
            props: {to: String},
            render(h) {
                return h('a', {attrs:{href: '#' + this.to}}, [this.$slots.default])
            }
        })

        Vue.component('route-view', { // 创建 route-view 组件
            render: (h) => {
                const comp = this.routeMap[this.app.current] // 渲染当前组件
                return h(comp)
            }
        })
    }
}

VueRouter.install = function(Vue) {
    Vue.mixin({
        beforeCreate() {
            if (this.$options.router) {
                Vue.prototype.$router = this.$options.router
                this.$options.router.init()
            }
        }
    })
}
```

### history模式的路由

```
let _Vue = null
class VueRouter {
    static install(Vue){
        //1 判断当前插件是否被安装
        if(VueRouter.install.installed){
            return;
        }
        VueRouter.install.installed = true
        //2 把Vue的构造函数记录在全局
        _Vue = Vue
        //3 把创建Vue的实例传入的router对象注入到Vue实例
        // _Vue.prototype.$router = this.$options.router
        _Vue.mixin({
            beforeCreate(){
                if(this.$options.router){
                    _Vue.prototype.$router = this.$options.router
                    
                }
               
            }
        })
    }
    constructor(options){
        this.options = options
        this.routeMap = {}
        // observable
        this.data = _Vue.observable({
            current:"/"
        })
        this.init()

    }
    init(){
        this.createRouteMap()
        this.initComponent(_Vue)
        this.initEvent()
    }
    createRouteMap(){
        //遍历所有的路由规则 吧路由规则解析成键值对的形式存储到routeMap中
        this.options.routes.forEach(route => {
            this.routeMap[route.path] = route.component
        });
    }
    initComponent(Vue){
        Vue.component("router-link",{
            props:{
                to:String
            },
            render(h){
                return h("a",{
                    attrs:{
                        href:this.to
                    },
                    on:{
                        click:this.clickhander
                    }
                },[this.$slots.default])
            },
            methods:{
                clickhander(e){
                    history.pushState({},"",this.to)
                    this.$router.data.current=this.to
                    e.preventDefault()
                }
            }
            // template:"<a :href='to'><slot></slot><>"
        })
        const self = this
        Vue.component("router-view",{
            render(h){
                // self.data.current
                const cm=self.routeMap[self.data.current]
                return h(cm)
            }
        })
        
    }
    initEvent(){
        //
        window.addEventListener("popstate",()=>{
            this.data.current = window.location.pathname
        })
    }
}
```
