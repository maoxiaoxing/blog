# Vuex的使用并实现一个简易版Vuex
---------------------

## Vuex 回顾

### 什么是 Vuex

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/vuex.png)

Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件
的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

- Vuex 是专门为 Vue.js 设计的状态管理库
- 它采用集中式的方式存储需要共享的数据
- 从使用角度，它就是一个 JavaScript 库
- 它的作用是进行状态管理，解决复杂组件通信，数据共享

### 什么情况下使用 Vuex

> 官方文档：
Vuex 可以帮助我们管理共享状态，并附带了更多的概念和框架。这需要对短期和长期效益进行权
衡。
如果您不打算开发大型单页应用，使用 Vuex 可能是繁琐冗余的。确实是如此——如果您的应用
够简单，您最好不要使用 Vuex。一个简单的 store 模式就足够您所需了。但是，如果您需要构建
一个中大型单页应用，您很可能会考虑如何更好地在组件外部管理状态，Vuex 将会成为自然而然
的选择。引用 Redux 的作者 Dan Abramov 的话说就是：Flux 架构就像眼镜：您自会知道什么时
候需要它。

当你的应用中具有以下需求场景的时候：

- 多个视图依赖于同一状态
- 来自不同视图的行为需要变更同一状态

建议符合这种场景的业务使用 Vuex 来进行数据管理，例如非常典型的场景：购物车。
**注意：Vuex 不要滥用，不符合以上需求的业务不要使用，反而会让你的应用变得更麻烦。**

### Vuex 的简单使用

下面我们先来回顾一下 vuex 是怎样使用的，我们来做一个增加硬币的案例

#### 安装

```js
// npm
npm install vuex --save

// yarn
yarn add vuex
```

#### 基础配置

在 src 目录下新建一个 store 文件夹，创建一个 index.js 文件。然后将写入下面的代码。

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    coin: 0
  },
})

export default store
```

然后在 mian.js 中引入 store

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router' // 引入 store 
import store from './store'

import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
Vue.use(ElementUI)

Vue.config.productionTip = false

new Vue({
  router,
  store, // 注册 store
  render: h => h(App),
}).$mount('#app')
```

#### Getters

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

1. 在 store 中增加 getters 属性

```js
const store = new Vuex.Store({
  state: {
    coin: 0
  },
  getters: {
    getCoin(state) {
      return state.coin
    }
  },
})
```

2. 在组件中使用 getters

- 直接使用

```html
<p>硬币：{{ $store.getters.getCoin }}</p>
```

- mapGetters

vuex 提供了一种将 store 中的 getter 映射到计算属性的方法

```html
<p>硬币：{{ getCoin }}</p>
```

```js
import { mapGetters } from 'vuex'

export default {
  computed: {
    ...mapGetters(['getCoin']),
  },
}
```

#### Mutations

Mutations 中的方法用来更改 store 中的 state 的数据，Mutations 中的数据必须是同步数据。

1. 在 store 中增加 mutations 字段

mutations 中的函数有两个参数，第一个是 store 中的 state，第二个是通过 store.commit 传过来的数据

```js
const store = new Vuex.Store({
  state: {
    coin: 0
  },
  getters: {
    getCoin(state) {
      return state.coin
    }
  },
  mutations: {
    commitCoin (state, n) {
      state.coin += n
    }
  },
})
```

2. 在组件中使用 mutations

- 直接使用

```html
<div>
  <el-input 
    v-model="coins"
    placeholder="请输入增加的硬币数目"
    style="width: 300px"
  ></el-input>
  <el-button 
    type="primary"
    @click="addCion"
  >增加一个硬币</el-button>
  <p>硬币：{{ getCoin }}</p>
</div>
```

