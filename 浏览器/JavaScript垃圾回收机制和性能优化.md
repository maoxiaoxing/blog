# JavaScript垃圾回收机制和性能优化
------------------------------------------------------
## 前言
我们都知道程序的运行需要一定的内存空间，且在运行过后就必须将不再用到的内存释放掉，否则就会出现下图中内存的占用持续升高的情况，一方面会影响程序的运行速度，另一方面严重的话则会导致整个程序的崩溃。
![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E5%86%85%E5%AD%98.png)
## JavaScript中的内存管理
- 内存：由可读写单元组成，表示一片可操作空间
- 管理：人为的去操作一片空间的申请、使用和释放
- 内存管理：开发者主动申请空间、使用空间、释放空间
- 管理流程：申请-使用-释放

部分语言需要（例如C语言）需要手动去释放内存，但是会很麻烦，所以很多语言都会提供自动的内存管理机制，称为“垃圾回收机制”，JavaScript语言中也提供了垃圾回收机制（Garbage Collecation），简称GC机制

## 全停顿（Stop The World ）
在介绍垃圾回收算法之前，我们先了解一下「全停顿」。垃圾回收算法在执行前，需要将应用逻辑暂停，执行完垃圾回收后再执行应用逻辑，这种行为称为 「全停顿」（Stop The World）。例如，如果一次GC需要50ms，应用逻辑就会暂停50ms。
全停顿的目的，是为了解决应用逻辑与垃圾回收器看到的情况不一致的问题。举个例子，在自助餐厅吃饭，高高兴兴地取完食物回来时，结果发现自己餐具被服务员收走了。这里，服务员好比垃圾回收器，餐具就像是分配的对象，我们就是应用逻辑。在我们看来，只是将餐具临时放在桌上，但是服务员看来觉得你已经不需要使用了，因此就收走了。你与服务员对于同一个事物看到的情况是不一致，导致服务员做了与我们不期望的事情。因此，为避免应用逻辑与垃圾回收器看到的情况不一致，垃圾回收算法在执行时，需要停止应用逻辑。

## JavaScript中的垃圾回收
JavaScript中会被判定为垃圾：
- 对象不再被引用是垃圾
- 对象不能从根上访问到时垃圾

常见的GC算法：
- 引用计数
- 标记清除
- 标记整理
- 分代回收

### （1）. 引用计数
最常使用的方法叫做"引用计数"（reference counting）：语言引擎有一张"引用表"，保存了内存里面所有的资源（通常是各种值）的引用次数。如果一个值的引用次数是0，就表示这个值不再用到了，因此可以将这块内存释放。
```
const user1 = {age: 11}
const user2 = {age: 22}
const user3 = {age: 33}

const userList = [user1.age, user2.age, user3.age]
```
上面这段代码，当执行过一遍过后，user1、user2、user3都是被userList引用的，所以它们的引用计数不为零，就不会被回收

```
function fn() {
    const num1 = 1
    const num2 = 2
}

fn()
```
上面代码中fn函数执行完毕，num1、num2都是局部变量，执行过后，它们的引用计数就都为零，所有这样的代码就会被当做“垃圾”，进行回收
<font color="red">引用计数算法有一个比较大的问题: 循环引用</font>
```
function objGroup(obj1, obj2) {
    obj1.next = obj2
    obj2.prev = obj1

    return {
        o1: obj1,
        o2: obj2,
    }
}

let obj = objGroup({name: 'obj1'}, {name: 'obj2'})
console.log(obj)
```
上面的这个例子中，obj1和obj2通过各自的属性相互引用，所有它们的引用计数都不为零，这样就不会被垃圾回收机制回收，造成内存浪费。
引用计数算法其实还有一个比较大的缺点，就是我们需要单独拿出一片空间去维护每个变量的引用计数，这对于比较大的程序，在空间开销还是比较大的。
- 引用计数算法优点：
    * 引用计数为零时，发现垃圾立即回收
    * 最大限度减少程序暂停
- 引用计数算法缺点：
    * 无法回收循环引用的对象
    * 空间开销比较大

### （2）. 标记清除（Mark-Sweep）
- 核心思想：分标记和清除两个阶段完成
- 遍历所有对象找标记活动对象
- 遍历所有对象清除没有标记对象
- 回收相应的空间

标记清除算法的优点是：对比引用计数算法，标记清除算法最大的优点是能够回收循环引用的对象，它也是v8引擎使用最多的算法。
标记清除算法的缺点是：

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4.png)

