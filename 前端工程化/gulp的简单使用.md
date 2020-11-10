# Gulp的简单使用
----------------------------------------

## 1. Gulp 的简单介绍
![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201024191705609-567659218.png)

Gulp 是一款比较主流的前端构建工具，它相比于 Grunt 的构建速度会更快一些，因为 Gulp 是基于内存去实现的，相对于磁盘读写，它的速度肯定会更快。而且它默认支持同时执行多个任务，所以效率会大大提高，而且它的使用方式相对于 Grunt 会更加简单一些，配置文件的结构也比较简单易懂，所以现在也是比较受欢迎的前端构建工具，很多大型项目会将 Gulp 和 Webpack 混合使用，去处理一些比较复杂的情况。

## 2. Gulp 的基本使用
我们创建一个文件，然后初始化 package.json，将 gulp 作为一个开发依赖添加
```
yarn init -y

yarn add gulp -D
```
然后在根目录创建一个 gulpfile.js 的文件, gulpfile.js 就是 gulp 的入口文件

- （1）我们可以在 gulpfile.js 做出一些配置，然后用 gulp 来帮助我们构建项目
```
exports.foo = (done) => {
    console.log('foo task working')

    done() // 任务完成
}
```
然后我们执行下面的命令
```
yarn gulp foo // 输出 foo task working
```

- （2）默认任务
我们导出一个 default 的成员
```
exports.default = (done) => {
    console.log('default task working')
    
    done() // 任务完成
}
```
然后我们执行下面的命令
```
yarn gulp // 输出 default task working
```

- （3）gulp4.0之前的任务执行
在 gulp4.0 之前，我们执行任务，是需要引入 gulp 模块的
```
const gulp = require('gulp')

gulp.task('bar', done => {
    console.log('bar task working')
    done()
})
```
然后我们执行下面的命令
```
yarn gulp bar // bar task working
```
可以看到，我们还是能正常执行这个任务，只是因为 gulp4.0 之后保留了这个 api，但是现在已经不被推荐使用了，更推荐我们使用导出成员的方式去配置项目，所以我们只需要了解就好了。

## 3. Gulp 的组合任务
gulp 为我们提供了 series（串行） 和 parallel（并行） 这两个 api 来处理串行任务和并行任务
```
const { series, parallel } = require('gulp')

const task1 = done => {
    setTimeout(() => {
        console.log('task1 working')
        done()
    }, 1000)
}

const task2 = done => {
    setTimeout(() => {
        console.log('task2 working')
        done()
    }, 1000)
}

const task3 = done => {
    setTimeout(() => {
        console.log('task3 working')
        done()
    }, 1000)
}

exports.foo = series(task1, task2, task3)

exports.bar = parallel(task1, task2, task3)
```
我们先来执行 foo 任务
```
// 分别输出 task1 working  task2 working  task3 working，每次输出间隔一秒
yarn gulp foo 
```
我们再来执行 bar 任务
```
// 一秒后，同时输出 task1 working  task2 working  task3 working
yarn gulp bar
```
这两个任务在实际开发中非常有用，例如我们在编译 css、js、html 的时候，它们之间是没有依赖的，因此我们就可以使用 parallel 同时编译；而像我们构建时，我们需要先编译，然后再构建，所以就可以使用 series 来串行执行这两个任务

## 4. Gulp 中的异步任务
Gulp 中的任务都是异步任务，其实就是通过异步函数去执行。那么 Gulp 是如何判断异步任务是否完成了呢，主要有下面三种方案
- （1）通过回调的方式
```
exports.callback = done => {
    console.log('callback task')
    done()
}
```
gulp 中的回调和 node 中的回调都是采用错误优先的方式，也就是说我们可以在函数中传入一个错误，就可以阻止接下来的任务的执行
```
exports.callback_error = done => {
    console.log('callback task')
    done(new Error('task failed'))
}
```

- （2）Promise
```
exports.promise = () => {
    console.log('promise task')
    return Promise.resolve()
}
```
```
exports.promise_error = () => {
    console.log('promise task')
    return Promise.reject(new Error('task failed'))
}
```
只要 resolve 或者 reject 后，就相当于停止了这个任务
既然我们能使用 promise，那么我们同样也能使用 async 和 await 这个语法糖
```
const timeout = time => {
    return new Promise(resolve => {
        setTimeout(resolve, time)
    })
}

exports.async = async () => {
    await timeout(1000)
    console.log('async task')
}
```

- （3）stream 方式
```
const fs = require('fs')
exports.stream = () => {
    const readStream = fs.createReadStream('package.json')
    const writeStream = fs.createWriteStream('temp.txt')
    readStream.pipe(writeStream)
    return readStream
}
```
我们也可以使用 readStream 的 end 事件模拟上面的过程
```
exports.stream = (done) => {
    const readStream = fs.createReadStream('package.json')
    const writeStream = fs.createWriteStream('temp.txt')
    readStream.pipe(writeStream)
    readStream.on('end', () => {
        done()
    })
}
```

## 5. Gulp 构建过程核心工作原理
Gulp 的构建过程一般都是将文件读出来，然后通过插件进行一些转换，然后写入到另一个位置。假如我们没有这样的构建工具，我们一般就是手动将一些代码通过某个压缩工具压缩过后，然后粘贴到一个新的文件中。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201024204433013-267321586.png)

下面我们通过底层 node 的 api 去显示这样一个过程
我们先再根目录创建这样的 css 文件 /css/normialize.css
我们在 gulpfile.js 中写入下面的代码
```
const fs = require('fs')
const { Transform } = require('stream')

exports.default = () =>{
    // 文件读取流
    const read = fs.createReadStream('./css/normialize.css')
    // 文件写入流
    const write = fs.createWriteStream('./css/normalize.min.css')

    // 文件转换流
    const transform = new Transform({
        transform: (chunk, encoding, callback) => {
            // 核心转换过程
            // chunk => 读取流中读取到的内容（Buffer）
            const input = chunk.toString()
            const output = input.replace(/\s+/g, '').replace(/\/\*.+?\*\//g, '')
            callback(null, output)
        }
    })

    // 把读出来的文件流导入写入文件流
    read
    .pipe(transform)
    .pipe(write)

    return read
}
```
之后我们执行下面的命令
```
yarn gulp
```
这样我们就能得到一个 normalize.min.css 的文件，里面就是被压缩之后的代码，上面的代码可以简化为下面的这张图

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201024205022038-1354434865.png)

Gulp 就是基于流的构建系统，Gulp 希望实现一个构建管道的概念，这样后续再做扩展插件的时候，就会有一个很统一的方式

## 6. Gulp 文件操作 API
在 Gulp 中也提供了很多强大的插件
例如我们想要将 .css 文件都压缩为 .min.css
我们可以先安装两个插件 
```
yarn add gulp-clean-css -D

yarn add gulp-rename -D
```
然后我们在 gulpfile.js 文件中做出一些配置
```
const { src, dest } = require('gulp')
const cleanCss = require('gulp-clean-css')
const rename = require('gulp-rename')

exports.default = () => {
    return src('src/*.css') // 创建读取流
        .pipe(cleanCss()) // 压缩文件
        .pipe(rename({extname: '.min.css'})) // 重命名文件
        .pipe(dest('dist')) // 创建写入流
}
```

