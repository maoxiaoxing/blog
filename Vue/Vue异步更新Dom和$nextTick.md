# Vue异步更新Dom和$nextTick
--------------------------------

## $nextTick 的使用场景

虽然 Vue 是数据驱动的，但是有时候我们不得不去操作 DOM 去处理一些特殊的场景，而 Vue 更新 DOM 是异步执行的，所以我们不得不去使用 $nextTick 去异步获取 DOM。
```javaScript
<template>
  <div>
    <span ref="msg">{{ msg }}</span>
  </div>
</template>

<script>
export default {
  data() {
    return {
      msg: 'hello nextTick'
    }
  },
  methods: {
    changeMsg() {
      this.msg = 'hello world'
      console.log(this.$refs.msg.innerHTML, '同步获取')
      this.$nextTick(() => {
        console.log(this.$refs.msg.innerHTML, '异步获取')
      })
    }
  },
  mounted() {
    this.changeMsg()
  }
}
</script>
```

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210105215027161-2034604706.png)

我们可以看到，当我我们直接改变数据后，获取 DOM 的话，值是没有改变的，而在 $nextTick 中却可以看到数据发生了变化，为什么呢？下面我们通过源码看一看原因

## Watcher 视图更新
```javaScript
update () {
  /* istanbul ignore else */
  if (this.lazy) {
    this.dirty = true
  } else if (this.sync) {
    /*同步则执行run直接渲染视图*/
    this.run()
  } else {
    /*异步推送到观察者队列中，由调度者调用。*/
    queueWatcher(this)
  }
}
```
如果你看过响应式原理的时候，在 Watcher 中会有一个 update 函数用来更新视图的，当 this.sync 为 false 的时候，就标志着是异步更新，所以会执行 queueWatcher 函数

```JavaScript
 /*将一个观察者对象push进观察者队列，在队列中已经存在相同的id则该观察者对象将被跳过，除非它是在队列被刷新时推送*/
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  /*检验id是否存在，已经存在则直接跳过，不存在则标记哈希表has，用于下次检验*/
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      /*如果没有flush掉，直接push到队列中即可*/
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      // 如果刷新了，那就从队列中取出，立即执行即可
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) { // 没有 waiting，则直接执行 nextTick
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      nextTick(flushSchedulerQueue)
    }
  }
}
```
通过 queueWatcher 函数，我们就能看出来了，Watcher 不是立即更新视图的，而是会放在一个队列中，此时是 waiting 等待状态，它会检查 id 是否重复，如果重复的话，就不会放进队列中；如果没有重复才会放入队列，而且当前 Watcher 是不能刷新的，如果刷新的话，就从队列中取出，没有刷新的 Watcher 才会被放入队列中。如果没有 waiting 等待状态了，那么就证明需要进入下一个 tick 了，会执行 nextTick 方法。

## nextTick
讲了这么多，终于到 nextTick 了
```javaScript
export let isUsingMicroTask = false // 是否使用了微任务

const callbacks = [] /*存放异步执行的回调*/
let pending = false /*一个标记位，如果已经有timerFunc被推送到任务队列中去则不需要重复推送*/

function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

// Here we have async deferring wrappers using microtasks.
// In 2.5 we used (macro) tasks (in combination with microtasks).
// However, it has subtle problems when state is changed right before repaint
// (e.g. #6813, out-in transitions).
// Also, using (macro) tasks in event handler would cause some weird behaviors
// that cannot be circumvented (e.g. #7109, #7153, #7546, #7834, #8109).
// So we now use microtasks everywhere, again.
// A major drawback of this tradeoff is that there are some scenarios
// where microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690, which have workarounds)
// or even between bubbling of the same event (#6566).
let timerFunc /*一个函数指针，指向函数将被推送到任务队列中，等到主线程任务执行完时，任务队列中的timerFunc被调用*/

// The nextTick behavior leverages the microtask queue, which can be accessed
// via either native Promise.then or MutationObserver.
// MutationObserver has wider support, however it is seriously bugged in
// UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
// completely stops working after triggering a few times... so, if native
// Promise is available, we will use it:
/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    // In problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) setTimeout(noop) // 如果是 isIOS 环境，则执行 setTimeout
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && ( // MutationObserver 在 IE 下的兼容性有问题
  isNative(MutationObserver) ||
  // PhantomJS and iOS 7.x
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  // Use MutationObserver where native Promise is not available,
  // e.g. PhantomJS, iOS7, Android 4.4
  // (#6466 MutationObserver is unreliable in IE11)
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // Fallback to setImmediate.
  // Technically it leverages the (macro) task queue,
  // but it is still a better choice than setTimeout.
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  // Fallback to setTimeout.
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}

export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  // 把 cb 加上异常处理存入 callbacks 数组中
  callbacks.push(() => {
    if (cb) {
      try {
        // 调用 cb()
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    // 调用
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    // 返回 promise 对象
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
nextTick 接收两个参数，一个是回调函数，一个是当前环境的上下文，执行 nextTick 会将回调函数放入 callbacks 回调队列中，然后通过 timerFunc 去执行。然后会判断当前执行环境是否有 Promise，如果有的话，通过 Promise.resolve().then 去执行回调函数中的内容，如果是 IOS 环境的话，则执行 setTimeout，因为 IOS 的某些版本对 Promise 的支持不太好；如果当前环境不支持 Promise，则降级使用微任务 MutationObserver，注释中也列举出了很多不支持 Promise 的环境，例如 e.g. PhantomJS, iOS7, Android 4.4；如果 MutationObserver 也不被支持的话，那么就使用宏任务 setImmediate 了；而最坏的情况就是使用 setTimeout 了，至于为什么不直接使用 setTimeout 而多一个 setImmediate，是因为 setImmediate 的执行速度要比 setTimeout，因为 setTimeout 即使将时间参数设为 0 的话，也还是会有 4 ms 的延迟。

## 为什么要异步更新视图

```HTML
<template>
  <div>
    <div>{{value}}</div>
  </div>
</template>
```
```javaScript
export default {
    data () {
        return {
            value: 0
        };
    },
    mounted () {
      for(let i = 0; i < 1000; i++) {
        this.value++;
      }
    }
}
```
当我们在 mounted 钩子函数中，循环改变某一个值的时候，如果没有异步更新，那么 value 每一次 ++ 的时候，都会操作 DOM 去更新，但是这种更新又是没有意义的，这样就会非常消耗性能。但是有了异步 DOM 队列，它只会在下一个 tick 执行，这样就能保障 i 从 0 直接到 1000 才执行，这样大大优化了性能。
