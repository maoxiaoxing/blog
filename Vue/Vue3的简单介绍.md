# Vue3的简单介绍

## Vue3和Vue2的区别

### 源码的组织方式

- 使用 TypeScript 重写

首先为了提升代码的可维护性，Vue3.0 抛弃了 Flow 类型注释，而是全部采用了更加严格的 TypeScript 重写，大型项目的开发都推荐使用类型化的语言，这样可以在编码的过程中帮助我们检查类型化的问题，比如给函数传参，如果类型不匹配，会有相应的类型提示。当然了，你可以不在你的项目中使用 TypeScript，而是使用 JavaScript，Vue3.0 也是完全支持的

- 使用了 monorepo 管理项目结构

monorepo 使用一个项目来管理多个包，把不同功能的代码放在不同的包中去管理，这样每个功能模块的划分都很明确，模块之间的依赖关系也很明确，并且每个功能模块都可以单独测试、单独发布、单独使用。
我们可以来看看源码中的 packages 的目录结构：

![](https://img2020.cnblogs.com/blog/1575596/202104/1575596-20210419231009340-1975656231.png)

> compiler-core 和平台无关的编译器
> compiler-dom 浏览器平台下的编译器，依赖于 compiler-core
> compiler-sfc sfc 的意思是 single-file-component 单文件组件，依赖于 compiler-core 和 compiler-dom
> compiler-ssr 服务器端渲染的编译器，依赖于 compiler-dom
> reactivity 数据响应式系统，可以独立使用
> runtime-core 和平台无关的运行时
> runtime-dom 针对浏览器的运行时
> runtime-test 为测试编写的轻量级的运行时
> server-renderer 服务器端渲染
> shared vue内部使用的一些内部api
> size-check 在 tree-sharking 后检查包的大小，这是一个私有的包，不会发布到 npm 上
> template-explorer 在浏览器里运行的实时编译组件，会输出 render 函数
> vue 构建完整版的 Vue，依赖于 compiler 和 runtime



### Composition API

Vue3.0 的代码虽然全部重写，但是 90% 以上的 API 依然兼容 2.x。并且根据社区的反馈，增加了 Composition API，这是为了解决 Vue2.x 在开发大型项目时，处理超大组件使用 Options API 不好拆分和重用的问题。

### 性能提升

Vue3.0 中使用 Proxy 重写了响应式原理，并且对编译器做了优化，重写了虚拟 DOM，从而让渲染和 update 的性能都有了大幅度的提升。另外官方介绍服务端渲染的性能也提升了两到三倍。

### Vite

随着 Vue3.0 的发布，管方也发布了新的构建工具 Vite，在开发和测试阶段 Vite 不需要打包，而可以直接运行项目，大大提升了开发的效率。
