# Webpack 不同环境下的配置
--------------------------------------

我们在使用 Webpack 去构建我们的项目的时候，我们肯定要区分本地开发环境和线上环境的一些配置，因为它们两者有很多不同的地方，例如线上环境我们就不需要开启 source map 这样的选项来防止别人暴露我们的源代码。所以我们可以通过使用不同配置文件去在不同环境下打包我们的代码。
我们至少需要三个文件去配置我们的项目

- webpack.common.js 公共配置
- webpack.dev.js 开发环境配置
- webpack.prod.js 生产环境配置

这样我们就能使用不同的配置文件去配置不同的环境，这时我们还需要一个插件 webpack-merge 去在开发配置文件和生产配置文件中合并我们的公共配置文件
```
// 安装 webpack-merge
yarn add webpack-merge --dev
```

## webpack.common.js

这个文件我们可以用来配置一些公共的 webpack 配置
```
const webpack = require('webpack')
const VueLoaderPlugin = require('vue-loader/lib/plugin') // 配合 vue-loader 使用，用于编译转换 .vue 文件
const HtmlWebpackPlugin = require('html-webpack-plugin') // 用于生成 index.html 文件

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'js/bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
      // 它会应用到普通的 `.js` 文件
      // 以及 `.vue` 文件中的 `<script>` 块
      {
        test: /.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },
      // 配置 eslint-loader 检查代码规范，应用到 .js 和 .vue 文件
      {
        test: /\.(js|vue)$/,
        use: {
          loader: 'eslint-loader',
          options: {
            formatter: require('eslint-friendly-formatter') // 默认的错误提示方式
          }
        },
        enforce: 'pre', // 编译前检查
        exclude: /node_modules/, // 不检查的文件
        include: [__dirname + '/src'], // 要检查的目录
      },
      // 它会应用到普通的 `.css` 文件
      // 以及 `.vue` 文件中的 `<style>` 块
      {
        test: /\.css$/,
        use: ['vue-style-loader', 'css-loader']
      },
      // 配置 less-loader ，应用到 .less 文件，转换成 css 代码
      {
        test: /\.less$/,
        // loader: ['style-loader', 'css-loader', 'less-loader'],
        use: [{
          loader: "style-loader" // creates style nodes from JS strings
        }, {
          loader: "css-loader" // translates CSS into CommonJS
        }, {
          loader: "less-loader" // compiles Less to CSS
        }]
      },
      {
        test: /\.(png|jpe?g|gif)$/,
        use: {
          loader: 'file-loader',
          options: {
            name: 'img/[name].[ext]'
          }
        }
      },
    ]
  },
  plugins: [
    new webpack.DefinePlugin({
      // 值要求的是一个代码片段
      BASE_URL: JSON.stringify('/')
    }),
    new VueLoaderPlugin(),
    new HtmlWebpackPlugin({
      title: 'lxcan vue project',
      template: './public/index.html'
    }),
  ]
}
```

## webpack.dev.js

这个文件我们可以配置一些开发环境的配置，例如我们可以开启 source map 和 dev-server 等功能来帮助我们有更好的开发体验，在进行开发环境的配置之前，我们需要使用 webpack-merge 去合并我们的公共配置文件
```
const  commonConfig =  require('./webpack.common')
const merge = require('webpack-merge')
const path = require('path')

module.exports = merge(commonConfig,{
    mode:'development',
    devtool:'cheap-module-eval-source-map',
    module:{
        rules:[
            {
                test: /\.js$/, 
                exclude: /node_modules/, 
                use: 'eslint-loader',
                enforce:'pre'
            },
        ]
    },
    devServer: {
        contentBase: path.join(__dirname, 'dist'),
        port: 9000,
        open: true,
        hotOnly: true
    }
})
```

## webpack.prod.js

这个文件我们就可以配置生产环境的配置，我们就可以不用使用 source map 和 dev-server 等这些开发环境的工具。在这个文件中，我们可以使用 CleanWebpackPlugin 去清除之前打包的代码，并且使用一些 hash 文件去能够让浏览器知道我们的代码更新
```
const common = require('./webpack.common')
const merge = require('webpack-merge')
const { CleanWebpackPlugin } = require('clean-webpack-plugin') // 打包之前清除 dist 目录
const CopyWebpackPlugin = require('copy-webpack-plugin') // 拷贝静态文件至输出目录

module.exports = merge(common, {
  mode: 'production',
  output: {
    filename: 'js/bundle-[contenthash:8].js'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new CopyWebpackPlugin(['public'])
  ]
})
```
这样不同环境的配置文件就完成了，当然我的很多配置都比较简陋，这里我只是提供一个不同环境配置文件的思想，其中不同配置文件的具体配置，你可以根据你的项目去进行配置。
