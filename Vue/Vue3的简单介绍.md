# Vue3的简单介绍

## Vue3和Vue2的区别

### 源码的组织方式

- 使用 TypeScript 重写

首先为了提升代码的可维护性，Vue3.0 抛弃了 Flow 类型注释，而是全部采用了更加严格的 TypeScript 重写，大型项目的开发都推荐使用类型化的语言，这样可以在编码的过程中帮助我们检查类型化的问题，比如给函数传参，如果类型不匹配，会有相应的类型提示。当然了，你可以不在你的项目中使用 TypeScript，而是使用 JavaScript，Vue3.0 也是完全支持的

- 使用了 monorepo 管理项目结构

monorepo 使用一个项目来管理多个包，把不同功能的代码放在不同的包中去管理，这样每个功能模块的划分都很明确，模块之间的依赖关系也很明确，并且每个功能模块都可以单独测试、单独发布、单独使用。
我们可以来看看源码中的 packages 的目录结构：

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210419231009340-1975656231.png)

> compiler-core 和平台无关的编译器
> compiler-dom 浏览器平台下的编译器，依赖于 compiler-core
> compiler-sfc sfc 的意思是 single-file-component 单文件组件，依赖于 compiler-core 和 compiler-dom
> compiler-ssr 服务器端渲染的编译器，依赖于 compiler-dom
> reactivity 数据响应式系统，可以独立使用
> runtime-core 和平台无关的运行时
> runtime-dom 针对浏览器的运行时，处理原生的 DOM 的 API 和 事件等
> runtime-test 为测试编写的轻量级的运行时，将 DOM 渲染成 js 对象，可以运行在所有的 js 环境里，可以验证 DOM 是否渲染正确，还可以用来序列化 DOM、触发 DOM 事件、记录某次 DOM 更新操作
> server-renderer 服务器端渲染
> shared vue内部使用的一些内部api
> size-check 在 tree-sharking 后检查包的大小，这是一个私有的包，不会发布到 npm 上
> template-explorer 在浏览器里运行的实时编译组件，会输出 render 函数
> vue 构建完整版的 Vue，依赖于 compiler 和 runtime

### Composition API

Vue3.0 的代码虽然全部重写，但是 90% 以上的 API 依然兼容 2.x。并且根据社区的反馈，增加了 Composition API，这是为了解决 Vue2.x 在开发大型项目时，处理超大组件使用 Options API 不好拆分和重用的问题。

#### 1. 怎样学习 Composition API？

学习 Composition API 的最好的方式就是查看官方的 RFC（Request For Comments），Vue2 升级到 Vue3 的大的变动就是通过 RFC 的机制去确认的，首先管方给出一些提案，然后收集社区的反馈并讨论，最后确认。
RFC官方地址：https://github.com/vuejs/rfcs
Composition API 文档：https://v3.vuejs.org/guide/composition-api-introduction.html#why-composition-api

#### 2. Composition API 的设计动机

- Options API

Vue2.x 中使用 Options API，即包含一个描述组件选项（data、methods、props等）的对象，Options API 在开发复杂组件时，同一个功能逻辑的代码会被拆分到不同选项。

下面我们来看一个案例

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210420225737001-178248134.png)

这个例子的代码很简单，就是当鼠标移动的时候，将鼠标的位置展示到页面上。但是当我们想要添加新功能例如搜索功能的时候，我们就需要在多个 option 中添加我们的代码，这就显得有些麻烦。而且 Options API 很难提取一些公共代码，虽然我们可以使用 mixin 去提取一些课重用的逻辑，但是 mixin 的使用也有很多问题，例如命名冲突和数据来源不清晰等问题。

- Composition API

Composition API 是 Vue3.0 新增的 API，是一组基于函数的 API，可以更灵活的组织组件的逻辑。

我们可以使用 Composition API 来重写一下我们刚才的逻辑

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210420230642966-1415541883.png)

上面的代码我们可以看到，我们将所有重复的功能都封装在 useMousePosition 这个函数中，当我们需要使用的时候，我们只需要在 setup 这个函数中去调用即可。这样我们在查看某个逻辑的时候，只需要查看某个函数即可，不用再在各个 option 中来回查找了。

