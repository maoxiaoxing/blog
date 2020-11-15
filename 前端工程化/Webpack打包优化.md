# Webpack打包优化
--------------------------------

Webapck 4 之后默认为我们做了很多配置项，内部开启了很多优化功能。对于开发人员，这种开箱即用的体验显然是很好的，但是同时也会导致我们忽略了很多需要学习的东西，一旦出现什么问题的时候，我们就无从下手了，下面我们就来看一下主要的优化配置项。

## DefinePlugin

DefinePlugin 是用来为我们的代码来注入全局成员的，在 production 模式下，这个插件就会默认开启。它会在我们的环境中注入了一个 process.env.NODE_ENV 这样一个环境变量，我们可以通过这个环境变量去判断运行环境，从而去执行一些相应的逻辑。
```
const webpack = require('webpack')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.DefinePlugin({
      // 值要求的是一个代码片段
      API_BASE_URL: JSON.stringify('https://api.example.com')
    })
  ]
}
```
这样我们就可以直接在环境中使用 API_BASE_URL 这个变量了
```
// main.js
console.log(API_BASE_URL)
```

## Tree-shaking

Tree-shaking 顾名思义就是摇树，伴随着摇这个动作，我们会将树上的枯树枝和枯树叶摇下来。而在我们的项目中 Tree-shaking 会将我们代码中没有引用的部分去掉，Tree-shaking 并不是某一个配置选项，它是一组功能搭配使用的效果。我们可以使用 optimization 去开启一些功能，optimization 就是优化的意思，下面我们来看看怎样去配置它
```
module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js'
  },
  optimization: {
    // 模块只导出被使用的成员
    usedExports: true,
    // 尽可能合并每一个模块到一个函数中
    concatenateModules: true,
    // 压缩输出结果
    // minimize: true
  }
}
```
我们可以将 usedExports 想象成它就是去标记"枯树叶"的，而 minimize 就是去摇下这些枯树叶的。而 concatenateModules 将所有的代码都尽可能的合并到一个函数中去，这样既提升了运行效率，又减少了代码的体积。这个特性又被称为 Scope Hoisting，这时 Webpack3 中提出的一个特性。

- Tree-shaking 与 babel

由于 Webpack 的发展比较快，所以我们在找资料的时候，找到的资料并不一定适用于我们当前的版本，Tree-shaking 更是如此，很多资料中都显示如果我们使用的 babel-loader 的话，就会导致 Tree-shaking 失效。因为 Tree-shaking 使用的前提就是必须使用 ES Modules 规范去组织我们的代码，而 @babel/preset-env 这个插件内部就会将 ES Modules 的代码转换为 commonjs 代码的方式，所以 Tree-shaking 就不能生效。但是实际你同时开启两者的话，Tree-shaking 还是会生效的，因为 @babel/preset-env 这个插件最新的版本内部将 ES Modules 转换为 commonjs 关掉了。
```
module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              // 如果 Babel 加载模块时已经转换了 ESM，则会导致 Tree Shaking 失效
              // ['@babel/preset-env', { modules: 'commonjs' }]
              // ['@babel/preset-env', { modules: false }]
              // 也可以使用默认配置，也就是 auto，这样 babel-loader 会自动关闭 ESM 转换
              ['@babel/preset-env', { modules: 'auto' }]
            ]
          }
        }
      }
    ]
  },
  optimization: {
    // 模块只导出被使用的成员
    usedExports: true,
    // 尽可能合并每一个模块到一个函数中
    // concatenateModules: true,
    // 压缩输出结果
    // minimize: true
  }
}
```

## sideEffects

