# Rollup的简单使用
------------------------

## 1. Rollup 概述
![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/rollup%E4%BB%8B%E7%BB%8D.png)

Rollup 是一款优秀的模块化打包器，它的作用与 Webpack 类似，但是相对于 Webpack，Rollup 更加小巧，它仅仅是一款模块化打包器，它并没有其他功能。例如 Webpack 有模块热替换功能， 而 Rollup 对这样的功能就没有很好的支持。Rollup 的诞生并不是为了和 Webpack 去竞争，而只是为了提供一个充分利用 ESM 各项特性的高效打包器。如果你同时看过 Rollup 和 Webpack 打包后的文件后，你就会发现 Rollup 的打包文件简洁的多。

## 2. 快速上手 Rollup
首先我们在根目录下初始化一个 package.json，然后将 rollup 作为我们的开发依赖下载
```
yarn init -y

yarn add rollup --dev
```
然后我们定义一个 src 的文件夹，在 src 文件夹下面定义两个文件 index.js 和 bar.js
```
// bar.js
export const foo = 'bar'
```
```
// index.js
import { foo } from './bar'

const baz = 'aaa'
console.log('hello Rollop')
console.log(foo)
```
我们可以执行下面的命令打包我们的文件
```
yarn rollup --format iife ./src/index.js --file dist/bundle.js
```
--format 是打包文件的格式，我们选择最贴近浏览器的 iife 模式，--file 是输出的文件位置。
打包结果如下
```
// dist/bundle.js
(function () {
	'use strict';

	const foo = 'bar';

	console.log('hello Rollop');
	console.log(foo);

}());
```
可以看到 Rollup 的打包结果非常简洁，它不像 Webpack 有那么多的注释，Rollup 的输出结果几乎没有任何多余的代码，它就是将我们各个模块的代码按照顺序拼接到一个函数中去。而且我们可以看到，它不会将我们多余的代码打包进去，因为 Rollup 默认会开启 tree shinking 模式。

## 3. Rollup 的配置文件
Rollup 同样支持以配置文件的方式去配置我们打包文件的各项参数，我们需要在根目录下定义一个 rollup.config.js 的文件。
```
export default {
    input: 'src/index.js',
    output: {
      file: 'dist/bundle.js',
      format: 'iife'
    }
}
```
这样我们就不用像上面那样去在命令行去指定那么多的参数了
```
yarn rollup --config
```
我们同样能输出 dist 文件

- 使用插件

如果我们在项目中有更高级的需求，比如我们想要去加载其他类型的文件，我们可以使用插件去扩展它们，而且 Rollup 只能支持用插件的方式去扩展，而不像 Webpack 那样有丰富的 loader 和 plugin。例如我们想在我们的项目使用 json 文件，我们可以使用 rollup-plugin-json 的插件
```
yarn add rollup-plugin-json --dev
```
我们需要在 rollup.config.js 中做出如下配置
```
import json from 'rollup-plugin-json'

export default {
  input: 'src/index.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife'
  },
  plugins: [
    json()
  ]
}
```
这样我们就可以在我们文件中使用 json 文件中的属性了

- 使用 Rollup 加载 NPM 模块
我们下载 rollup-plugin-node-resolve 作为我们的开发依赖
```
yarn add rollup-plugin-node-resolve --dev
```
之后我们在 rullup.config.js 做出配置
```
import json from 'rollup-plugin-json'
import resolve from 'rollup-plugin-node-resolve'

export default {
  input: 'src/index.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife'
  },
  plugins: [
    json(),
    resolve()
  ]
}
```
这样我们就可以直接引入 node_modules 中的模块了
```
import _ form 'lodash'
```

- 在 Rollup 中加载 commonjs 模块

如果我们想在 Rollup 加载 commonjs 规范的 js 代码，我们可以使用 rollup-plugin-commonjs 这个插件
```
yarn add rollup-plugin-commonjs --dev
```
之后我们在 rollup.config.js 中做出配置
```
import json from 'rollup-plugin-json'
import resolve from 'rollup-plugin-node-resolve'
import commonjs from 'rollup-plugin-commonjs'

export default {
  input: 'src/index.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife'
  },
  plugins: [
    json(),
    resolve(),
    commonjs()
  ]
}
```
后面我们就可以使用 commonjs 规范去编写我们的代码了

- 代码拆分

Rollup 支持我们使用 代码拆分（code-spliting）的方式去打包我们的代码，所以我们在配置文件中不能指定输出文件，输出方式也不能使用输出一个函数的 iife 模式
```
export default {
  input: 'src/index.js',
  output: {
    // file: 'dist/bundle.js',
    // format: 'iife'
    dir: 'dist',
    format: 'amd'
  }
}
```
下面我们使用按模块的按需加载
```
// logger.js
export const log = msg => {
  console.log('---------- INFO ----------')
  console.log(msg)
  console.log('--------------------------')
}
```
```
import('./logger').then(({ log }) => {
  log('code splitting~')
})
```
之后我们使用打包命令打包，这样 dist 就会有两个打包文件

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/rollup%E6%89%93%E5%8C%85%E6%88%AA%E5%9B%BE.png)

- 多入口打包

Rollup 同样支持多入口打包，我们可以使用数组或者对象的形式去配置输出文件
```
export default {
  // input: ['src/index.js', 'src/album.js'],
  input: {
    foo: 'src/index.js',
    bar: 'src/album.js'
  },
  output: {
    dir: 'dist',
    format: 'amd'
  }
}
```
接下来我们就可以使用打包命令来打包我们的代码了，值得注意的是，使用 amd 的打包方式打包的代码不能直接在 html 文件中引用，而是需要借助 require.js 这样的库去辅助我们。
```
<script src="https://unpkg.com/requirejs@2.3.6/require.js" data-main="foo.js"></script>
```

## 4. 总结

通过上面对 Rollup 的简单使用，我们可以发现，Rollup 还是有一定的优点的
- 输出结果更加扁平
- 自动移除未引用代码
- 打包结果依然完全可读

但是它的缺点也同样明显
- 加载非 ESM 的第三方模块比较复杂
- 模块最终都被打包到一个函数中，无法实现 HMR
- 浏览器中，代码拆分功能依赖 AMD 库

我们可以看出来，如果我们在开发一个应用程序，我们就需要大量加载第三方模块库，并且需要代码拆分和热模块替换等功能，这时显然 Webpack 这样的打包工具更适合我们；但是如果我们在开发框架或者一些公共库的时候，我们就可以使用 Rollup，因为我们不用加载大量的第三方库。
简单来说 Webpack 大而全， Rollup 小而美，但是现在社区中的大部分人更加希望 Webpack 和 Rollup 这样的打包工具共存并互相借鉴，我们希望更加专业工具去做更加专业的事
