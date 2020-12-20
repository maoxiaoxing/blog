# 手写 Vue2.0 响应式原理
--------------------------

今天来实现一个简易版的Vue2.0响应式
```
class Vue {
    constructor(options) {
        this.$options = options
        this.$data = options.data

        // 重写数组方法
        let arrayPrototype = Array.prototype
        const methods = ['pop', 'push', 'shift', 'unshift']
        this.proto = Object.create(arrayPrototype)
        methods.forEach(method => {
            this.proto[method] = function() {
                arrayPrototype[method].call(this, ...arguments)
            }
        })

        // 响应化
        this.observe(this.$data)

        // 测试代码
        // new Watcher(this, 'test')
        // this.test

        // 创建编译器
        // new Compile(options.el, this)

        if (options.created) {
            options.created.call(this)
        }
    }
    

    // 递归遍历，使传递进来的对象响应化
    observe(value) {
        if (!value || typeof value !== 'object') {
            return
        }

        if (Array.isArray(value)) {
            Object.setPrototypeOf(value, this.proto)
        }

        Object.keys(value).forEach(key => {
            // 对key做响应式处理
            this.defineReactive(value, key, value[key])
            this.proxyData(key)
        })
    }

    // 在Vue根上定义属性代理data中的数据,这样就能通过 this 调用数据
    proxyData(key) {
        Object.defineProperty(this, key, {
            get() {
                return this.$data[key]
            },
            set(newVal) {
                this.$data[key] = newVal
            }
        })
    }

    defineReactive(obj, key, val) { 
        // 递归响应，处理嵌套对象
        this.observe(val)

        // 创建Dep实例： Dep和key一对一对应
        const dep = new Dep()

        // 给obj定义属性
        Object.defineProperty(obj, key, {
            get() {
                // 将Dep.target指向的Watcher实例加入到Dep中, 这部分是收集依赖
                Dep.target && dep.addDep(Dep.target)
                console.log('get')
                return val
            },

            set(newVal) {
                if (newVal !== val) {
                    val = newVal
                    console.log('set')
                    // console.log(`${key}属性更新了`)
                    dep.notify() // 通知视图更新
                }
            }
        })
    }
}

// Dep: 管理若干watcher实例，它和key一对一关系
class Dep {
    constructor() {
        this.deps = []
    }

    addDep(watcher) {
        this.deps.push(watcher)
    }

    notify() {
        this.deps.forEach(watcher => watcher.update())
    }
}

// 实现update函数可以更新
class Watcher {
    constructor(vm, key, cb) {
        this.vm = vm
        this.key = key
        this.cb = cb

        // 将当前实例指向Dep.target
        Dep.target = this
        this.vm[this.key]
        Dep.target = null
    }

    update() {
        console.log(`${this.key}属性更新了`)
        this.cb.call(this.vm, this.vm[this.key])
    }
}
```
这样就实现了一个简易版的Vue，Vue2.0有两个比较明显的问题：
1. 需要注意的是Object.defineProperty的缺点是不能代理数组，所以我们需要对数组的方法进行重写，详细部分请重新看上面的代码，这部分是面试要考的重点。
2. Vue2.0还有一个比较明显的缺点就是，对象上不存在的属性，不能被代理，因为从上面的代码，我们能看出observe遍历的是对象上已经有的属性，所以没有的属性就不会被代理，我们就必须通过调用$set()方法去将新加的属性响应式，这个缺点会在Vue3.0中弥补，有兴趣的同学可以看看Vue3.0的源码。 
3. 我们能看到在defineReactive一进来就调用了observe方法，一是需要代理根对象，二是能代理嵌套对象，而且在new Vue的过程中一次递归代理了所有的对象，这样就会出现一个明显的问题，就是一旦data中的数据层级特别深的时候，就会出现页面渲染比较慢的现象。而且我有一点不明白的地方是为什么在递归代理的时候，为什么不优化一下算法，可以利用栈的思想实现深度优先的非递归实现，然后循环这个栈去代理对象，当然了，尤大大这么写，肯定是人家的原因，但是这是我源码的时候，比较困惑的地方，如果有知道的同学，希望能在下方评论给我解答一下。

