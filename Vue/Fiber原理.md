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
