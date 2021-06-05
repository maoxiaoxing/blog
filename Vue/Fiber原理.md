# Fiber 原理

## 在 Fiber 出现之前 React 存在什么问题

在 React 16 之前的版本对比更新 VirtualDOM 的过程是采用 Stack 架构实现的，也就是循环加递归。这种对比方式有一个问题，就是一旦任务开始进行就无法中断，如果应用中的组件数量庞大，Virtual DOM 的层级比较深，主线程被长期占用，直到整棵 VirtualDOM 树比对更新完成之后主线程才能被释放，主线程才能执行其他任务。这就会导致一些用户交互，动画等任务无法立即得到执行，页面就会产生卡顿, 非常的影响用户体验。
核心问题：递归无法中断，执行任务耗时长，JavaScript 是单线程的，和 Native GUI 互斥，比较 VirtualDOM 的过程中无法执行其他任务，导致任务延迟页面卡顿，用户体验差。

## Stack 架构的简单实现

我们来实现一个简单的获取 jsx，然后将 jsx 转换成 DOM ，然后添加到页面中的过程

```js
const jsx = (
  <div id="a1">
    <div id="b1">
      <div id="c1"></div>
      <div id="c2"></div>
    </div>
    <div id="b2"></div>
  </div>
)

function render(vdom, container) {
  // 创建元素
  const element = document.createElement(vdom.type)
  // 为元素添加属性
  Object.keys(vdom.props)
    .filter(propName => propName !== "children") // 过滤 children 属性
    .forEach(propName => (element[propName] = vdom.props[propName]))
  // 递归创建子元素
  if (Array.isArray(vdom.props.children)) {
    vdom.props.children.forEach(child => render(child, element))
  }
  // 将元素添加到页面中
  container.appendChild(element)
}

render(jsx, document.getElementById("root"))
```

![](https://img2020.cnblogs.com/blog/1575596/202106/1575596-20210604083840102-1239227374.png)

可以看到，jsx 代码被转换成了真实的 DOM 添加到了页面中

## Fiber 如何解决性能问题的思路

1. 在 Fiber 架构中 React 放弃了递归调用，采用循环来模拟递归，因为循环可以随时被中断。
2. Fiber 将大的渲染任务拆分成一个个小任务
3. React 使用 requestIdleCallback 去利用浏览器的空闲时间去执行小任务，React 在执行一个任务单元后，查看是否有其他高优先级的任务，如果有，放弃占用线程，先执行优先级高的任务。

## requestIdleCallback

我们先来看看 requestIdleCallback 在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback) 上的解释

> window.requestIdleCallback()方法将在浏览器的空闲时段内调用的函数排队。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间timeout，则有可能为了在超时前执行函数而打乱执行顺序。

### requestIdleCallback 的语法

requestIdleCallback 接收两个参数，一个名为 IdleDeadline 的回调函数，一个是可选参数

![](https://img2020.cnblogs.com/blog/1575596/202106/1575596-20210605083507552-1546195424.png)

IdleDeadline 参数上有一个 timeRemaining() 的方法，返回一个时间 [DOMHighResTimeStamp](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMHighResTimeStamp), 并且是浮点类型的数值，它用来表示当前闲置周期的预估剩余毫秒数。如果idle period已经结束，则它的值是0。你的回调函数(传给requestIdleCallback的函数)可以重复的访问这个属性用来判断当前线程的闲置时间是否可以在结束前执行更多的任务。

### requestIdleCallback 的作用

浏览器的页面都是通过引擎一帧一帧绘制出来的，当每秒绘制的帧数达到 60 的时候，页面就是流畅的，玩过 fps 游戏的都知道，当这个帧数小于 60 的时候，人的肉眼就能感知出来卡顿。一秒 60 帧，每一帧分到的时间就是 1000/60 ≈ 16 ms，如果每一帧执行的时间小于 16 ms，就说明浏览器有空余时间，那么能不能通过浏览器的空余时间去处理任务呢，这样就不用一直等待主任务执行完了，requestIdleCallback 就是利用浏览器的空余时间去执行任务的。

上面说了一堆，有的人可能已经懵了，你别废话，直接上代码给我一个示例就行了。诶，那我们就用下面的例子看看 requestIdleCallback 到底有什么神奇之处

```html
<style>
  #box {
    padding: 20px;
    background: palegoldenrod;
  }
</style>

<!-- body -->
<div id="box"></div>
  <button id="btn1">执行计算任务</button>
  <button id="btn2">更改背景颜色</button>
<script>
  const box = document.getElementById('box')
  const btn1 = document.getElementById('btn1')
  const btn2 = document.getElementById('btn2')

  let number = 999999
  let value = 0

  function calc() {
    while (number > 0) {
      value = Math.random() < 0.5 ? Math.random() : Math.random()
      console.log(value)
      number--
    }
  }

  btn1.onclick = function () {
    calc()
  }

  btn2.onclick = function () {
    box.style.background = 'green'
  }
</script>
```

上面的代码，我们通过一个很长的循环创建随机数，增加浏览器的计算量，你可以通过本地的 ide 试试这个 demo，就会发现当你点击 执行计算任务 后，再点击 更改背景颜色 按钮后，box 的颜色不会立马改变，而是会等待几秒后才发生改变（如果电脑性能差，可能会更慢）。这时因为 native GUI 和 v8引擎的渲染是互斥的，所以页面渲染会有一些延迟。

```html
<style>
  #box {
    padding: 20px;
    background: palegoldenrod;
  }
</style>
<!-- body -->
<div id="box"></div>
<button id="btn1">执行计算任务</button>
<button id="btn2">更改背景颜色</button>
<script>
  const box = document.getElementById('box')
  const btn1 = document.getElementById('btn1')
  const btn2 = document.getElementById('btn2')

  let number = 999999
  let value = 0

  function calc(deadline) {
    while (number > 0 && deadline.timeRemaining() > 0) {
      value = Math.random() < 0.5 ? Math.random() : Math.random()
      console.log(value)
      number--
    }
    requestIdleCallback(calc)
  }

  btn1.onclick = function () {
    requestIdleCallback(calc)
  }

  btn2.onclick = function () {
    box.style.background = 'green'
  }
</script>
```

