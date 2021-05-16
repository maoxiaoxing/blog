# Vue 项目优化

## v-if 和 v-show 区分使用场景

v-if 是 真正 的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建；也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

- true：渲染元素
- false：不渲染，移除元素

v-show 就简单得多， 不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 的 display 属性进行切换。

- true：display: block
- false：display: none

所以，v-if 适用于在运行时很少改变条件，不需要频繁切换条件的场景；v-show 则适用于需要非常频繁切换条件的场景。

## 多使用计算属性 computed

computed： 是计算属性，依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed 的值；
当我们需要进行数值计算，并且依赖于其它数据时，我们尽量不要使用 watch，而是应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时，都要重新计算；你可以将计算属性理解成可以处理逻辑的 data。

```js
export default {
  data() {
    return {
      actualPrice: 10, // 实际价格
      sellingPrice: 20, // 售卖价格
      profit: 0, 利润
    }
  },
  watch: {
    actualPrice: {
      handler(val) {
        this.profit = this.sellingPrice - this.actualPrice
      },
      immediate: true,
    },
    sellingPrice: {
      handler(val) {
        this.profit = this.sellingPrice - this.actualPrice
      },
      immediate: true,
    }
  }
}
```

这种处理价格的逻辑如果用 watch 去处理，不但麻烦而且性能不是很好，如果使用 computed 去处理就会非常方便，而且 computed 能够缓存计算结果，如果你的依赖值不发生改变，那么逻辑运算方法就不会被触发

```js
export default {
  data() {
    return {
      actualPrice: 10, // 实际价格
      sellingPrice: 20, // 售卖价格
    }
  },
  computed: {
    profit() {
      return this.sellingPrice - this.actualPrice
    }
  },
}
```

## v-for 遍历避免同时使用 v-if

v-for 的优先级其实是比 v-if 高的，所以当两个指令出现来一个 DOM 中，那么 v-for 渲染的当前列表，每一次都需要进行一次 v-if 的判断。而相应的列表也会重新变化，这个看起来是非常不合理的。因此当你需要进行同步指令的时候。尽量使用计算属性，先将 v-if 不需要的值先过滤掉。他看起像是下面这样的。

```js
// 计算属性
computed: {
  filterList() {
    return this.showData.filter((data) => {
      return data.isShow
    })
  }
}
  
// DOM
  
<ul>
  <li v-for="item in filterList" :key="item.id">
  {{ item.name }}
  </li>
</ul>
```

## v-for 遍历必须为 item 添加 key，且避免使用 index 作为标识

在列表数据进行遍历渲染时，需要为每一项 item 设置唯一 key 值，方便 Vue.js 内部机制精准找到该条列表数据。当 state 更新时，新的状态值和旧的状态值对比，较快地定位到 diff 。
而且 vFor 是不推荐使用 index 下标来作为 key 的值，这是一个非常好理解的知识点，可以从图中看到，当index作为标识的时候，插入一条数据的时候，列表中它后面的key都发生了变化，那么当前的 vFor 都会对key变化的 Element 重新渲染，但是其实它们除了插入的 Element 数据都没有发生改变，这就导致了没有必要的开销。所以，尽量不要用index作为标识，而去采用数据中的唯一值，如 id 等字段。

![](https://img2020.cnblogs.com/blog/1575596/202105/1575596-20210516173044650-185225398.png)

## 长列表性能优化

Vue 会通过 Object.defineProperty 对数据进行劫持，来实现视图响应数据的变化，然而有些时候我们的组件就是纯粹的数据展示，不会有任何改变，我们就不需要 Vue 来劫持我们的数据，在大量数据展示的情况下，这能够很明显的减少组件初始化的时间，那如何禁止 Vue 劫持我们的数据呢？可以通过 Object.freeze 方法来冻结一个对象，一旦被冻结的对象就再也不能被修改了。

```js
export default {
  data () {
    return {
      users: {}
    }
  },
  async created () {
    const users = await axios.get('/api/users')
    this.users = Object.freeze(users)
  }
}
```

## 优化无限列表性能

项目当中，会涉及到非常多的长列表场景，区别于普通的分页来说，大部分的前端在做这种 无限列表 的时候，大部分新手前端都是通过一个 vFor 将数据遍历出来，想的多一点的就是做一个分页。滚动到底部的时候就继续请求 API 。其实这也是未思考妥当的。随着数据的加载，DOM会越来越多，这样就导致了性能开销的问题产生了，当页面上的DOM太多的时候，难免给我的客户端造成一定的压力，所以对于长列表渲染的时候，建议将DOM移除掉，类似于图片懒加载的模式，只有出现在视图上的DOM才是重要的DOM。网络上有一些很好的解决方案，像 ElementUI 和 Ant-design 这样的 UI 库都是有无限滚动的组件的，如 [vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller)、[vue-virtual-scroll-list](https://github.com/tangbc/vue-virtual-scroll-list) 库等等，大家可以理性的选择。

其它解决方案见：https://github.com/vuejs/awesome-vue#infinite-scroll。

## 释放组件资源

Vue 组件销毁时，会自动清理它与其它实例的连接，解绑它的全部指令及事件监听器，但是仅限于组件本身的事件。 如果在 js 内使用 addEventListene 等方式是不会自动销毁的，我们需要在组件销毁时手动移除这些事件的监听，以免造成内存泄露，如：

```js
created() {
  addEventListener('click', this.click, false)
},
beforeDestroy() {
  removeEventListener('click', this.click, false)
}
```

再比如定时器：

```js
created() {
  this.currentInterVal = setInterval(code, millisec, lang)
},
beforeDestroy() {
  clearInterval(this.currentInterVal)
}
```

## 图片资源优化

在网页中，往往存在大量的图片资源，这些资源或大或小。当我们页面中DOM中存在大量的图片时，难免不会碰到一些加载缓慢的问题，甚至加载失败的问题。

- 小图标使用 SVG 或者字体图标，或者制作雪碧图
- 通过 base64 和 webp  的方式加载小型图片
- 能通过 cdn 加速的大图尽量用 cdn
- 对图片资源进行懒加载
- 压缩图片大小
    * 如果是纯静态图片，建议使用统一的工具进行压缩
    * 使用 tinypng 等工具
    * 注意：不建议用 webpack 构建压缩图片。
    * 如果是动态图片，比如用户创建的文章中的图片，这个肯定是建议由后端来压缩，现代化的后端也很少自己处理图片压缩了，大多数都是用的付费的云服务。