Webpack4 中还新增了一个叫 sideEffects 的新特性，它允许我们去标识我们的代码是否有副作用，从而为 Tree shaking 提供更大的压缩空间。副作用就是模块去执行时除了导出成员之外所做的事情，sideEffects 一般只有我们在去开发一个 npm 模块的时候才会去使用，那是因为官网将 sideEffects 和 Tree shaking 混到了一起，所以很多人误认为它们两个是因果关系，其实它们两个的关系不大。
当我们去封装组件的时候，我们一般会将所有的组件都导入在一个文件中，然后通过这个文件集体导出，但是其他文件引入这个文件的时候，就会将这个导出文件的所有组件都引入
```
// components/index.js
export { default as Button } from './button'
export { default as Heading } from './heading'
```
```
// main.js
import { Button } from './components'
document.body.appendChild(Button())
```
这样 Webpack 在打包的时候，也会将 Heading 组件打包到文件中，这时 sideEffects 就能解决这个问题
```
module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js'
  },
  optimization: {
    sideEffects: true,
  }
}
```
同时我们在 packag.json 中导入将没有副作用的文件关闭，这样就不会将无用的文件打包到项目中了
```
{
  "name": "side-effects",
  "version": "0.1.0",
  "main": "index.js",
  "author": "maoxiaoxing",
  "license": "MIT",
  "scripts": {
    "build": "webpack"
  },
  "devDependencies": {
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.9"
  },
  "sideEffects": false
}
```
使用 sideEffects 的需要注意的是，我们的代码中真的没有副作用，如果有副作用的代码，我们就不能去这样配置了。
```
// exten.js
// 为 Number 的原型添加一个扩展方法
Number.prototype.pad = function (size) {
  // 将数字转为字符串 => '8'
  let result = this + ''
  // 在数字前补指定个数的 0 => '008'
  while (result.length < size) {
    result = '0' + result
  }
  return result
}
```
例如我们在 extend.js 文件中为 Number 的原型添加一个方法，我们并没有向外导出成员，只是基于原型扩展了一个方法，我们在其他文件导入这个 extend.js
```
// main.js
// 副作用模块
import './extend'
console.log((8).pad(3))
```
如果我们还标识项目中所有模块没有副作用的话，这个添加在原型的方法就不会被打包进去，在运行中肯定会报错，还有就是我们在代码中导入的 css 模块，也都是副作用模块，我们就可以在 package.json 中去标识我们的副作用模块
```
{
  "name": "side-effects",
  "version": "0.1.0",
  "main": "index.js",
  "author": "maoxiaoxing",
  "license": "MIT",
  "scripts": {
    "build": "webpack"
  },
  "devDependencies": {
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.9"
  },
  "sideEffects": [
    "./src/extend.js",
    "*.css"
  ]
}
```
这样标识的有副作用的模块也会被打包进来。

## Webpack 代码分割（Code Splitting）

模块化的优势固然很明显，但是也存在一些弊端，就是在我们的项目中所有的代码都会被打包到一起，如果我们的项目过大的话，那么我们的打包结果就会特别大。但是实际的情况是，我们在首次加载的时候，并不是所有的模块都是必须加载的，但是这些模块又被打包到一起，所以一方面在浏览器运行的时候会慢，一方面也会浪费一些流量和带宽。所以合理的方式就是将我们的代码按照一定的规则打包到多个 js 文件中去，做分包处理、按需加载，这样我们就会大大提高我们的应用的响应速率。那么有人可能会想到 Webpack 不就是将我们代码中散落的代码合并到一个函数中去执行，从而去提高效率，这里为什么又要做分包处理，不是自相矛盾吗？其实任何事情都是物极必反，Webpack 做代码合并是因为我们在开发中往往模块化颗粒度太细，所以 Webpack 必须将很多代码合并到一起，但是如果总体代码量过大的话，就会导致我们的单个打包文件过大，反而影响效率。所以模块化颗粒度太小不行，太大也不行，而 Code Splitting 就是为了解决我们模块化颗粒度太大的问题。

- 多入口打包

多入口打包就是将一个页面作为一个打包入口，而对于不同页面中公共的部分再去提取到公共的文件中去，而多入口打包的配置也很容易
```
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: { // 多入口打包，多个入口文件
    index: './src/index.js',
    album: './src/album.js'
  },
  output: {
    filename: '[name].bundle.js' // 由于多入口打包，采用占位符
  },
  optimization: {
    splitChunks: {
      // 自动提取所有公共模块到单独 bundle
      chunks: 'all'
    }
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Multi Entry',
      template: './src/index.html',
      filename: 'index.html',
      chunks: ['index'] // 配置chunk，防止同时载入
    }),
    new HtmlWebpackPlugin({
      title: 'Multi Entry',
      template: './src/album.html',
      filename: 'album.html',
      chunks: ['album']
    })
  ]
}
```

- Webpack 按需加载

