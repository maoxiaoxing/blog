# Grunt 的简单使用
-----------------------------

## 1. Grunt 的简单介绍
![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201024170823333-189958915.png)

grunt 可以说是前端构建工具的鼻祖了，它的插件生态非常完善，用官方的话来说：grunt 的插件几乎可以自动化帮助你做任何你想做的事情。虽然 grunt 虽然强大，但是现在 grunt 基本已经退出历史舞台了。因为 grunt 的工作过程是基于临时文件实现的，所以它的构建速度相对较慢，例如你去使用 grunt 对 sass 文件的构建，我们一般先会对 sass 文件进行编译操作，再去添加一些私有属性的前缀，最后再去压缩代码。在这样的过程当中，grunt 的每一步都会有磁盘读写操作。比如在 sass 文件编译过后，它会将 sass 文件结果写入一个临时文件，然后下一个插件再去读取这个文件，如果你有一个超大型的项目，文件特别多的话，每处理一个文件都会进行磁盘读写操作，就会导致构建速度特别慢，因为你要频繁地进行 I/O 操作。
下面我们就来简单的学习一下 grunt

## 2. Grunt 的基本使用
我们创建一个文件，然后初始化 package.json，添加 grunt 模块
```
yarn init -y

yarn add grunt -D
```
然后在根目录创建一个 gruntfile.js 的文件, gruntfile.js 就是 grunt 的入口文件
- （1）我们就可以在 gruntfile.js 中使用 grunt 去处理一些任务
```
module.exports = grunt => {
    grunt.registerTask('foo', () => {
        console.log('hello grunt')
    })
}
```
然后我们在控制台执行以下命令
```
yarn grunt foo // 输出 hello grunt
```
这样 grunt 就会帮助我们执行 foo 任务

- （2）默认任务
如果我们将 registerTask 的第一个参数设置为 default 那么它就是一个默认任务，我们就不用指定任务
```
grunt.registerTask('default', '任务描述', () => {
    console.log('default task')
})
```
然后我们在控制台执行以下命令
```
yarn grunt // 输出 default task
```
如果我们将 registerTask 第二个参数设置为数组，它就会执行多个任务
```
grunt.registerTask('default', ['foo', 'bar'])
```

- (3) 异步任务
我们并不能直接在 registerTask 中执行异步任务，而是需要借助它内置的 async 函数
```
grunt.registerTask('async-task', function() {
    const done = this.async()

    setTimeout(() => {
        console.log('async task')
    }, 1000)
})
```

## 3. Grunt 标记任务失败
我们在实际开发中，如果有一个文件找不到了，我们就需要 grunt 将这个任务标记为失败任务
```
grunt.registerTask('bad', () => {
    console.log('bad work')
    return false
})
```
这时控制台就会报错
如果我们有多个任务，串行执行，那么失败的任务之后的任务都不会被执行
```
grunt.registerTask('bad', () => {
    console.log('bad work')
    return false
})

grunt.registerTask('foo', () => {
    console.log('foo task')
})

grunt.registerTask('bar', () => {
    console.log('bar task')
})

grunt.registerTask('default', ['foo', 'bad', 'bar'])
```
这时控制台就会报错, bad 任务之后的任务都不会被执行

## 3. Grunt 的配置方法
除了 registerTask 处理任务的方法，grunt 还有一个 initConfig 初始化配置选项的 api
```
grunt.initConfig({
    // foo: 'bar'
    foo: {
        bar: 123
    }
})

grunt.registerTask('foo', () => {
    console.log(grunt.config('foo.bar'))
})
```
在控制台执行下面的命令
```
yarn grunt foo // 输出 123
```

## 4. Grunt 多目标任务
多目标模式，可以让我们根据配置形成多个子任务, 我们可以借助 initConfig 去初始化任务
```
grunt.initConfig({
    build: {
        options: {
            foo: 'bar'
        },
        css: {
            options: {
                foo: 'baz'
            }
        },
        js: '2'
    }
})

// 多目标模式，可以让我们根据配置形成多个子任务
grunt.registerMultiTask('build', function() {
    // console.log('build task')
    console.log(this.options())
    console.log(`target: ${this.target}, data: ${this.data}`)
})
```
值得注意的是，当我们在子任务中配置 option 会覆盖掉父任务中的 option

5. Grunt 的常见的插件使用
Grunt 提供的强大的插件，这些插件内部都封装了一些通用的任务，下面我们来看看怎么使用它们