```js
import { mapGetters } from 'vuex'

export default {
  data() {
    return {
      coins: 1
    }
  },
  computed: {
    ...mapGetters(['getCoin']),
  },
  methods: {
    addCion() {
      // store.commit
      this.$store.commit('commitCoin', Number(this.coins))
    }
  }
}
```
$store.commit 接收两个参数，一个是 store 中的 mutations 的函数名称，一个是需要传入 mutations 函数的参数。

- mapMutations

```js
import { 
  mapGetters,
  mapMutations,
} from 'vuex'

export default {
  data() {
    return {
      coins: 1
    }
  },
  computed: {
    ...mapGetters(['getCoin']),
  },
  methods: {
    ...mapMutations(['commitCoin']),
    addCion() {
      // 等同于 this.$store.commit('commitCoin', Number(this.coins))
      this.commitCoin(Number(this.coins))
    }
  }
}
```

#### Actions

> Action 类似于 mutation，不同在于：
> - Action 提交的是 mutation，而不是直接变更状态。
> - Action 可以包含任意异步操作。

1. 在 store 中增加 actions 字段

```js
const store = new Vuex.Store({
  state: {
    coin: 0
  },
  getters: {
    getCoin(state) {
      return state.coin
    }
  },
  mutations: {
    commitCoin (state, n) {
      state.coin += n
    }
  },
  actions: {
    addCoinSync(context, payload) {
      setTimeout(() => {
        context.commit('commitCoin', payload)
      }, 1000)
    }
  }
})
```

2. 在组件中使用 actions

- 直接使用

```js
import {
  mapGetters,
  mapMutations,
} from 'vuex'

export default {
  data() {
    return {
      coins: 1
    }
  },
  computed: {
    ...mapGetters(['getCoin']),
  },
  methods: {
    ...mapMutations(['commitCoin']),
    addCion() {
      // this.$store.commit('commitCoin', Number(this.coins))
      // this.commitCoin(Number(this.coins))
      this.$store.dispatch('addCoinSync', Number(this.coins))
    }
  }
}
```

- mapActions

```js
import { 
  mapGetters,
  mapMutations,
  mapActions,
} from 'vuex'

export default {
  data() {
    return {
      coins: 1
    }
  },
  computed: {
    ...mapGetters(['getCoin']),
  },
  methods: {
    ...mapMutations(['commitCoin']),
    ...mapActions(['addCoinSync']),
    addCion() {
      // 等同于 this.$store.dispatch('addCoinSync', Number(this.coins))
      this.addCoinSync(Number(this.coins))
    }
  }
}
```

## Vuex 模拟实现

### 实现思路

上面我们回顾了 Vuex 的使用，下面我们来自己模拟一个 Vuex

- 实现 install 方法
  * Vuex 是 Vue 的一个插件，所以和模拟 VueRouter 类似，先实现 Vue 插件约定的 install 方 法
- 实现 Store 类
  * 实现构造函数，接收 options
  * state 的响应化处理
  * getter 的实现
  * commit、dispatch 方法

```js
let Vue

class Store{
    constructor(options) {
      this.state = new Vue({
          data: options.state
      })

      this.mutations = options.mutations
      this.actions = options.actions

      options.getters && this.handleGetters(options.getters)
    }

    commit = (type, arg) => {
      this.mutations[type](this.state, arg)
    }

    dispatch(type, arg) {
      this.actions[type]({
        commit: this.commit,
        state: this.state,
      }, arg)
    }

    handleGetters(getters) {
      this.getters = {}
      // 遍历getters所有key
      Object.keys(getters).forEach(key => {
        // 为this.getters定义若干属性，这些属性是只读的
        Object.defineProperty(this.getters, key, {
            get: () => {
                return getters[key](this.state)
            }
        })
      }) 
    }
}

// 向外提供 install
function install (_Vue) { 
  Vue = _Vue

  Vue.mixin({
    beforeCreate() {
      // 在 beforeCreate 中注册 store
      if (this.$options.store) {
          Vue.prototype.$store = this.$options.store
      }
    }
  })
 }

export default {
  Store,
  install,
}
```