上图我们可以看到，红色区域是一个根对象，就是一个全局变量，会被标记；而蓝色区域就是没有被标记的对象，会被回收机制回收。这时就会出现一个问题，表面上蓝色区域被回收了三个空间，但是这三个空间是不连续的，当我们有一个需要三个空间的对象，那么我们刚刚被回收的空间是不能被分配的，这就是“空间碎片化”。
### 3. 标记整理（Mark-Compact）
为了解决内存碎片化的问题，提高对内存的利用，引入了标记整理算法。
- 标记整理可以看做是标记清除的增强
- 标记阶段的操作和标记清除一致
- 清除阶段会先执行整理，移动对象位置,将存活的对象移动到一边，然后再清理端边界外的内存。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E6%A0%87%E8%AE%B0%E6%95%B4%E7%90%86%E5%89%8D.png)

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E6%A0%87%E8%AE%B0%E6%95%B4%E7%90%86%E5%90%8E.png)

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E6%A0%87%E8%AE%B0%E6%95%B4%E7%90%86%E5%9B%9E%E6%94%B6%E5%90%8E.png)

标记整理的缺点是：移动对象位置，不会立即回收对象，回收的效率比较慢。

### 增量标记（Incremental Marking）
为了减少全停顿的时间，V8对标记进行了优化，将一次停顿进行的标记过程，分成了很多小步。每执行完一小步就让应用逻辑执行一会儿，这样交替多次后完成标记。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E5%A2%9E%E9%87%8F%E6%A0%87%E8%AE%B0.png)

长时间的GC，会导致应用暂停和无响应，将会导致糟糕的用户体验。从2011年起，v8就将「全暂停」标记换成了增量标记。改进后的标记方式，最大停顿时间减少到原来的1/6。

## v8引擎垃圾回收策略
- 采用分代回收的思想
- 内存分为新生代、老生代
- 针对不同对象采用不同算法
（1）新生代：对象的存活时间较短。新生对象或只经过一次垃圾回收的对象。
（2）老生代：对象存活时间较长。经历过一次或多次垃圾回收的对象。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E6%96%B0%E7%94%9F%E4%BB%A3%E8%80%81%E7%94%9F%E4%BB%A3%E5%AD%98%E5%82%A8.png)

V8堆的空间等于新生代空间加上老生代空间。且针对不同的操作系统对空间做了内存的限制。
| 类型 \ 系统位数 | 64位 | 32位 |
| -----| ---- | ---- |
| 老生代 | 1400MB | 700MB |
| 新生代 | 32MB | 16MB |

限制内存的原因：
1. 针对浏览器来说，这样的内存是足够使用的
2. 针对浏览器的GC机制，经过不断的测试，如果内存再设置大一点，GC回收的时间就会达到用户的感知，会造成感知上的卡顿。

### （1）. 回收新生代对象
回收新生代对象主要采用复制算法（Scavenge 算法）加标记整理算法。而Scavenge 算法的具体实现，主要采用了Cheney算法。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E6%96%B0%E7%94%9F%E4%BB%A3%E5%9B%9E%E6%94%B6form-to.png)

Cheney算法将内存分为两个等大空间，使用空间为From，空闲空间为To。
检查From空间内的存活对象，若对象存活，检查对象是否符合晋升条件，若符合条件则晋升到老生代，否则将对象从 From 空间复制到 To 空间。若对象不存活，则释放不存活对象的空间。完成复制后，将 From 空间与 To 空间进行角色翻转。
##### 对象晋升机制
- 一轮GC还存活的新生代需要晋升
- 当对象从From 空间复制到 To 空间时，若 To 空间使用超过 25%，则对象直接晋升到老生代中。设置为25%的比例的原因是，当完成 Scavenge 回收后，To 空间将翻转成From 空间，继续进行对象内存的分配。若占比过大，将影响后续内存分配。

### （2）. 回收老生代对象
- 回收老生代对象主要采用标记清除、标记整理、增量标记算法，主要使用标记清除算法，只有在内存分配不足时，采用标记整理算法
- 首先使用标记清除完成垃圾空间的回收
- 采用标记整理进行空间优化
- 采用增量标记进行效率优化

##### 新生代和老生代回收对比
- 新生代由于占用空间比较少，采用空间换时间机制
- 老生代区域空间比较大，不太适合大量的复制算法和标记整理，所以最常用的是标记清除算法，为了就是让全停顿的时间尽量减少