按需加载是我们在开发中常见的需求，我们在处理打包的时候，我们可以需要哪个模块是，再加载哪个模块。Webpack 中支持动态导入的方式去支持按需加载我们的模块，所有动态加载的模块都会被自动分包，相比于分包加载的方式，动态加载的方式更加灵活。
例如我们有两个模块 album 和 posts，我们就可以使用 import 去实现动态导入，import 返回一个 promise 对象，
```
// ./posts/posts.js
export default () => {
    const posts = document.createElement('div')
    posts.className = 'posts'
    ...
    return posts
}
```
```
// ./album/album.js
export default () => {
    const album = document.createElement('div')
    album.className = 'album'
    ...
    return album
}
```
```
// import posts from './posts/posts'
// import album from './album/album'

const render = () => {
  const hash = window.location.hash || '#posts'

  const mainElement = document.querySelector('.main')

  mainElement.innerHTML = ''

  if (hash === '#posts') {
    // mainElement.appendChild(posts())\
    // 魔法注释：给模块重命名
    import(/* webpackChunkName: 'components' */'./posts/posts').then(({ default: posts }) => {
      mainElement.appendChild(posts())
    })
  } else if (hash === '#album') {
    // mainElement.appendChild(album())
    import(/* webpackChunkName: 'components' */'./album/album').then(({ default: album }) => {
      mainElement.appendChild(album())
    })
  }
}

render()

window.addEventListener('hashchange', render)
```

## css 的模块化打包

- MiniCssExtractPlugin 是一个能够将 css 文件从打包文件中单独提取出来的插件，通过这个插件我们就可以实现 css 模块的按需加载。
- optimize-css-assets-webpack-plugin 是一个能够压缩 css 文件的插件，因为使用了 MiniCssExtractPlugin 之后，就不需要使用 style 标签的形式去加载 css 了，所以我们就不需要 style-loader 了
- terser-webpack-plugin 因为 optimize-css-assets-webpack-plugin 是需要使用在 optimization 的 minimizer 中的，而开启了 optimization，Webpack 就会认为我们的压缩代码需要自己配置，所以 js 文件就不会压缩了，所以我们需要安装 terser-webpack-plugin 再去压缩 js 代码
```
// 安装 mini-css-extract-plugin
yarn add mini-css-extract-plugin --dev
// 安装 optimize-css-assets-webpack-plugin
yarn add optimize-css-assets-webpack-plugin --dev
// 安装 terser-webpack-plugin
yarn add terser-webpack-plugin --dev
```
接下来我们就可以配置它们了
```
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin')
const TerserWebpackPlugin = require('terser-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: {
    main: './src/index.js'
  },
  output: {
    filename: '[name].bundle.js'
  },
  optimization: {
    minimizer: [
      new TerserWebpackPlugin(), // 压缩 js 代码
      new OptimizeCssAssetsWebpackPlugin() // 压缩模块化的 css 代码
    ]
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 'style-loader', // 将样式通过 style 标签注入
          MiniCssExtractPlugin.loader, // 使用 MiniCssExtractPlugin 的 loader 就不需要 style-loader 了
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Dynamic import',
      template: './src/index.html',
      filename: 'index.html'
    }),
    new MiniCssExtractPlugin()
  ]
}
```

## 输出 hash 文件名

一般我们去部署前端的资源文件的时候，我们都会启用服务器的静态资源缓存，这样对于用户的浏览器而言就可以缓存住我们的静态资源，后续就不再需要请求服务器去请求这些静态资源了，这样我们的应用的相应速度就会有一个大幅度的提升。不过开启客户端的静态资源缓存也会有问题，如果我们在设置缓存时间过短的话，那么缓存就没什么意义了，而设置过长的话，一旦应用发生了更新，就没有办法即时更新到客户端。为了解决这个问题，我们就需要在生产模式下，为文件名使用 hash，这样一旦我们的文件资源发生改变，我们的文件名称也会随之发生改变，而对于客户端而言，全新的文件名也就意味着全新的请求，这样我们就可以将缓存时间设置的非常长，也不用去担心文件不更新的问题。

- hash      
项目级别的 hash，一旦任何文件发生修改，都会生成新的 hash

- chunkhash
只要是同一路的打包，hash 都是相同的，例如一个模块内的 js 和 css 的 hash 前缀都是相同的

- contenthash
文件级别的 hash，根据文件内容输出的 hash 值，只要是不同的文件就有不同的 hash 值，这也是最推荐的 hash 方式

```
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin')
const TerserWebpackPlugin = require('terser-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: {
    main: './src/index.js'
  },
  output: {
    filename: '[name]-[contenthash:8].bundle.js'
  },
  optimization: {
      ...
  },
  module: {
      ...
  },
  plugins: [
    ...
    new MiniCssExtractPlugin({
      filename: '[name]-[contenthash:8].bundle.css'
    })
  ]
}

```


