```
Array.prototype.my_filter = function(fn, context) {

    let resArr = []
    const me = this

    const ctx = context ? context : this // 判断上下文

    if (typeof fn !== 'function') {
        throw new Error(`${fn} is not a function`)
    }

    me.forEach((item, index) => {
        const bool = fn.call(ctx, item, index, me) // 绑定上下文，并执行结果
        if (bool) { // 如果符合条件，就放进新的数组
            resArr.push(item)
        }
    })

    return resArr
}
```
下面来验证一下
```
const arr = [1,2,3]

const newArr = arr.my_filter(function(item, index, _arr) {
    console.log(this, item, index, _arr)
    return item >= 2
})

console.log(newArr)
```
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200805223049463-1301551721.png)

再来验证一下绑定上下文
```
const arr = [1,2,3]
const arr1 = [4,5,6]

const newArr = arr.my_filter(function(item, index, _arr) {
    console.log(this, item, index, _arr)
    return item >= 2
}, arr1)

console.log(newArr)
```
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200805223141594-1115804185.png)

再来看一下错误验证
```
const arr = [1,2,3]
const arr1 = [4,5,6]

const newArr = arr.my_filter(1, arr1)

console.log(newArr)
```
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200805223259387-243195692.png)
