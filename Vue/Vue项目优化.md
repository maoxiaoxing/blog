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
