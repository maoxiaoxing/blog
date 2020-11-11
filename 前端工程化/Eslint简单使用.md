# Eslint 简单使用
-----------------------------------

## 前端规范化

随着前端开发需求的日益膨胀，前端的规范化在一个开发团队中扮演着越来越重要的角色。那么我们为什么要有前端规范化呢？在我们日常实际的开发过程中，很多中型或者大型的软件开发都需要多人去协同开发，即使是在一个前端团队中，不同的开发人员也有不同的编码习惯和喜好。而这些不同的开发习惯必然会对项目进度造成一定的阻碍，会提高我们的项目维护成本，甚至在开发过程中就会对项目进度造成阻塞性的障碍，所以每个项目或者团队都需要有统一的标准。
那么我们在开发的过程中，哪些内容需要进行规范化呢？个人认为只要是开发过程中开发人员编写的产物（代码、文档、甚至是提交日志）都需要进行标准化，其中代码的规范最为重要，例如代码的缩进、是否以分号结尾、变量的命名规范等等。

### 前端规范化的方法

- 在开发之前人为的进行约定      
在开发之前，开发团队可以就这个问题开会，讨论约定一个统一的开发规范，并落实到文档上，来约束每个人的开发习惯。其实这个工作在每个团队的日常建设中，前端 leader 就应该去撰写一个文档来规范每个成员的开发习惯。但是这种方式始终是人为约定，落实到每个人的时候难免会出现一些问题。
- 使用 lint 工具        
前端 leader 应该去维护一个属于自己团队的脚手架，然后在脚手架中按照前端比较流行的规范配置这样的 lint 工具，要照顾到大多数人的开发习惯。这样整个团队就有了工具化的编码规范，这样即使在项目与项目之间进行协调合作的时候，在复用某个项目功能的时候，也不会因为编码风格不一样而导致难以协同开发的问题。

## Eslint 基本介绍

Eslint 目前是市面上最为主流的 JavaScript Lint 工具，它用来监测我们的 JS 代码，以此来保证我们的代码质量。Eslint 有自己的报错系统，所以很容易统一开发者的编码风格，它可以约束我们在开发过程中一些不合理的编码，例如我们定义一个从未使用的变量，或者我们在比较的时候喜欢用"=="符号。通过这些约束，不仅能够让整个项目有更高的代码质量，而且在一定程度上能够提升开发者的编码能力，因为你在开发过程中重复遇到一个问题，下次你就会记住这个问题，久而久之，你的编码能力就得到了一个隐性的提升。

## Eslint 快速上手

### 安装 Eslint

首先我们需要初始化一个 package.json，然后使用 npm 将 eslint 模块作为开发依赖去安装
```
npm init -y

npm i eslint --save-dev
```
之后我们可以使用 yarn 这样的工具去执行 node_modules 中的 bin 目录中的 eslint 文件，这个文件为我们提供了 eslint 的脚手架工具，我们可以使用 yarn 去查看 eslint 的版本，用来确保我们是否安装成功
```
yarn eslint --version
```
如果你的命令行输出了一个版本号，那么你就成功安装了 eslint

### Eslint 配置文件
当我们安装好 eslint 这个插件，我们就可以使用 node_modules 的 bin 目录下 eslint 为我们提供的脚手架去检查我们的代码了。假如我们想要检查根目录下的 test.js 文件，我们就可以执行下面的命令
```
yarn eslint ./test.js
```
然后上面的直接结果会报出一个错误

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201103215315331-1321328158.png)

上面提示我们没有 ESLint 没有发现一个配置文件，去为这个项目做一些配置项，请我们执行下面的命令
```
yarn eslint --init
```
接下来命令行会提出一些问题

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201103215729806-116789769.png)

上面的意思就是，你想怎样去使用 ESLint？
- 只检查语法错误
- 检查语法错误并发现问题代码
- 检查语法错误，发现问题代码，并校验代码的风格      

我选择的的是第三个，在开发的时候，我们对编写代码越严格越能提高整个项目的质量，并且能够提高我们的个人能力。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201103220144047-370507130.png)

