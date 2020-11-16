# webpack 的简单使用
----------------------------------------

## 1. Webpack 的简单介绍
![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack.png)

[Webpack](https://www.webpackjs.com/) 是一个模块化的打包工具，它可以将我们的模块化代码都打包到一个 js 文件中。而对于我们在不同环境中有兼容性问题的代码，我们就可以通过模块加载器（loader）去解决。而且 Webpack 具备代码拆分的能力（Code Splitting），它可以将应用中的代码根据我们的要求去打包，我们可以将应用初次加载的代码打包到一起，其他模块的代码打包到一起，实现渐进式加载。这样我们就可以避免将所有代码都打包到一起，而引起打包文件过大，导致页面加载速度过慢的问题。Webpack 还支持将其他类型的文件打包到 js 文件中。Webpack 是目前最主流的打包工具，它的功能非常强大，覆盖了绝大部分的现代 web 应用开发过程，下面我们来看看如何使用它来构建我们自己的项目。

## 2. Webpack 的基本使用
### 安装

我们想在项目使用 Webpack，首先我们需要在项目中安装它，我们要初始化一个 package.json，然后将 webpack 和 webpack-cli 作为我们的开发依赖引入
```
yarn init -y // 或者 npm init -y

yarn add webpack webpack-cli --dev
```
你可以使用 npm 去下载依赖，下面我都使用 yarn 去安装依赖，如果你想了解 yarn，可以去[yarn 官网](https://yarn.bootcss.com/)看一下。
安装之后，我们可以使用下面的命令查看 Webpack 的版本
```
yarn webpack --version
```
如果能够正常输出版本，那就证明我们安装成功了。
接下来我们就可以使用 webpack 去打包我们的文件。webpack 默认会从 src 下面的 index.js 开始打包，将所有的 js 文件导打包，之后我们所有的 js 文件就会被打包到 dist 目录下的 main.js 文件中。

### Webpack 基本配置文件

Webpack4 之后的版本支持零配置的方式打包，Webpack 默认约定将 "src/index.js" 作为默认入口文件打包到 "dist/mian.js" 中，这种零配置的方式虽然好，但是有时候我们想自定义这个入口文件和输出文件。Webpack 中提供了一种方式，我们可以在项目的根目录下创建一个 webpack.config.js 的文件，这个文件是一个运行在 node.js 中的文件，我们需要按照 commonjs 的规范去编写我们的代码。例如我们想将 src 目录下的 main.js 作为入口文件，将 dist 目录作为输出目录，将 bundle.js 作为输出文件，我们可以做如下配置
```
const path = require('path')
module.exports = {
    entry: './src/main.js', // 入口文件
    output: {
        filename: 'bundle.js', // 输出文件
        path: path.join(__dirname, 'dist') // 输出目录
    }
}
```
接下来我们运行打包命令，就可以将我们的项目打包到 dist 目录下的 bundle.js 文件中去
```
yarn webpack
```

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack%E5%8E%8B%E7%BC%A9.png)

webpack 的打包会默认开启生产模式，它会自动帮助我们加载很多插件，去压缩我们的代码，但是这样肯定是不太利于我们去开发的
我们可以通过命令行去指定以什么模式去打包
```
yarn webpack --mode development
```
这样我们就以开发模式去打包我们的代码

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E6%89%93%E5%8C%85.png)

可以看到开发模式的代码多了很多注释，还会添加一些调试过程中的辅助，当然我们也可以在配置文件中去配置这个模式
```
const path = require('path')

module.exports = {
  // 这个属性有三种取值，分别是 production、development 和 none。
  // 1. 生产模式下，Webpack 会自动优化打包结果；
  // 2. 开发模式下，Webpack 会自动优化打包速度，添加一些调试过程中的辅助；
  // 3. None 模式下，Webpack 就是运行最原始的打包，不做任何额外处理；
  mode: 'development',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist')
  }
}
```

### Webpack Loader

loader 在 Webpack 中扮演着至关重要的角色，下面我们来看看 Webpack 官方文档对 loader 解释
```
loader 用于对模块的源代码进行转换。
loader 可以使你在 import 或"加载"模块时预处理文件。因此，loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的强大方法。
loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。
loader 甚至允许你直接在 JavaScript 模块中 import CSS文件！
```
Webpack 的 loader 非常强大，你可以通过 loader 在你的项目中去加载任何资源！

#### css-loader 和 style-loader