下面我们来一张 Options API 和 Composition API 的对比图：

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210420231055321-385741458.png)

相同颜色的代表同样的功能，我们可以看到 Options API 中同样的功能被拆分成了不同的代码块，当组件的功能比较复杂的时候，同一逻辑的代码被拆分到不同的位置，开发者就需要耗费一定的精力去组织这些逻辑。而 Composition API 中相同的逻辑都在同一个代码块，这样在处理组件的时候会比较清晰。当然了 Composition API 只是 Vue3 新增的一组 API，你完全可以将 Options API 和 Composition API 结合起来使用，更加灵活的实现你的逻辑。


### 性能提升

Vue3.0 中使用 Proxy 重写了响应式原理，并且对编译器做了优化，重写了虚拟 DOM，从而让渲染和 update 的性能都有了大幅度的提升。另外官方介绍服务端渲染的性能也提升了两到三倍。

#### 1. 响应式系统升级

Vue2.x 中的响应式原理的核心是 Object.defineProperty，在 data 初始化的时候会遍历所有成员对数据进行递归响应式处理，即使你没有使用这个属性的时候也会进行响应式处理；
而 Vue3.0 中使用 Proxy 对象重写响应式系统，本身 Proxy 的性能就会比 Object.defineProperty 好，另外 Proxy 代理的对象可以拦截属性的访问、赋值、新增、删除等操作，而且 Proxy 可以监听数组的索引和 length 属性。Proxy 代理的对象只有在访问某个属性的时候才会触发代理，而不是像 Vue2.x 中初始化就递归处理所有属性，使用 Proxy 默认就可以监听动态添加的属性，而在 Vue2.x 中只能使用 Vue.$set 方法去为对象动态添加属性。所以 Vue3.0 中使用 Proxy 重写响应式系统大大提升了整个框架的性能。 

#### 2. 编译优化

我们先来通过一个组件来回顾 Vue2.x 的编译执行过程

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210421224516062-1176756196.png)

在 Vue2.x 中模板首先会编译成 render 函数，这个过程一般是在构建的过程中完成的，在编译的时候，会编译静态根节点和静态节点，静态根节点必须要求有一个静态子节点，当组件发生变化时会触发 Watcher，然后触发 Watcher 的 update 函数，最终去执行虚拟 DOM 的 patch 操作，diff 的过程中会去遍历比较所有的虚拟 DOM ，通过双指针的算法去比较新旧 DOM，找到差异然后更新到真实 DOM 上。Vue2.x 中渲染的最小单位是组件，通过标记静态根节点，在 diff 的过程中跳过静态根节点，优化 diff 操作，但是静态节点还是需要 diff，这个过程没有被优化。
Vue3.0 为了提升性能，会标记和提升所有的静态节点，diff 的时候只需要对比动态节点内容。另外在 Vue3.0 中引入了 Fragments 的特性，模板中不需要放一个唯一的根节点，模板中可以直接放文本内容，也可以放很多同级的 HTML 标签，但是在 vscode 中需要升级你的 vetur 插件，否则编译器会报错。

下面我们使用官方提供的 [explorer](https://vue-next-template-explorer.netlify.app/) 插件来看看模板做了哪些优化

template：

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210422224812050-1702766284.png)

template 被编译后：

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210422224859120-635655396.png)

在 Vue3.0 中，如果模板的最外层没有根节点的话，就会创建一个 Fragment 片段，操作 Fragment 的代价是很小的，相对于操作 DOM 耗费的性能非常小。
我们能看到 _hoisted_1、_hoisted_2、_hoisted_3 这样的静态节点被提升到了 render 的外层，这样只有在页面初始化的时候会创建一次，下次视图重新渲染后直接引用静态节点即可，这样的静态节点也不会参与 diff 的过程。
在编译的后的图示的第 18 行代码，我们能看到后面的注释是 9,9 是 Vue3.0 中引入的 patch-flag 的概念，代表了当前的文本和 props 是动态内容，同时还记录的动态绑定的属性是 id，当 diff 的时候，只会检查动态绑定的文本和 id 属性是否发生变化，这样大大提升了虚拟 DOM diff 的性能。
在第 20 行代码中，我们能看到有一个 cache，这个 cache 缓存了绑定的函数，在首次渲染的时候，会缓存这个函数，当再次调用这个函数的时候，会从缓存中读取，这样也避免了很多不必要的更新。