- （1）grunt-contrib-clean
grunt-contrib-clean 用来帮助我们来清除项目开发中的一些临时文件
首先我们来安装一下这个插件
```
yarn add grunt-contrib-clean
```
然我我们在 gruntfile.js 中配置一下
```
grunt.initConfig({
    clean: {
        temp: 'temp/*.js'
    }
})

grunt.loadNpmTasks('grunt-contrib-clean')
```
然我们执行下面的命令
```
yarn grunt clean
```
这样 temp 文件夹下面的 app.js 就被清除了

- （2）grunt-sass   
处理 sass 文件
首先我们先安装 grunt-sass 模块
```
yarn add grunt-sass sass -D
```
然后我们在 gruntfile.js 文件中做一些配置
```
const sass = require('sass')

module.exports = grunt => {
    grunt.initConfig({
        sass: {
            options: {
                implementation: sass,
                sourceMap: true,
            },
            main: {
                files: {
                    'dist/css/main.css': 'src/scss/main.scss'
                }
            }
        },
    })

    grunt.loadNpmTasks('grunt-sass')
}
```
然后执行下面的命令
```
yarn grunt sass
```

- （3）grunt-babel
grunt-babel 用来编译 es6 的代码
首先我们先安装它
```
yarn grunt grunt-babel @babel/core @babel/preset-env -D
```
随着我们使用更多的插件，我们会不断地使用 loadNpmTasks，这显然会比较麻烦，所以我们需要使用一个 load-grunt-tasks的插件，它会帮助我们自动加载任务
```
yarn add load-grunt-tasks -D
```
我们再来整理一下我们的 gruntfile.js 中的配置
```
const sass = require('sass')
const loadGruntTasks = require('load-grunt-tasks')

module.exports = grunt => {
    grunt.initConfig({
        sass: {
            options: {
                implementation: sass,
                sourceMap: true,
            },
            main: {
                files: {
                    'dist/css/main.css': 'src/scss/main.scss'
                }
            }
        },
        babel: {
            options: {
                sourceMap: true,
                presets: ['@babel/preset-env']
            },
            main: {
                files: {
                    'dist/js/app.js': 'src/js/app.js'
                }
            }
        },
    })

    // grunt.loadNpmTasks('grunt-sass')

    loadGruntTasks(grunt) // 自动加载所有的 grunt 插件的中的任务
}
```
然我们再来执行一下
```
yarn grunt babel
```
这时 dist 目录就会多出一个 app.js 文件，就是 es6 转换成 es5 的代码

- （4）grunt-contrib-watch
我们在实际开发的时候肯定能遇到这样的需求，在我们写完一段代码之后，需要实时更新，我们就需要 grunt-contrib-watch 帮助我们去监听文件，我们来安装一下这个插件
```
yarn add grunt-contrib-watch -D
```
然我们再来对 gruntfile.js 做出一些配置
```

const sass = require('sass')
const loadGruntTasks = require('load-grunt-tasks')

module.exports = grunt => {
    grunt.initConfig({
        sass: {
            options: {
                implementation: sass,
                sourceMap: true,
            },
            main: {
                files: {
                    'dist/css/main.css': 'src/scss/main.scss'
                }
            }
        },
        babel: {
            options: {
                sourceMap: true,
                presets: ['@babel/preset-env']
            },
            main: {
                files: {
                    'dist/js/app.js': 'src/js/app.js'
                }
            }
        },
        watch: {
            js: {
                files: ['src/js/*.js'],
                tasks: ['babel']
            },
            css: {
                files: ['src/scss/*.scss'],
                tasks: ['sass']
            }
        }
    })

    // grunt.loadNpmTasks('grunt-sass')

    loadGruntTasks(grunt) // 自动加载所有的 grunt 插件的中的任务

    grunt.registerTask('default', ['sass', 'babel', 'watch'])
}
```
然后我们执行下面的命令
```
yarn grunt
```
然后我们每次再修改文件后，就是会实时更新了

## 5. 总结
上面就是 grunt 的基本使用，其实我们对于 grunt 只需要了解就可以了，不需要在这上面消耗太多的精力，因为现在基本没有使用 grunt 的项目了，除非一些"史前"的项目。当然我们如果你了解 grunt 的话，对于你学习 gulp 是有一定帮助的，因为它们之间的语法会有一些相似的地方。