其实 Webpack 内部默认是有 js loader 的，但是它只能处理 js 文件，所以当我们去在模块中引入 css 文件的时候，Webpack 就会报出错误，这个时候 css-loader 就会体现它的作用，它会帮助我们去处理 css 文件，并将这些 css 文件生成一个 js 模块打包到输出的 js 文件中，然后 style-loader 会将这个模块通过 style 标签的形式引入到页面中去
```
// 安装 css-loader
yarn add css-loader style-loader --dev

```
安装好 css-loader 和 style-loader 之后我们要在 Webpack 的配置文件中做出如下配置
```
const path = require('path')
module.exports = {
    entry: './src/main.js', // 入口文件
    output: {
        filename: 'bundle.js', // 输出文件
        path: path.join(__dirname, 'dist') // 输出目录
    },
    module: {
      rules: [
        {
          test: /.css$/,
          use: [
            'style-loader',
            'css-loader',
          ]
        },
      ]
    }
}
```
需要注意的是，loader 的加载默认是从后向前加载，所以 css-loader 要放在后面，因为我们要先将 css 转换成 js 模块，才能正常打包

#### file-loader

我们在实际开发中，不光要写一些逻辑代码，往往还要引入一些文件去帮助我们去优化项目，例如我们经常会在项目中引入一些图片文件。Webpack 为我们提供了 file-loader 去帮助我们处理这些文件
```
// 安装 file-loader
yarn add file-loader --dev
```
安装好 file-loader 之后，我们就可以在配置文件中进行一些配置
```
module: {
    rules: [
        {
            test: /\.(png|jpe?g|gif)$/,
            use: "file-loader"
        },
    ]
}
```
配置好之后，我们再去打包，dist 目录下就会多出打包后的图片，但是如果你这时候打开项目地址的话，你会发现图片加载不进来，这是因为 Webpack 会默认加载项目的根目录，而你的图片是在 dist 目录下的，所以这时候需要你做出一些配置去让它默认加载 dist 目录下的图片，做法就是在输出文件的时候指定根目录
```
const path = require('path')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    publicPath: 'dist/' // 指定 dist 为根目录
  },
  module: {
    rules: [
      {
        test: /.png$/,
        use: 'file-loader'
      }
    ]
  }
}
```

#### url-loader

除了像 file-loader 这种通过拷贝物理文件方式去处理文件资源外，还有像通过 Data URLs 这种方式去表述文件，Data URLs 是一种特殊的 url 协议，它可以通过 url 去表述文件，正常我们都是通过一个 url 去服务器访问这个资源文件，而 Data URLs 本身就会包含这个文件的所有内容，从而不用去服务器访问这个资源，减少了 http 请求。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/base64.png)

例如下面就是一个 Data URLs 表示的文件类型
```
data:text/html;chartset=UTF-8,<h1>html content</h1>
```
上面这个 url 就表示这是一个 html 类型的文件，编码是 utf-8，文件内容是一个 h1 标签

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/base64%E6%A0%BC%E5%BC%8F%E8%BE%93%E5%87%BA%E5%86%85%E5%AE%B9.png)

当我们的资源是图片的时候，就会被解析成 base64 编码，浏览器依旧能根据这个编码解析出这个图片的内容

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/png%E8%BD%AC%E6%88%90base64.png)

这样我们就可以通过 url-loader 去加载任何我们想要加载的文件
```
// 安装 url-loader
yarn add url-loader --dev
```
然后我们就可以不用使用 file-loader 去处理我们的文件了，而去使用 url-loader 去加载我们的文件

```
module: {
  rules: [
    {
      test: /.png$/,
      use: {
        loader: 'url-loader',
        options: {
          limit: 10 * 1024 // 超出 10 KB 使用 file-loader
        }
      }
    }
  ]
}
```
使用 url-loader 的方式去处理我们的文件当然会显得很方便，但是只是适合比较小的文件，如果文件过大的话，会影响我们的打包速度，所以我们可以为文件设置一个最大限度，如果文件超过这个临界值的话，还是使用传统的 file-loader 去通过物理拷贝的方式求处理文件。值的注意的是，如果我们通过限制文件大小的方式去处理文件的话，即使你的配置中没有 file-loader，你还是需要下载 file-loader，这样 Webpack 才会正常工作。 

#### babel-loader

由于 Webpack 默认就能处理我们代码中的 import 和 export，所以很多人可能会认为 Webpack 会自动编译 es6 的代码。实则不然，因为模块打包需要，所以 Webpack 才会处理 import 和 export，而其他的 es6 代码，Webpack并不能去处理它们，babel-loader 能够帮助我们去处理这些 es6 的代码。
```
// 安装 babel-loader
yarn add babel-loader @babel/core @babel-preset-env --dev
```
我们在安装 babel-loader 的时候还需要安装 @babel/core @babel-preset-env 这两个插件。