#### 3. 源码体积的优化

Vue3.0 中移除了一些不常用的 API，例如：inline-template、filter（我本人挺喜欢用的[手动狗头]）等。
Vue3.0 中对 Tree-shaking 的支持更好，Tree-shaking 依赖 ES Module，也就是 ES6 中的模块化语法（import、export），通过编译的静态分析，找到没有引入的模块，在打包的时候过滤掉，让打包的体积更小。Vue3.0 在设计之初，就考虑到了 Tree-shaking，内置的组件比如 transition、keep-live，内置的指令比如 v-model 都是按需引入的，另外 Vue3.0 的其他很多 API 都是支持 Tree-shaking 的，只有核心模块和你使用了的才会打包，不使用就不会打包。

### Vite

随着 Vue3.0 的发布，管方也发布了新的构建工具 Vite，在开发和测试阶段 Vite 不需要打包，而可以直接运行项目，大大提升了开发的效率。
我们再来介绍 Vite 之前，先来看看 ES Module 规范，因为 Vite 是基于 ES Module 规范的，如果你对 ES Module 规范不太了解，可以看看我之前写的 [模块化开发](https://www.cnblogs.com/bejamin/p/13982677.html)。
现代浏览器基本都支持 ES Module （IE不支持【手动可恶！】），ES Module 有一些特性：
- 通过下面的方式加载模块

```js
<script type="module" src="..."></script>
```

- 支持模块的 script 默认延迟加载
  * 类似于 script 标签设置 defer
  * 在文档解析完成后，触发 DOMContentLoaded 事件前执行

Vite 之所以快，就是因为 Vite 直接使用 ES Module 去加载模块，在开发模式下不需要打包就可以直接运行；而 Vue-cli 开发模式下必须对项目打包才可以运行。

#### Vite 特点

- 快速冷启动

因为不需要打包，所以 Vite 可以利用 ES Module 的特性快速启动项目

- 按需编译

代码是按需编译的，因此只有代码在当前运行环境需要加载的时候才会编译，你不需要在开启开发服务器的时候等待所有项目文件打包，当项目比较大的时候，这个节省的时间就比较明显。

- 模块热更新

Vite 支持模块热更新，并且模块热更新的性能与模块总数无关，无论你有多少模块，hmr 的速度始终比较快。

Vite 在生产环境下使用 Rollup 打包，Rollup 基于浏览器原生的 ES Module 的方式打包，不需要使用 babel 把 import 转换成 require，以及相应的辅助函数，因此打包的体积会比 webpack 打包的体积小很多。

#### Vite 创建项目

- Vite 创建 vue 项目

```bash
npm init vite-app <project-name>
cd <project-name>
npm install
npm run dev
```

- 基于模板创建项目

```bash
npm init vite-app -- template react
```

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210425224748581-386153013.png)

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210425224444588-768371206.png)

项目运行起来，我们可以右键查看网页源代码，就能看到 Vue 的入口文件 main.js 确实是通过 ES Module 方式引入的。

我们再来看看控制台 netWork 请求到的 App.vue 的解析结果

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210425225426832-2068268611.png)

首先 Vite 开启的服务器会劫持以 .vue 结尾的请求，会把以 .vue 结尾的文件解析成 js 文件，并将响应头的 Content-Type 设置为 application/javascript，目的就是为了告诉浏览器，现在我给你发送的是一个 javascript 脚本

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210425225517275-2073245084.png)

这次我们来看看 App.vue 又被请求了一次，不同的是这次带了一个 type=template 的参数，这次请求服务器之后，服务器会把 App.vue 这个单文件组件通过 Vue3.0 中的模块 compiler-sfc 给编译成 render 函数，这基本就是 Vite 的工作原理。