上面就是第二个问题，意思就是你的项目中采用什么模块化的规范
- JavaScript modules 也就是 ESM 规范，
- commonjs规范
- 没有用到任何模块化

这个问题可以根据自己的需求进行选择

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201103220504121-298361065.png)

上面是第三个问题，意思就是你用的是哪一款矿建
- React 框架
- Vue.js 框架
- 没有使用任何框架

这个问题也可以根据实际情况去选择

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/1575596-20201103220714630-1077155849.png)

你是否使用 ts？
如果你用到了，你就可以输入y

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E8%BF%90%E8%A1%8C%E5%9C%A8%E4%BB%80%E4%B9%88%E7%8E%AF%E5%A2%83.png)

你的代码是在什么环境中运行的？
一般我们前端的代码都是在浏览器环境中运行的，因此我们就选择 Browser 就可以了。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E5%AE%9A%E4%B9%89%E4%BB%80%E4%B9%88%E9%A3%8E%E6%A0%BC%E7%9A%84%E4%BB%A3%E7%A0%81.png)

你想怎样去定义你的项目的代码风格？
- 使用市面上主流的风格
- 通过询问你一些问题去形成一个风格
- 根据你的 js 文件去推断你的风格

我们就选择市面上主流的风格就好了，如果我们的项目有新的成员去加入，也可以让他更快更好的去加入。

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E6%83%B3%E8%A6%81%E4%BB%80%E4%B9%88%E6%A0%87%E5%87%86.png)

选完风格之后，它又会让你进一步选择你想要的风格？

我选择的是 Standard，这个风格的最大特点就是不用再末尾添加分号，至于你想要在开发中使用什么样的风格，可以具体去研究一下

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/eslint%E8%A6%81%E4%BB%80%E4%B9%88%E6%A0%BC%E5%BC%8F%E7%9A%84%E9%85%8D%E7%BD%AE%E5%8C%96%E6%96%87%E4%BB%B6.png)

这个问题就是，你想要以何种类型的文件去作为你的配置文件
一般我们都会去选择 js 文件作为我们的配置文件