## 内存泄漏识别方法
我们先写一段比较消耗内存的代码
```
<button class="btn">点击</button>

<script>
    const btn = document.querySelector('.btn')
    const arrList = []

    btn.onclick = function() {
        for(let i = 0; i < 100000; i++) {
            const p = document.createElement('p')
            // p.innerHTML = '我是一个p元素'
            document.body.appendChild(p)
        }

        arrList.push(new Array(1000000).join('x'))
    }
</script>
```
使用浏览器的Performance来监控内存变化

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E7%9B%91%E6%8E%A7%E5%89%8D.png)

点击录制，然后我们操作们感觉消耗性能的操作，操作完成之后，点击stop停止录制

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201008221429100-838980332.png)

然后我们看一看是那些地方引起了内存的泄漏，我们只需要关注内存即可

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201008221600994-165015063.png)

可以看到内存在短时间消耗的比较快，下降的小凹槽，就是浏览器在进行垃圾回收

## 性能优化
- 1.避免使用全局变量
    * 全局变量会挂载在window下
    * 全局变量至少有一个引用计数
    * 全局变量存活更久，但是持续占用内存
在明确数据作用域的情况下，尽量使用局部变量

- 2.减少判断层级
```
function doSomething(part, chapter) {
    const parts = ['ES2016', '工程化', 'Vue', 'React', 'Node']

    if (part) {
        if (parts.includes(part)) {
            console.log('属于当前课程')
            if (chapter > 5) {
                console.log('您需要提供 VIP 身份')
            }
        }
    } else {
        console.log('请确认模块信息')
    }
}

doSomething('Vue', 6)

// 减少判断层级
function doSomething(part, chapter) {
    const parts = ['ES2016', '工程化', 'Vue', 'React', 'Node']

    if (!part) {
        console.log('请确认模块信息')
        return
    }

    if (!parts.includes(part)) return
    console.log('属于当前课程')

    if (chapter > 5) {
        console.log('您需要提供 VIP 身份')
    }
}

doSomething('Vue', 6)
```

- 3.减少数据读取次数
对于频繁使用的数据，我们要对数据进行缓存
```
<div id="skip" class="skip"></div>

<script>
    var oBox = document.getElementById('skip')

    // function hasEle (ele, cls) {
    //     return ele.className === cls
    // }

    function hasEle (ele, cls) {
        const className = ele.className
        return className === cls
    }

    console.log(hasEle(oBox, 'skip'))
</script>
```

- 4.减少循环体中的活动
```
var test = () => {
    var i
    var arr = ['maoxiaoxing', 25, '能被看见的努力，都是肤浅的努力']
    for(i = 0; i < arr.length; i++) {
        console.log(arr[i])
    }
}

// 优化后，将arr.length单独提出，防止每次循环都获取一次
var test = () => {
    var i
    var arr = ['maoxiaoxing', 25, '能被看见的努力，都是肤浅的努力']
    var len = arr.length
    for(i = 0; i < len; i++) {
        console.log(arr[i])
    }
}
```

- 5.事件绑定优化
```
<ul class="ul">
    <li>毛小星</li>
    <li>25</li>
    <li>能看见的努力，都是肤浅的努力</li>
</ul>

<script>
    var list = document.querySelectorAll('li')
    function showTxt(ev) {
        console.log(ev.target.innerHTML)
    }

    for (item of list) {
        item.onclick = showTxt
    }

    // 优化后
    function showTxt(ev) {
        var target = ev.target
        if (target.nodeName.toLowerCase() === 'li') {
            console.log(ev.target.innerHTML)
        }
    }

    var ul = document.querySelector('.ul')
    ul.addEventListener('click', showTxt)
</script>
```
- 6.避开闭包陷阱
```
<button class="btn">点击</button>

<script>
    function foo() {
        let el = document.querySelector('.btn')
        el.onclick = function() {
            console.log(el.className)
        }
    }
    foo()

    // 优化后
    function foo1() {
        let el = document.querySelector('.btn')
        el.onclick = function() {
            console.log(el.className)
        }
        el = null // 将el置为 null 防止闭包中的引用使得不能被回收
    }
    foo1()
</script>
```


## 参考资料
- [拉勾教育（推荐）](https://kaiwu.lagou.com/)
- [阮一峰：JavaScript 内存泄漏教程](http://www.ruanyifeng.com/blog/2017/04/memory-leak.html)
- [深入理解V8的垃圾回收原理](https://www.jianshu.com/p/b8ed21e8a4fb)
- [JavaScript中的垃圾回收和内存泄漏](https://github.com/ljianshu/Blog/issues/65)
