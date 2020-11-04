# var、let、const三种声明变量方式之间的差异
------------------------------------------------

- var声明的变量会挂载到window上，而let和const声明的变量不会
```
var a = 'foo'
console.log(window.a) // foo

let b = 'bar'
console.log(window.b) // undefined

const c = 'baz'
console.log(window.c) // undefined
```

- var 存在变量提升，而let和const不存在变量提升，let和const上面的区域称为“暂时性死区”，在定义之前不可以使用
```
console.log(a) // undefined
var a = 'foo'

console.log(b) // test4.html:14 Uncaught ReferenceError: Cannot access 'b' before initialization
let b = 'bar'

console.log(c) // test4.html:14 Uncaught ReferenceError: Cannot access 'c' before initialization
const c = 'baz'
```

- let和const的声明会形成块级作用域
```
if (true) {
    var a = 'foo'
    let b = 'bar'
}
console.log(a) // foo
console.log(b) // Uncaught ReferenceError: b is not defined

if (true) {
    var a = 'foo'
    const c = 'baz'
}
console.log(a) // foo
console.log(c) // Uncaught ReferenceError: c is not defined
```

- 在同一个作用域下，let和const不能再次声明同一个变量，而var可以
```
var a = 'foo'
var a = 'bar'
console.log(a) // bar

let b = 'foo'
let b = 'bar'
console.log(b) // test4.html:34 Uncaught SyntaxError: Identifier 'b' has already been declared

const c = 'foo'
const c = 'bar'
console.log(c) // Uncaught SyntaxError: Identifier 'c' has already been declared
```

- let定义变量之后可以修改，而const表面上像是声明一个“常量”。const并不是保证变量的值不得改动，而是指变量指向的内存地址不得改变。对于简单数据类型（Number、String、Boolean），值就保存在变量指向的内存地址，因此等同于常量；而对于复合数据类型（主要是对象和数组），变量只是保存了指向堆内存的地址，至于堆内的数据是不是可变的，就不能控制了。
```
const person = {}
person.name = 'maoxiaoxing'
console.log(person) // {name: "maoxiaoxing"}

person = {name: 'king'} // test4.html:45 Uncaught TypeError: Assignment to constant variable.

const a = ['foo']
a.push('bar')
a[2] = 'baz'
console.log(a) // ["foo", "bar", "baz"]
```