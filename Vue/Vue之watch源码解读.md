# Vue之watch源码解读
---------------

## 回顾 watch 的用法

watch 是 Vue 中的一个监听数据变化的一个方法，我们在阅读源码之前先来回顾一下 watch 的用法

### 监听基本数据类型

```html
<div>
  {{ name }}
  <button @click="changeName">改变name</button>
</div>
```

```javaScript
export default {
  data() {
    return {
      name: 'maoxiaoxing',
    }
  },
  watch: {
    name(val, oldval) {
      console.log(val, oldval)
    }
  },
  methods: {
    changeName() {
      this.name = this.name === 'maoxiaoxing' ? 'yangxiaoA' : 'maoxiaoxing'
    }
  }
}
```

watch 可以接收两个参数，一个是变化之后的数据，一个是变化之前的数据，你可以基于这两个值处理一些逻辑

### 监听对象

```html
<div>
  {{ obj.name }}
  <button @click="changeName">改变name</button>
</div>
```

```javaScript
export default {
  data() {
    return {
      obj: {
        name: 'maoxiaoxing',
      }
    }
  },
  watch: {
    obj: {
      handler(val, oldval) {
        console.log(val, oldval)
      },
      deep: true,
      immediate: true,
    }
  },
  methods: {
    changeName() {
      this.obj.name = this.obj.name === 'maoxiaoxing' ? 'yangxiaoA' : 'maoxiaoxing'
    }
  },
  created() {
    console.log('created')
  }
}
```

在监听对象变化的时候，加上 deep 这个属性即可深度监听对象数据；如果你想在页面进来时就执行 watch 方法，加上 immediate 即可。值得注意的是，<mark>设置了 immediate 属性的 watch 的执行顺序是在 created 生命周期之前的</mark>

### watch 接收参数为数组

我在看 Vue 源码的时候，发现了一个比较有意思的地方，如果说 watch 监听的属性不去设置一个方法而是接收一个数组的话，可以向当前监听的属性传递多个方法

```javaScript
export default {
  data() {
    return {
      name: 'jack',
    }
  },
  watch: {
    name: [
      { handler: function() {console.log(1)}, immediate: true },
      function(val) {console.log(val, 2)}
    ]
  },
  methods: {
    changeName() {
      this.name = this.name === 'maoxiaoxing' ? 'yangxiaoA' : 'maoxiaoxing'
    }
  }
}
```
数组中可以接收不同形式的参数，可以是方法，也可以是一个对象，具体的书写方式和普通的 watch 没什么不同。可以接收数据为参数这一点在官方文档没有找到，至于为什么可以这样写，下面的源码讲解会提及。

## 初始化 watch

### initState

```javaScript
// src\core\instance\state.js
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props) // 如果有 props ，初始化 props
  if (opts.methods) initMethods(vm, opts.methods) // 如果有 methods ，初始化 methods 里面的方法
  if (opts.data) { // 如果有 data 的话，初始化，data；否则响应一个空对象
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed) // 如果有 computed ，初始化 computed
  if (opts.watch && opts.watch !== nativeWatch) { // 如果有 watch ，初始化 watch
    initWatch(vm, opts.watch)
  }
}
```

首先在 initState 初始化 watch，如果有 watch 这个属性的话，就将 watch 传入 initWatch 方法中处理

### initWatch

```javaScript
// src\core\instance\state.js
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
```

这个函数主要就是初始化 watch，我们可以看到 initWatch 会遍历 watch，然后判断每一个值是否是数组，如果是数组的就遍历这个数组，创建多个回调函数，这块也就解释了上边 watch 监听的数据可以接收数组为参数原因；如果不是数组的话，就直接创建回调函数。从这里我们也能看到，我们学习源码的好处，通过学习源码，我们能学习到一些在官方文档中没有提到的写法。

### createWatcher

```javaScript
function createWatcher (
  vm: Component,
  expOrFn: string | Function,
  handler: any,
  options?: Object
) {
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {
    handler = vm[handler]
  }
  return vm.$watch(expOrFn, handler, options)
}
```

createWatcher 中会判断 handler 是否是对象，如果是对象将 handler 挂载到 options 这个属性，再将对象的 handler 属性提取出来；如果 handler 是一个字符串的话，会从 Vue 实例找到这个方法赋值给 handler。从这里我们也能看出来，watch 还可以支持字符串的写法。执行 Vue 实例上的 $watch 方法。

### $watch

```javaScript
Vue.prototype.$watch = function (
  expOrFn: string | Function,
  cb: any,
  options?: Object
): Function {
  // 获取 Vue 实例 this
  const vm: Component = this
  if (isPlainObject(cb)) {
    // 判断如果 cb 是对象执行 createWatcher
    return createWatcher(vm, expOrFn, cb, options)
  }
  options = options || {}
  // 标记为用户 watcher
  options.user = true
  // 创建用户 watcher 对象
  const watcher = new Watcher(vm, expOrFn, cb, options)
  // 判断 immediate 如果为 true
  if (options.immediate) {
    // 立即执行一次 cb 回调，并且把当前值传入
    try {
      cb.call(vm, watcher.value)
    } catch (error) {
      handleError(error, vm, `callback for immediate watcher "${watcher.expression}"`)
    }
  }
  // 返回取消监听的方法
  return function unwatchFn () {
    watcher.teardown()
  }
}
```

$watch 函数是 Vue 的一个实例方法，也就是我们可以使用 Vue.$watch 去调用，这里不再过多赘述，[官方文档](https://cn.vuejs.org/v2/api/#vm-watch)中讲的很详细。$watch 会创建一个 Watcher 对象，这块也是涉及响应式原理，在 watch 中改变的数据可以进行数据的响应式变化。同时也会判断是否有 immediate 这个属性，如果有的话，就直接调用回调。

### Watcher

```javaScript
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      // expOrFn 是字符串的时候，例如 watch: { 'person.name': function... }
      // parsePath('person.name') 返回一个函数获取 person.name 的值
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  /*获得getter的值并且重新进行依赖收集*/
  get () {
     /*将自身watcher观察者实例设置给Dep.target，用以依赖收集。*/
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      /*
      执行了getter操作，看似执行了渲染操作，其实是执行了依赖收集。
      在将Dep.target设置为自身观察者实例以后，执行getter操作。
      譬如说现在的的data中可能有a、b、c三个数据，getter渲染需要依赖a跟c，
      那么在执行getter的时候就会触发a跟c两个数据的getter函数，
      在getter函数中即可判断Dep.target是否存在然后完成依赖收集，
      将该观察者对象放入闭包中的Dep的subs中去。
    */
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      /*如果存在deep，则触发每个深层对象的依赖，追踪其变化*/
      if (this.deep) {
        /*递归每一个对象或者数组，触发它们的getter，使得对象或数组的每一个成员都被依赖收集，形成一个“深（deep）”依赖关系*/
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  ... 其他方法
}
```

上面的 Watcher 我省略了一些其他方法，只保留了 get 函数，我们能在 get 函数中看到如果有 deep 属性的话，就会递归处理对象中的每一个属性，以达到深度监听的效果。这里关于 watch 的使用和原理讲解就完结了，我们通过阅读源码，不仅能够了解 Vue 框架内部是怎样实现的，同时也能看到一些官方文档没有提及的用法，对我们是很有帮助的。