- @babel/core

@babel/core是babel的核心库，所有的核心Api都在这个库里，这些Api供babel-loader调用。

- @babel-preset-env

@babel-preset-env 是用来转换 es6 中的具体特性的一个插件集合

下面我们来看看怎么配置 babel-loader
```
module: {
  rules: [
    {
      test: /.js$/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    },
  ]
}
```

#### 开发一个 loader

下面我们来来自己开发一个 loader，然后通过这个过程我们再来深入了解 loader 的工作原理。下面我们来开发一个 markdown-loader，这样我们就能将 markdown 转换为一个字符串然后渲染到浏览器中去。
首先我们创建一个 markdown-loader.js 的文件，用来开发我们的 loader。每一个 Webpack loader 都需要导出一个函数，这个函数就是对整个资源的处理过程，这个函数中有一个参数，这个参数是一个入口，就是我们需要处理的资源，出口就是我们需要返回的内容，值得注意的是，这个返回内容必须是一段 javaScript 代码，否则 Webpack 就会报错。
```
module.exports = source => { // source 就是我们需要处理的资源
  return 'console.log("hello loader")'
}
```
接下来我们就可以在配置文件中引入这个 loader
```
module: {
  rules: [
    {
      test: /.md$/,
      use: [
        './markdown-loader'
      ]
    }
  ]
}
```
下面我们需要引入一个去解析 markdown 文件的插件 marked
```
yarn add marked --dev
```
下面我们在 loader 中去处理 markdown 资源
```
const marked = require('marked')

module.exports = source => {
  const html = marked(source)
  return `export default ${JSON.stringify(html)}`
}
```
这样我们就完成了一个处理 markdown 类型文件的 loader，当然除了上面这种处理方式，我们还有其他的处理方式。因为 loader 支持链式调用，所以我们可以直接返回一个 html 字符串，然后将这个 html 字符串交给 html-loader 去处理
```
// 下载 html-loader
yarn add html-loader --dev
```
我们修改一下 loader 中对 markdown 资源的处理
```
const marked = require('marked')

module.exports = source => {
  const html = marked(source)
  // 返回 html 字符串交给下一个 loader 处理
  return html
}
```
之后我们在配置文件中引入 html-loader 去处理 markdown-loader 返回的内容，值得注意的是 loader 的执行顺序，use 数组后面的先执行，所以我们要将 html-loader 放在数组的前面。
```
module: {
  rules: [
    {
      test: /.md$/,
      use: [
        'html-loader', // 处理 markdown-loader 返回的内容
        './markdown-loader'
      ]
    }
  ]
}
```

### Webpack 插件机制

插件机制是 Webpack 的另外一个核心机制，目的是增强 Webpack 的自动化能力。loader 专注于实现各种资源的加载，plugin 是为了解决其他的自动化工作，例如 plugin 能够帮助我们去清除上一个打包的打包文件或者能够帮助我们压缩我们的代码，plugin 几乎能够帮助我们去实现大多数的前端工程化工作，下面我们来了解一些 Webpack 中比较常用的插件。

#### clean-webpack-plugin

如果你自己创建一个新项目的话，你打包过一个文件目录，当你去修改了一些配置而再去打包的话，你的打包目录下就会有之前遗留的文件，这显然不是我们想要的结果，而 clean-webpack-plugin 就能够帮助在打包之前清除上一次的打包文件。
```
yarn add clean-webpack-plugin --dev
```
下面我们来配置一下这个插件
```
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    publicPath: 'dist/'
  },
  plugins: [
    new CleanWebpackPlugin()
  ]
}
```
上面的配置文件中，我们引入了 clean-webpack-plugin，然后结构出 CleanWebpackPlugin 这个插件，之后我们在 plugins 中创建一个数组，然后在数组成员中执行这个插件，这样下次我们打包代码之前就会自动清除 dist 目录下的文件。

#### html-webpack-plugin

