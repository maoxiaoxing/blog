# 彻底搞清楚 Iterator（遍历器）接口
---------------------------------------

## 1. 什么是Iterator(遍历器)
我们都知道 JavaScript 中对数组有很多种遍历方式，但是如果我们想像数组那样去遍历其他数据，我们该怎么办呢？Iterator遍历器就为我们提供了这种机制。
Iterator 是一个接口，它为 JavaScript 中的各种数据结构提供了一种规范。任何数据结构只要部署了 Iterator 接口，那么我们就可以使用 for...of 遍历这个数据结构
在 JavaScript 中部署了 Iterator 接口的有Array、String、Map、Set。
Iterator 的遍历过程是这样的。
（1）创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。
（2）第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。
（3）第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。
（4）不断调用指针对象的next方法，直到它指向数据结构的结束位置。
我们可以通过 Symbol.iterator 属性获取到遍历器
```
const arr = [1, 2, 3]
const arrIterator = arr[Symbol.iterator]()
console.log(arrIterator.next()) // { value: 1, done: false }
console.log(arrIterator.next()) // { value: 2, done: false }
console.log(arrIterator.next()) // { value: 3, done: false }
console.log(arrIterator.next()) // { value: undefined, done: true }
```
我们通过 Symbol.iterator 能获取到 iterator 遍历器，然后通过 next 方法获取到每一个属性的信息对象，信息对象中的 value 就是每一次遍历的值，done 代表了当前数据结构是否被遍历完，没有遍历完就为false，遍历完就为true。

## 2. 实现 iterator 遍历器
之前我们遍历自定义的数据结构可能是这样的
```
const tools = {
    learn: ['html', 'css', 'js'],
    life: ['吃饭', '睡觉', '陪女朋友逛街'],
    hobby: ['看书', '篮球', '台球'],

    each(cb) {
        const all = [].concat(this.learn, this.life, this.hobby)
        for(item of all) {
            cb(item)
        }
    }
}

tools.each((item) => {
    console.log(item)
})
```
下面我们再来看看用"遍历器模式"实现
```
const tools = {
    learn: ['html', 'css', 'js'],
    life: ['吃饭', '睡觉', '陪女朋友逛街'],
    hobby: ['看书', '篮球', '台球'],

    [Symbol.iterator]() {
        const all = [...this.learn, ...this.life, ...this.hobby]
        let index = 0
        return {
            next() {
                return {
                    value: all[index],
                    done: index++ >= all.length,
                }
            }
        }
    }
}

for(const item of tools) {
    console.log(item)
}
```
其实这样实现"迭代器模式"，会稍显复杂一下，我们可以使用 ES6 中的 Generator 函数来实现迭代器
```
const tools = {
    learn: ['html', 'css', 'js'],
    life: ['吃饭', '睡觉', '陪女朋友逛街'],
    hobby: ['看书', '篮球', '台球'],

    *[Symbol.iterator]() {
        const all = [...this.learn, ...this.life, ...this.hobby]
        for(item of all) {
            yield item
        }
    }
}

for (item of tools) {
    console.log(item)
}
```

3. 调用 Iterator 接口的场合
- （1）结构赋值
对数组和 Set 结构进行结构赋值时
```
const set = new Set().add(1).add(2).add(3)
const [first, second] = set
console.log(first, second) // 1 2

const arr = [1,2,3]
const [first, ...rest] = arr
console.log(first, rest) // 1 [2, 3]
```

- （2）扩展运算符
```
const str = 'king'
console.log([...str]) // ['k', 'i', 'n', 'g']
```
只要是某个数据机构部署了 Iterator 接口，那么它就可以使用 rest 运算符将其转化为数组

- （3）Generator函数
yield* 后面如果是一个可以遍历的数据结构，那么就会调用 Iterator 接口
```
const g = function*() {
    yield 1
    yield* [2,3]
}

const i = g()
console.log(i.next())
console.log(i.next())
console.log(i.next())
console.log(i.next())
```
- （4）其他场合
    * for...of
    * Array.from()
    * Map(), Set(), WeakMap(), WeakSet()（比如new Map([['a',1],['b',2]])）
    * Promise.all()
    * Promise.race()

## for...of循环
了解 ES6 的同学一般去遍历数组会使用 Array.forEach() 方法
```
const arr = [1,2,3]
arr.forEach((item) => {
    console.log(item) // 1 2 3
})
```
但是 forEach 方法中无法使用 break 关键字，这就让我们处理某些逻辑的时候不那么方便
而 for...of 中却可以使用 break
```
for (let item of arr) {
    console.log(item) // 1 2
    if (item > 1) break
}
```
