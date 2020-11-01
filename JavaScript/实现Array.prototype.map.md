```javaScript
Array.prototype.my_map = function(fn, context) {

    let resArr = []
    const me = this

    const ctx = context ? context : me // 定义上下文

    if (typeof fn !== 'function') {
        throw new Error(`${fn} is not a function`)
    }

    me.forEach((item, index) => {
        resArr.push(fn.call(ctx, item, index, me)) // 将回调结果放入数组中
    })

    return resArr // 返回map后的数组
}
```
下面来验证一下
```
const arr = [1,2,3]

const newArr = arr.my_map(function(item, index, _arr) {
    console.log(this, item, index, _arr)
    return item * 2
}, arr1)

console.log(newArr)
```

![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200805221719978-974881354.png)

可以看到还是比较成功的，再来验证一下上下文有没有绑定成功
```
const arr = [1,2,3]
const arr1 = [4,5,6]

const newArr = arr.my_map(function(item, index, _arr) {
    console.log(this, item, index, _arr)
    return item * 2
}, arr1)

console.log(newArr)
```
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200805221748709-438814975.png)

再看一下错误处理
```
const arr = [1,2,3]
const arr1 = [4,5,6]

const newArr = arr.my_map(123, arr1)

console.log(newArr)
```
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200805221832793-608006533.png)

ok!大功告成了