我们在开发的时候，需要将打包生成的 js 文件引入到 src 目录下的 index.html，这种硬编码的方式显然是不好的，因为如果你使用的一些 hash-chunk 这样的插件的话，打包出来的文件名称每一次都不一样，你就需要每一次都手动去引入这个文件，这种做法显然会显得比较"笨"，html-webpack-plugin 就能帮助我们去解决这个问题。
```
yarn add html-webpack-plugin --dev
```
下面我们来看看它的基础配置
```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    // publicPath: 'dist/'
  },
  plugins: [
    new HtmlWebpackPlugin(),
  ]
}
```
这样 Webapck 在打包的时候，就会在打包目录下自动生成一个 index.html 文件，需要注意的是，如果做了这样的配置，我们就不需要将 publicPath 配置为 dist 了，因为它直接就引入了当前文件下的 bundle.js，当然你也可以将 publicPath 设置为 './'。
当然这样显然还是有许多问题的，比如我们想修改生成的这个 html 文件的标题和一些配置项。我们可以在 CleanWebpackPlugin 这个构造函数中传入一些参数去配置这些选项。
```
plugins: [
  // 用于生成 index.html
  new HtmlWebpackPlugin({
    title: 'Webpack Plugin Sample', // 配置标题
    meta: { // 配置 meta 选项
      viewport: 'width=device-width'
    },
    template: './src/index.html' // 模板文件
  }),
]
```
然后在模板文件中通过模板语法去使用这些写入的选项，我们可以使用 htmlWebpackPlugin 去获取到上面的这些配置
```
// ./src/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Webpack</title>
</head>
<body>
  <div class="container">
    <h1><%= htmlWebpackPlugin.options.title %></h1>
  </div>
</body>
</html>
```
当我们执行打包结果后，就会生成下面的 html 文件
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Webpack</title>
<meta name="viewport" content="width=device-width"></head>
<body>
  <div class="container">
    <h1>Webpack Plugin Sample</h1>
  </div>
