# 浅拷贝和深拷贝
--------------------------------------------

## 一、数据类型
我们都知道JavaScript数据类型分为基本数据类型（String、Number、Boolean、Null、Undefined、Symbol，未来还会有BigInt）和引用数据类型（Object），当然Object还包括Date、function、Array、RegExp。
基本数据类型和引用数据类型在存储方式上是有很大差别的。
- 基本数据类型：变量名和值都存在栈内存中，是一对一的关系
- 引用数据类型：变量名和一个指向值的地址存储在栈地址中，而值存储在堆内存中，是一对多的关系

下面我们用代码和几张图来展示他们之间的差异：
- 基本数据类型
```
let a = 1
let b = a
console.log(b) // 1
b = 2 // 改变变量 b ，变量 a 不变
console.log(a) // 1
console.log(b) // 2
```
![](https://img2020.cnblogs.com/blog/1575596/202010/1575596-20201011202309690-1831187996.png)

- 引用数据类型
```
const person1 = {
    name: 'king',
    age: 25,
}

const person2 = person1
person2.name = 'maoxiaoxing'
console.log(person2) // { name: 'maoxiaoxing', age: 25 }
console.log(person1) // { name: 'maoxiaoxing', age: 25 }
```
![](https://github.com/maoxiaoxing/blog/raw/master/Img/1575596-20201011203917445-303972876%20(1).png)

## 二、浅拷贝与深拷贝的区别
- 赋值其实是将原始对象存储在栈中的指向堆内存的地址赋值给新对象，而不是堆中的数据。所以新对象的任何数据类型的属性值发生改变，都会影响到原始数据。
- 浅拷贝：创建一个新对象，这个对象对原始对象的属性值有着一份精确的拷贝。如果属性是基本类型数据，拷贝的就是基本类型的值，改变原始数据属性，新对象属性不会发生改变；如果属性是引用类型，那么就是拷贝的这个属性的内存地址，如果属性值发生改变，就会影响到原始对象。
- 深拷贝：创建一个新对象，将原始对象从内存中完整的拷贝出来，从堆内存中开辟出一个新的区域存放新对象，且修改新对象，不会影响原始对象

下面我们通过一段代码来看看区别：
```
// 赋值操作
const _ = require("lodash") // 引入lodash

const person1 = {
    name: 'king',
    age: 25,
    hobby: ['篮球'],
}

const person2 = person1
person2.name = 'maoxiaoxing'
person2.hobby.push('读书')
console.log(person2) // { name: 'maoxiaoxing', age: 25, hobby: [ '篮球', '读书' ] }
console.log(person1) // { name: 'maoxiaoxing', age: 25, hobby: [ '篮球', '读书' ] }
```
上面的代码，我们可以看到我们无论是改变 person2 中的任何数据类型的属性值，person1 都会跟随发生变化。
下面我们再来看看浅拷贝
```
// 浅拷贝
const _ = require("lodash") // 引入lodash
const person1 = {
    name: 'king',
    age: 25,
    hobby: ['篮球'],
}

const person3 = _.clone(person1) // lodash函数库中 clone 是一个浅拷贝方法
person3.name = 'maoxiaoxing'
person3.hobby.push('读书')

console.log(person1) // { name: 'king', age: 25, hobby: [ '篮球', '读书' ] }
console.log(person3) // { name: 'maoxiaoxing', age: 25, hobby: [ '篮球', '读书' ] }
```
可以看到，改变 person3 中的基本数据类型，原始数据中对象的属性是不会发生改变的，而 改变 person3 中的引用数据类型后，原始数据也会发生改变。
我们再来看看深拷贝
```
const _ = require("lodash") // 引入lodash

const person1 = {
    name: 'king',
    age: 25,
    hobby: ['篮球'],
}

const person4 = _.cloneDeep(person1)
person4.name = 'maoxiaoxing'
person4.hobby.push('读书')

console.log(person1) // { name: 'king', age: 25, hobby: [ '篮球' ] }
console.log(person4) // { name: 'maoxiaoxing', age: 25, hobby: [ '篮球', '读书' ] }
```
可以看到，经过深拷贝后的新对象person4，无论是改变基本数据类型还是引用数据类型的属性，都不会影响到原始对象。

## 三、浅拷贝的实现方式
1. for 循环只赋值第一层
```
function oneCopy(obj) {
    const resObj = Array.isArray(obj) ? [] : {}
    for (let i in obj) {
        resObj[i] = obj[i]
    }
    return resObj
}
const person1 = {
    name: 'king',
    age: 25,
    hobby: ['篮球'],
}
const person5 = oneCopy(person1)
person5.name = 'maoxiaoxing'
person5.hobby.push('读书')
console.log(person1) // { name: 'king', age: 25, hobby: [ '篮球', '读书' ] }
console.log(person5) // { name: 'maoxiaoxing', age: 25, hobby: [ '篮球', '读书' ] }
```

2. es6中的 Object.assign() 方法
```
const person1 = {
    name: 'king',
    age: 25,
    hobby: ['篮球'],
}
const person6 = Object.assign({}, person1)
person6.name = 'maoxiaoxing'
person6.hobby.push('读书')
console.log(person1) // { name: 'king', age: 25, hobby: [ '篮球', '读书' ] }
console.log(person6) // { name: 'maoxiaoxing', age: 25, hobby: [ '篮球', '读书' ] }
```

3. es6中的rest运算符
```
const person1 = {
    name: 'king',
    age: 25,
    hobby: ['篮球'],
}
const person7 = {...person1}
person7.name = 'maoxiaoxing'
person7.hobby.push('读书')
console.log(person1) // { name: 'king', age: 25, hobby: [ '篮球', '读书' ] }
console.log(person7) // { name: 'maoxiaoxing', age: 25, hobby: [ '篮球', '读书' ] }
```
还有很多，我就不一一列举了，有兴趣的大家可以去挖掘一下哪些api是有浅拷贝功能的

## 四、深拷贝的实现
1. JSON.stringify
```
const obj1 = {
    a: 1,
    b: [1, 2]
}
const obj2 = JSON.parse(JSON.stringify(obj1))
obj2.a = 2
obj2.b.push(3)
console.log(obj1) // { a: 1, b: [ 1, 2 ] }
console.log(obj2) // { a: 2, b: [ 1, 2, 3 ] }
```
JSON.stringify 虽然可以实现深拷贝，但是却不能处理函数和正则，而且当对象中有循环引用会报错
像下面这样
```
const obj1 = {
    a: 1,
    b: [1, 2],
    c: obj1 // 循环引用
}
const obj2 = JSON.parse(JSON.stringify(obj1))
obj2.a = 2
obj2.b.push(3)
console.log(obj1) // TypeError: Converting circular structure to JSON
console.log(obj2)
```

2. 手写递归方法
```
function deepClone(obj, hash = new WeakMap()) {
    if (obj === null) return obj; // 如果是null或者undefined我就不进行拷贝操作
    if (obj instanceof Date) return new Date(obj);
    if (obj instanceof RegExp) return new RegExp(obj);
    // 可能是对象或者普通的值  如果是函数的话是不需要深拷贝
    if (typeof obj !== "object") return obj;
    // 是对象的话就要进行深拷贝
    if (hash.get(obj)) return hash.get(obj);
    let cloneObj = new obj.constructor();
    // 找到的是所属类原型上的constructor,而原型上的 constructor指向的是当前类本身
    hash.set(obj, cloneObj);
    for (let key in obj) {
      if (obj.hasOwnProperty(key)) {
        // 实现一个递归拷贝
        cloneObj[key] = deepClone(obj[key], hash);
      }
    }
    return cloneObj;
}

const obj1 = {
    a: 1,
    b: [1, 2],
}
obj1.c = obj1
const obj2 = deepClone(obj1)
obj2.a = 2
obj2.b.push(3)
console.log(obj1)
console.log(obj2)
```

## 参考文章
- [彻底讲明白浅拷贝与深拷贝](https://www.jianshu.com/p/35d69cf24f1f)
- [如何写出一个惊艳面试官的深拷贝?](https://juejin.im/post/6844903929705136141)
- [深拷贝的终极探索（99%的人都不知道）](https://segmentfault.com/a/1190000016672263)