![](https://gitee.com/maoxiaoxing/lagou/raw/master/picture/%E9%9C%80%E8%A6%81%E4%B8%8B%E8%BD%BD%E5%AE%83%E4%BB%AC%E5%90%97.png)

之后还有一个问题就是提示你去下载一些依赖，我们选择是，去下载我们的插件就好了，最后我们就会生成一个 .eslintrc.js 的文件

```
module.exports = {
  env: {
    browser: true,
    es2021: true
  },
  extends: [
    'standard'
  ],
  parserOptions: {
    ecmaVersion: 12
  },
  rules: {
  }
}
```
下面我们来解读一下这个文件
- env
表示我们的项目运行的环境，一般会有如下这些成员
```
browser - 浏览器环境中的全局变量
node - Node.js 全局变量和 Node.js 作用域
commonjs - CommonJS 全局变量和 CommonJS 作用域（用于 Browserify/Webpack 打包的只在浏览器中运行的代码）
shared-node-browser - Node.js 和 Browser 通用全局变量
es6 - 启用除了modules 以外的所有 ECMAScript 6 特性（该选项会自动设置 ecmaVersion 解析器选项为 6）
worker - Web Workers 全局变量
amd - 将 require() 和 define() 定义为像 amd 一样的全局变量
mocha - 添加所有的 Mocha 测试全局变量
jasmine - 添加所有的 Jasmine 版本 1.3 和 2.0 的测试全局变量
jest - Jest 全局变量
phantomjs - PhantomJS 全局变量
protractor - Protractor 全局变量
qunit - QUnit 全局变量
jquery - JQuery 全局变量
prototypejs - Prototype.js 全局变量
shelljs - ShellJS 全局变量
meteor - Meteor 全局变量
mongo - MongoDB 全局变量
applescript - AppleScript 全局变量
nashorn - Java 8 Nashorn 全局变量
serviceworker - Service Worker 全局变量
atomtest - Atom 测试全局变量
embertest - Ember 测试全局变量
webextensions - WebExtensions 全局变量
greasemonkey - GreaseMonkey 全局变量
```
而且这些环境都不是互斥的，我们在同一个项目共同使用这些成员。

- extends
这个是扩展我们的标准的，我们可以看到它是一个数组，它也同时能够配置多个

- parserOptions
这是一个语法解析器，这个解析器的作用就是控制我们是否可以使用某一个语法

- rules
这个选项就是配置我们写代码的一些规则
- "off"或者0，不启用这个规则
- "warn"或者1，出现问题会有警告
- "error"或者2，出现问题会报错

```
"no-alert": 0,//禁止使用alert confirm prompt
"no-array-constructor": 2,//禁止使用数组构造器
"no-bitwise": 0,//禁止使用按位运算符
"no-caller": 1,//禁止使用arguments.caller或arguments.callee
"no-catch-shadow": 2,//禁止catch子句参数与外部作用域变量同名
"no-class-assign": 2,//禁止给类赋值
"no-cond-assign": 2,//禁止在条件表达式中使用赋值语句
"no-console": 2,//禁止使用console
"no-const-assign": 2,//禁止修改const声明的变量
"no-constant-condition": 2,//禁止在条件中使用常量表达式 if(true) if(1)
"no-continue": 0,//禁止使用continue
"no-control-regex": 2,//禁止在正则表达式中使用控制字符
"no-debugger": 2,//禁止使用debugger
"no-delete-var": 2,//不能对var声明的变量使用delete操作符
"no-div-regex": 1,//不能使用看起来像除法的正则表达式/=foo/
"no-dupe-keys": 2,//在创建对象字面量时不允许键重复 {a:1,a:1}
"no-dupe-args": 2,//函数参数不能重复
"no-duplicate-case": 2,//switch中的case标签不能重复
"no-else-return": 2,//如果if语句里面有return,后面不能跟else语句
"no-empty": 2,//块语句中的内容不能为空
"no-empty-character-class": 2,//正则表达式中的[]内容不能为空
"no-empty-label": 2,//禁止使用空label
"no-eq-null": 2,//禁止对null使用==或!=运算符
"no-eval": 1,//禁止使用eval
"no-ex-assign": 2,//禁止给catch语句中的异常参数赋值
"no-extend-native": 2,//禁止扩展native对象
"no-extra-bind": 2,//禁止不必要的函数绑定
"no-extra-boolean-cast": 2,//禁止不必要的bool转换
"no-extra-parens": 2,//禁止非必要的括号
"no-extra-semi": 2,//禁止多余的冒号
"no-fallthrough": 1,//禁止switch穿透
"no-floating-decimal": 2,//禁止省略浮点数中的0 .5 3.
"no-func-assign": 2,//禁止重复的函数声明
"no-implicit-coercion": 1,//禁止隐式转换
"no-implied-eval": 2,//禁止使用隐式eval
"no-inline-comments": 0,//禁止行内备注
"no-inner-declarations": [2, "functions"],//禁止在块语句中使用声明（变量或函数）
"no-invalid-regexp": 2,//禁止无效的正则表达式
"no-invalid-this": 2,//禁止无效的this，只能用在构造器，类，对象字面量
"no-irregular-whitespace": 2,//不能有不规则的空格
"no-iterator": 2,//禁止使用__iterator__ 属性
"no-label-var": 2,//label名不能与var声明的变量名相同
"no-labels": 2,//禁止标签声明
"no-lone-blocks": 2,//禁止不必要的嵌套块
"no-lonely-if": 2,//禁止else语句内只有if语句
"no-loop-func": 1,//禁止在循环中使用函数（如果没有引用外部变量不形成闭包就可以）
"no-mixed-requires": [0, false],//声明时不能混用声明类型
"no-mixed-spaces-and-tabs": [2, false],//禁止混用tab和空格
"linebreak-style": [0, "windows"],//换行风格
"no-multi-spaces": 1,//不能用多余的空格
"no-multi-str": 2,//字符串不能用\换行
"no-multiple-empty-lines": [1, {"max": 2}],//空行最多不能超过2行
"no-native-reassign": 2,//不能重写native对象
"no-negated-in-lhs": 2,//in 操作符的左边不能有!
"no-nested-ternary": 0,//禁止使用嵌套的三目运算
"no-new": 1,//禁止在使用new构造一个实例后不赋值
"no-new-func": 1,//禁止使用new Function
"no-new-object": 2,//禁止使用new Object()
"no-new-require": 2,//禁止使用new require
"no-new-wrappers": 2,//禁止使用new创建包装实例，new String new Boolean new Number
"no-obj-calls": 2,//不能调用内置的全局对象，比如Math() JSON()
"no-octal": 2,//禁止使用八进制数字
"no-octal-escape": 2,//禁止使用八进制转义序列
"no-param-reassign": 2,//禁止给参数重新赋值
"no-path-concat": 0,//node中不能使用__dirname或__filename做路径拼接
"no-plusplus": 0,//禁止使用++，--
"no-process-env": 0,//禁止使用process.env
"no-process-exit": 0,//禁止使用process.exit()
"no-proto": 2,//禁止使用__proto__属性
"no-redeclare": 2,//禁止重复声明变量
"no-regex-spaces": 2,//禁止在正则表达式字面量中使用多个空格 /foo bar/
"no-restricted-modules": 0,//如果禁用了指定模块，使用就会报错
"no-return-assign": 1,//return 语句中不能有赋值表达式
"no-script-url": 0,//禁止使用javascript:void(0)
"no-self-compare": 2,//不能比较自身
"no-sequences": 0,//禁止使用逗号运算符
"no-shadow": 2,//外部作用域中的变量不能与它所包含的作用域中的变量或参数同名
"no-shadow-restricted-names": 2,//严格模式中规定的限制标识符不能作为声明时的变量名使用
"no-spaced-func": 2,//函数调用时 函数名与()之间不能有空格
"no-sparse-arrays": 2,//禁止稀疏数组， [1,,2]
"no-sync": 0,//nodejs 禁止同步方法
"no-ternary": 0,//禁止使用三目运算符
"no-trailing-spaces": 1,//一行结束后面不要有空格
"no-this-before-super": 0,//在调用super()之前不能使用this或super
"no-throw-literal": 2,//禁止抛出字面量错误 throw "error";
"no-undef": 1,//不能有未定义的变量
"no-undef-init": 2,//变量初始化时不能直接给它赋值为undefined
"no-undefined": 2,//不能使用undefined
"no-unexpected-multiline": 2,//避免多行表达式
"no-underscore-dangle": 1,//标识符不能以_开头或结尾
"no-unneeded-ternary": 2,//禁止不必要的嵌套 var isYes = answer === 1 ? true : false;
"no-unreachable": 2,//不能有无法执行的代码
"no-unused-expressions": 2,//禁止无用的表达式
"no-unused-vars": [2, {"vars": "all", "args": "after-used"}],//不能有声明后未被使用的变量或参数
"no-use-before-define": 2,//未定义前不能使用
"no-useless-call": 2,//禁止不必要的call和apply
"no-void": 2,//禁用void操作符
"no-var": 0,//禁用var，用let和const代替
"no-warning-comments": [1, { "terms": ["todo", "fixme", "xxx"], "location": "start" }],//不能有警告备注
"no-with": 2,//禁用with
"array-bracket-spacing": [2, "never"],//是否允许非空数组里面有多余的空格
"arrow-parens": 0,//箭头函数用小括号括起来
"arrow-spacing": 0,//=>的前/后括号
"accessor-pairs": 0,//在对象中使用getter/setter
"block-scoped-var": 0,//块语句中使用var
"brace-style": [1, "1tbs"],//大括号风格
"callback-return": 1,//避免多次调用回调什么的
"camelcase": 2,//强制驼峰法命名
"comma-dangle": [2, "never"],//对象字面量项尾不能有逗号
"comma-spacing": 0,//逗号前后的空格
"comma-style": [2, "last"],//逗号风格，换行时在行首还是行尾
"complexity": [0, 11],//循环复杂度
"computed-property-spacing": [0, "never"],//是否允许计算后的键名什么的
"consistent-return": 0,//return 后面是否允许省略
"consistent-this": [2, "that"],//this别名
"constructor-super": 0,//非派生类不能调用super，派生类必须调用super
"curly": [2, "all"],//必须使用 if(){} 中的{}
"default-case": 2,//switch语句最后必须有default
"dot-location": 0,//对象访问符的位置，换行的时候在行首还是行尾
"dot-notation": [0, { "allowKeywords": true }],//避免不必要的方括号
"eol-last": 0,//文件以单一的换行符结束
"eqeqeq": 2,//必须使用全等
"func-names": 0,//函数表达式必须有名字
"func-style": [0, "declaration"],//函数风格，规定只能使用函数声明/函数表达式
"generator-star-spacing": 0,//生成器函数*的前后空格
"guard-for-in": 0,//for in循环要用if语句过滤
"handle-callback-err": 0,//nodejs 处理错误
"id-length": 0,//变量名长度
"indent": [2, 4],//缩进风格
"init-declarations": 0,//声明时必须赋初值
"key-spacing": [0, { "beforeColon": false, "afterColon": true }],//对象字面量中冒号的前后空格
"lines-around-comment": 0,//行前/行后备注
"max-depth": [0, 4],//嵌套块深度
"max-len": [0, 80, 4],//字符串最大长度
"max-nested-callbacks": [0, 2],//回调嵌套深度
"max-params": [0, 3],//函数最多只能有3个参数
"max-statements": [0, 10],//函数内最多有几个声明
"new-cap": 2,//函数名首行大写必须使用new方式调用，首行小写必须用不带new方式调用
"new-parens": 2,//new时必须加小括号
"newline-after-var": 2,//变量声明后是否需要空一行
"object-curly-spacing": [0, "never"],//大括号内是否允许不必要的空格
"object-shorthand": 0,//强制对象字面量缩写语法
"one-var": 1,//连续声明
"operator-assignment": [0, "always"],//赋值运算符 += -=什么的
"operator-linebreak": [2, "after"],//换行时运算符在行尾还是行首
"padded-blocks": 0,//块语句内行首行尾是否要空行
"prefer-const": 0,//首选const
"prefer-spread": 0,//首选展开运算
"prefer-reflect": 0,//首选Reflect的方法
"quotes": [1, "single"],//引号类型 `` "" ''
"quote-props":[2, "always"],//对象字面量中的属性名是否强制双引号
"radix": 2,//parseInt必须指定第二个参数
"id-match": 0,//命名检测
"require-yield": 0,//生成器函数必须有yield
"semi": [2, "always"],//语句强制分号结尾
"semi-spacing": [0, {"before": false, "after": true}],//分号前后空格
"sort-vars": 0,//变量声明时排序
"space-after-keywords": [0, "always"],//关键字后面是否要空一格
"space-before-blocks": [0, "always"],//不以新行开始的块{前面要不要有空格
"space-before-function-paren": [0, "always"],//函数定义时括号前面要不要有空格
"space-in-parens": [0, "never"],//小括号里面要不要有空格
"space-infix-ops": 0,//中缀操作符周围要不要有空格
"space-return-throw-case": 2,//return throw case后面要不要加空格
"space-unary-ops": [0, { "words": true, "nonwords": false }],//一元运算符的前/后要不要加空格
"spaced-comment": 0,//注释风格要不要有空格什么的
"strict": 2,//使用严格模式
"use-isnan": 2,//禁止比较时使用NaN，只能用isNaN()
"valid-jsdoc": 0,//jsdoc规则
"valid-typeof": 2,//必须使用合法的typeof的值
"vars-on-top": 2,//var必须放在作用域顶部
"wrap-iife": [2, "inside"],//立即执行函数表达式的小括号风格
"wrap-regex": 0,//正则表达式字面量用小括号包起来
"yoda": [2, "never"]//禁止尤达条件
```