<script type="text/javascript" src="bundle.js"></script></body>
</html>
```
在实际开发中，我们还有需要输出多个页面文件的需求，生成过个 html 文件也很简单，只需要我们在 plugins 中多创建几个 HtmlWebpackPlugin 实例即可。
```
plugins: [
  // 用于生成 index.html
  new HtmlWebpackPlugin({
    title: 'Webpack Plugin Sample',
    meta: {
      viewport: 'width=device-width'
    },
    template: './src/index.html'
  }),
  // 用于生成 about.html
  new HtmlWebpackPlugin({
    filename: 'about.html'
  })
]
```
这样在 dist 目录下就会生成 index.html 和 about.html 两个 html 文件了。

#### copy-webpack-plugin

在我们去打包文件的时候，往往还需要将一些静态的文件复制输出到目录下，例如 public 目录下面的文件

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack%E8%B5%8B%E5%80%BC%E9%9D%99%E6%80%81%E8%B5%84%E6%BA%90.png)

这时候，我们就可以借助 copy-webpack-plugin 这个插件去帮助我们去完成这个需求
```
// 安装 copy-webpack-plugin
yarn add copy-webpack-plugin --dev
```
然后我们在配置文件中做出下面的配置
```
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CopyWebpackPlugin = require('copy-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    // publicPath: 'dist/'
  },
  plugins: [
    // 用于生成 index.html
    new HtmlWebpackPlugin({
      title: 'Webpack Plugin Sample',
      meta: {
        viewport: 'width=device-width'
      },
      template: './src/index.html'
    }),
    // 用于生成 about.html
    new HtmlWebpackPlugin({
      filename: 'about.html'
    }),
    new CopyWebpackPlugin([
      // 'public/**'
      'public'
    ])
  ]
}
```
#### 开发一个 Webpack 插件

相比于 loader，plugin 拥有更加宽泛的能力，plugin 几乎可以触及到 Webpack 编译的每一个环节，那么它是如何实现的呢？
其实 Webpack 插件实现也特别简单，plugin 通过钩子机制实现，我们可以在这些"钩子"上面挂载一些任务，从而去扩展 Webpack 的能力。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack%E6%8F%92%E4%BB%B6%E9%92%A9%E5%AD%90%E5%9B%BE%E8%A7%A3.png)

Webpack 要求我们的插件必须是一个函数或者是一个包含 apply 方法的对象
```
class MyPlugin {
  apply (compiler) {
    console.log('MyPlugin 启动')
  }
}
```
下面我们开发一个删除 bundle.js 文件中的注释的插件

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack%E6%89%93%E5%8C%85%E5%B8%A6%E6%B3%A8%E9%87%8A%E7%9A%84%E6%96%87%E4%BB%B6.png)

下面我们到 [Webpack 官方文档](https://www.webpackjs.com/api/compiler-hooks/#emit) 去找一个为 emit 的钩子，它的特性符合我们的需求

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack%20emit%E9%92%A9%E5%AD%90.png)

我们可以通过 compiler.hooks.emit.tap 就能访问到这个钩子 tap 函数接收两个参数：第一个参数是插件名称；第二个参数是我们需要挂载到这个钩子上的函数或者任务，这个函数有一个参数，可以理解为此次打包的上下文

```
class MyPlugin {
  apply (compiler) {
    compiler.hooks.emit.tap('MyPlugin', compilation => {
      // compilation => 可以理解为此次打包的上下文
      for (const name in compilation.assets) {
        // compilation.assets 为资源文件信息，它是一个对象
        // name 就是每个资源的名字，例如 bundle.js
        if (name.endsWith('.js')) {
          const contents = compilation.assets[name].source() // 文件资源
          const withoutComments = contents.replace(/\/\*\*+\*\//g, '') // 正则替换注释
          compilation.assets[name] = { // 覆盖文件
            source: () => withoutComments,
            size: () => withoutComments.length
          }
        }
      }
    })
  }
}
```
这样一个插件就写好了，我们在配置文件中去使用这个插件
```
plugins: [
  new MyPlugin()
]
```
这样以 js 为结尾的文件的注释就被去掉了

## Webpack 开发体验

上面的章节我们已经讲了 loader 和 plugin 的基本使用，但是在实际开发过程中，只完成这些肯定是不够的，我们在开发过程中肯定要满足以下几点需求

- 项目以 server 的形式去运行
- 修改代码之后，自动编译、自动更新
- 提供 source map 文件，帮助定位错误代码

实现这个需求，就能够帮助我们在实际开发中提升我们的开发效率，这几个需求在 Webpack 中都有相应的功能去实现了，下面我们来了解一下这些功能。

### Webpack 自动编译

每次再我们修改完代码之后，我们都需要手动去打包才能更新我们的代码到浏览器，这样就很麻烦，Webpack 中有一种 watch 工作模式能够帮助我们监听文件变化，从而帮助我们自动打包。
使用这种模式也很简单，只需要执行下面的命令即可
```
yarn webpack --watch
```
这样在每次打包完之后，就不会自动退出命令行，webpack 会自动监视我们打包后的文件，从而实时更新

### Webpack Dev Server

Webpack Dev Server 会帮助我们自动开启一个 HTTP Server，它会集成自动编译和自动刷新浏览器等功能。
```
// 安装 webpack-dev-server
yarn add webpack-dev-server --dev
```
webpack-dev-server 为我们提供了一个 cli 程序，你可以直接运行它，也可以在 package.json 中的 script 中去配置命令，这里我们去配置一个命令，从而方便去使用它

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack-dev-server.png)

这时我们就可以使用下面的命令去运行 webpack-dev-server
```
yarn dev
```
这时我们就可以实现自动编译和自动刷新浏览器的功能了，为了提升构建效率， webpack-dev-server 并没有将打包文件写入磁盘当中，而是将这些 http 文件存放在内存中，从内存中去读取这些文件，这样就会减少很多不必要的磁盘读写操作，从而大大提高我们的构建效率。
接下来我们看看 webpack-dev-server 的一些常见配置
```
devServer: {
  contentBase: './public', // 指定静态文件输出目录
  open: true, // 启动 server 之后自动打开浏览器
  hotOnly: true, 热更新
  proxy: { // 代理 api
    '/api': {
      // http://localhost:8080/api/users -> https://api.github.com/api/users
      target: 'https://api.github.com',
      // http://localhost:8080/api/users -> https://api.github.com/users
      pathRewrite: { // 代理路径重写
        '^/api': ''
      },
      // 不能使用 localhost:8080 作为请求 GitHub 的主机名
      changeOrigin: true // 会以实际 api 地址请求
    }
  }
},
```

### Source Map

当我们打包之后，生成的打包文件中的代码，和们在开发过程中的代码完全不同，所以这就为我们在浏览器上调试带来了困难，我们就无法定位我们的错误代码。Source Map 就能为我们解决这个问题，顾名思义，Source Map 就是源代码地图的意思它会映射我们转换之后的代码，通过这个 Source Map 文件我们就可以逆向得到我们开发的代码。

![](https://img2020.cnblogs.com/bhttps://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack-souorce-map.png)

配置 Source Map 也很简单，我们只需要在配置文件中配置一个 devtool 字段，就可以开启 Source Map
```
module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist')
  },
  devtool: 'eval', // 开启 source map
}
```
Source Map 有很多模式，你可以根据自己的需要来开启对应的模式

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/webpack-devtool.png)

我个人在开发的时候会选择 cheap-module-source-map，有一下几个原因

- 我的代码每行不会超出 120 个字符，能定位到行就够了
- 使用框架的情况比较多，loader 转换之后差异过大
- 虽然首次打包速度慢，但是开发中更多使用 server 去运行，重写打包速度相对较快

而在生产模式中，我往往会将 devtool 设置为 none，因为这样别人就不会看到你的源代码。当然这些模式的选择也没有绝对，你可以根据自己的需求去选择这些模式。




