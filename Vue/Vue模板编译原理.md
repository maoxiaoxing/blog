# Vue模板编译原理

## 模板使用

我们在使用 Vue 的时候，如果你不是使用 JSX 去写的话，在写 DOM 的时候我们一般都会使用 template 去编写我们的代码，又或者我们会使用 render 函数去处理一些比较复杂的业务。Vue 的官方文档提到 render 函数会比 template 模板更加底层一些，render 函数饿编译效率会更高一些，但是大量的写 render 函数肯定会增加我们开发的难度并且会降低效率，所以我们需要在实际开发中去恰当选择书写模式，快速解决问题才是我们最终的目的。
模板编译分为三个阶段：生成 ast、优化静态内容、生成 render
下面我们通过一段代码来回顾一下 template 和 render 写 DOM 的方式

- template 模板

```html
<div> 
<h1 @click="handler">title</h1> 
<p>some content</p> 
</div>
```

- render 函数

```javaScript
render (h) { 
  return h('div', [ 
    h('h1', { on: { click: this.handler} }, 'title'), 
    h('p', 'some content') 
  ]) 
}
```

我们能看到对于同一段 DOM ，render 函数会显得更加复杂，如果说你的 DOM 结构嵌套层级比较深的话，render 函数可能会让代码的维护变得困难一些。所以 template 模板为我们提供了更加便捷的书写 DOM 的方式，这也是 Vue 用的比较爽的原因之一。

## 模板编译

### 编译的入口

我们在使用 vue-cli 去创建 Vue 项目的时候，在 main.js 中都会看到下面的代码
```javaScript
new Vue({
  router,
  store,
  render: (h) => h(App)
}).$mount('#app')
```
#### $mount

$mount一般就是编译的入口，所以我们先来看看模板编译的入口文件
位置在 src\platforms\web\entry-runtime-with-compiler.js
```javaScript
Vue.prototype.$mount = function (
  el?: string | Element,
  // 非ssr情况下为 false，ssr 时候为true
  hydrating?: boolean
): Component {
  // 获取 el 对象
  el = el && query(el)

  /* istanbul ignore if */
  // el 不能是 body 或者 html
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  // 把 template/el 转换成 render 函数
  if (!options.render) {
    let template = options.template
    // 如果模板存在
    if (template) {
      if (typeof template === 'string') {
        // 如果模板是 id 选择器
        if (template.charAt(0) === '#') {
          // 获取对应的 DOM 对象的 innerHTML
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        // 如果模板是元素，返回元素的 innerHTML
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        // 否则返回当前实例
        return this
      }
    } else if (el) {
      // 如果没有 template，获取el的 outerHTML 作为模板
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }
      // 把 template 转换成 render 函数
      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  // 调用 mount 方法，渲染 DOM
  return mount.call(this, el, hydrating)
}
```
在模板编译的时候，render 的优先级是要优于 template 的，$mount 这个函数内部会判断是否有 render 函数，如果没有的话，就会通过 compileToFunctions 函数将 template 转换成 render 函数。

### 编译过程

#### createCompiler

![](https://img2020.cnblogs.com/blog/1575596/202101/1575596-20210130220448015-1903920736.png)

下面我们再来看看 compileToFunctions 到底做了什么
compileToFunctions 在 src/platforms/web/complier/index.js 文件
```javaScript
import { baseOptions } from './options'
import { createCompiler } from 'compiler/index'

const { compile, compileToFunctions } = createCompiler(baseOptions)

export { compile, compileToFunctions }
```
```javaScript
// src\platforms\web\compiler\options.js
export const baseOptions: CompilerOptions = {
  expectHTML: true,
  modules,
  directives,
  isPreTag,
  isUnaryTag,
  mustUseProp,
  canBeLeftOpenTag,
  isReservedTag,
  getTagNamespace,
  staticKeys: genStaticKeys(modules)
}
```
我们能看到 compileToFunctions 这个函数是由 createCompiler 去创建的，参数是一个 baseOptions，baseOptions 是一个对象，它会有很多属性，会有一些 pre 标签、自闭和标签等属性，比较重要的就是 directives 指令和 modules。
directives 会用插值表达式去处理像 v-html、v-text、v-model等指令，而 modules 则会处理一些 :bind、v-if、v-for、行内 style、class等属性。
下面我们再来看看 createCompiler 函数做了什么
```javaScript
// src\compiler\index.js

// `createCompilerCreator` allows creating compilers that use alternative
// parser/optimizer/codegen, e.g the SSR optimizing compiler.
// Here we just export a default compiler using the default parts.
export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  // 把模板转换成 ast 抽象语法树
  // 抽象语法树，用来以树形的方式描述代码结构
  const ast = parse(template.trim(), options)
  if (options.optimize !== false) {
    // 优化抽象语法树
    optimize(ast, options)
  }
  // 把抽象语法树生成字符串形式的 js 代码
  const code = generate(ast, options)
  return {
    ast,
    // 渲染函数
    render: code.render,
    // 静态渲染函数，生成静态 VNode 树
    staticRenderFns: code.staticRenderFns
  }
})
```
createCompiler 做了三件事
1. 将模板转换成 AST 语法树
2. 优化抽象语法树
3. 把抽象语法树生成字符串形式的 js 代码

#### 抽象语法树

上面我们说到了抽象语法树，那么什么是抽象语法树？

- 抽象语法树简称 AST（Abstract Syntax Tree）
- 使用对象的形式描述树形的代码结构
- 此处的抽象语法树是用来描述树形结构的 HTML 字符串

为什么要使用抽象语法树？

- 模板字符串转换成 AST 后，可以通过 AST 对模板做优化处理
- 标记模板中的静态内容，在 patch 的时候直接跳过静态内容
- 在 patch 的过程中静态内容不需要对比和重新渲染

上面都是一些对 AST 语法树的抽象描述，那么我们看看到底什么是抽象语法树
我们通过 [https://astexplorer.net/](https://astexplorer.net/) 这个工具看一下什么是抽象语法树，astexplorer 是一个能够将我们写的代码转换成抽象语法树的软件，有兴趣的可以去玩一玩

```vue
<template>
  <div><span>模板解析</span></div>
  <div>123</div>
</template>
```
上面的这段模板代码最终将会被转换成下面这样

![](https://img2020.cnblogs.com/blog/1575596/202101/1575596-20210131225423935-401678950.png)

我们能看到 html 模板被转换成了对象的形式，这样方便对 html 进行处理，比如对 DOM 做 diff 操作等。

#### parse

```javaScript
/**
 * Convert HTML string to AST.
 */
export function parse (
  template: string,
  options: CompilerOptions
): ASTElement | void {
  // 1. 解析 options
  warn = options.warn || baseWarn

  platformIsPreTag = options.isPreTag || no
  platformMustUseProp = options.mustUseProp || no
  platformGetTagNamespace = options.getTagNamespace || no
  const isReservedTag = options.isReservedTag || no
  maybeComponent = (el: ASTElement) => !!el.component || !isReservedTag(el.tag)

  transforms = pluckModuleFunction(options.modules, 'transformNode')
  preTransforms = pluckModuleFunction(options.modules, 'preTransformNode')
  postTransforms = pluckModuleFunction(options.modules, 'postTransformNode')

  delimiters = options.delimiters

  const stack = []
  const preserveWhitespace = options.preserveWhitespace !== false
  const whitespaceOption = options.whitespace
  let root
  let currentParent
  let inVPre = false
  let inPre = false
  let warned = false

  // 2. 对模板解析
  parseHTML(template, {
    warn,
    expectHTML: options.expectHTML,
    isUnaryTag: options.isUnaryTag,
    canBeLeftOpenTag: options.canBeLeftOpenTag,
    shouldDecodeNewlines: options.shouldDecodeNewlines,
    shouldDecodeNewlinesForHref: options.shouldDecodeNewlinesForHref,
    shouldKeepComment: options.comments,
    outputSourceRange: options.outputSourceRange,
    // 解析过程中的回调函数，生成 AST
    start (tag, attrs, unary, start, end) {
      // check namespace.
      // inherit parent ns if there is one
      const ns = (currentParent && currentParent.ns) || platformGetTagNamespace(tag)

      // handle IE svg bug
      /* istanbul ignore if */
      if (isIE && ns === 'svg') {
        attrs = guardIESVGBug(attrs)
      }

      let element: ASTElement = createASTElement(tag, attrs, currentParent)
      if (ns) {
        element.ns = ns
      }

      if (process.env.NODE_ENV !== 'production') {
        if (options.outputSourceRange) {
          element.start = start
          element.end = end
          element.rawAttrsMap = element.attrsList.reduce((cumulated, attr) => {
            cumulated[attr.name] = attr
            return cumulated
          }, {})
        }
        attrs.forEach(attr => {
          if (invalidAttributeRE.test(attr.name)) {
            warn(
              `Invalid dynamic argument expression: attribute names cannot contain ` +
              `spaces, quotes, <, >, / or =.`,
              {
                start: attr.start + attr.name.indexOf(`[`),
                end: attr.start + attr.name.length
              }
            )
          }
        })
      }

      if (isForbiddenTag(element) && !isServerRendering()) {
        element.forbidden = true
        process.env.NODE_ENV !== 'production' && warn(
          'Templates should only be responsible for mapping the state to the ' +
          'UI. Avoid placing tags with side-effects in your templates, such as ' +
          `<${tag}>` + ', as they will not be parsed.',
          { start: element.start }
        )
      }

      // apply pre-transforms
      for (let i = 0; i < preTransforms.length; i++) {
        element = preTransforms[i](element, options) || element
      }

      if (!inVPre) {
        processPre(element)
        if (element.pre) {
          inVPre = true
        }
      }
      if (platformIsPreTag(element.tag)) {
        inPre = true
      }
      if (inVPre) {
        processRawAttrs(element)
      } else if (!element.processed) {
        // structural directives
        // 结构化的指令
        // v-for
        processFor(element)
        processIf(element)
        processOnce(element)
      }

      if (!root) {
        root = element
        if (process.env.NODE_ENV !== 'production') {
          checkRootConstraints(root)
        }
      }

      if (!unary) {
        currentParent = element
        stack.push(element)
      } else {
        closeElement(element)
      }
    },

    end (tag, start, end) {
      ...
    },

    chars (text: string, start: number, end: number) {
      ...
    },
    comment (text: string, start, end) {
      // adding anyting as a sibling to the root node is forbidden
      // comments should still be allowed, but ignored
      ...
    }
  })
  return root
}
```
上面的 parse 函数就是将模板解析成 AST 语法树，parse 函数核心函数就是 parseHTML，parseHTML 有两个参数，第一个参数是模板，第二个参数是 options 选项，options有几个核心函数 start、end、chars、comment 分别为处理开始标签、结束标签、文本内容、注释节点。


#### createCompileToFunctionFn

```javaScript
// src\compiler\create-compiler.js

export function createCompilerCreator (baseCompile: Function): Function {
  // baseOptions 平台相关的options
  // src\platforms\web\compiler\options.js 中定义
  return function createCompiler (baseOptions: CompilerOptions) {
    function compile (
      template: string,
      options?: CompilerOptions
    ): CompiledResult {
      ...
      // 省略一些代码，这些代码主要是处理指令、modules和一些警告
    }

    return {
      compile,
      compileToFunctions: createCompileToFunctionFn(compile)
    }
  }
}
```

createCompileToFunctionFn 是由上面的 createCompilerCreator 返回的

```javaScript
// src\compiler\to-function.js

export function createCompileToFunctionFn (compile: Function): Function {
  const cache = Object.create(null)

  return function compileToFunctions (
    template: string,
    options?: CompilerOptions,
    vm?: Component
  ): CompiledFunctionResult {
    options = extend({}, options)
    const warn = options.warn || baseWarn
    delete options.warn

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      // detect possible CSP restriction
      try {
        new Function('return 1')
      } catch (e) {
        if (e.toString().match(/unsafe-eval|CSP/)) {
          warn(
            'It seems you are using the standalone build of Vue.js in an ' +
            'environment with Content Security Policy that prohibits unsafe-eval. ' +
            'The template compiler cannot work in this environment. Consider ' +
            'relaxing the policy to allow unsafe-eval or pre-compiling your ' +
            'templates into render functions.'
          )
        }
      }
    }

    // check cache
    // 1. 读取缓存中的 CompiledFunctionResult 对象，如果有直接返回
    // 可以自定义插值表达式，例如将默认的 {{}} 改成 任意标识符
    const key = options.delimiters
      ? String(options.delimiters) + template
      : template
    if (cache[key]) {
      return cache[key]
    }

    // compile
    // 2. 把模板编译为编译对象(render, staticRenderFns)，字符串形式的js代码
    const compiled = compile(template, options)

    // check compilation errors/tips
    if (process.env.NODE_ENV !== 'production') {
      if (compiled.errors && compiled.errors.length) {
        if (options.outputSourceRange) {
          compiled.errors.forEach(e => {
            warn(
              `Error compiling template:\n\n${e.msg}\n\n` +
              generateCodeFrame(template, e.start, e.end),
              vm
            )
          })
        } else {
          warn(
            `Error compiling template:\n\n${template}\n\n` +
            compiled.errors.map(e => `- ${e}`).join('\n') + '\n',
            vm
          )
        }
      }
      if (compiled.tips && compiled.tips.length) {
        if (options.outputSourceRange) {
          compiled.tips.forEach(e => tip(e.msg, vm))
        } else {
          compiled.tips.forEach(msg => tip(msg, vm))
        }
      }
    }

    // turn code into functions
    const res = {}
    const fnGenErrors = []

    // 3. 把字符串形式的js代码转换成js方法
    res.render = createFunction(compiled.render, fnGenErrors)
    res.staticRenderFns = compiled.staticRenderFns.map(code => {
      return createFunction(code, fnGenErrors)
    })

    // check function generation errors.
    // this should only happen if there is a bug in the compiler itself.
    // mostly for codegen development use
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      if ((!compiled.errors || !compiled.errors.length) && fnGenErrors.length) {
        warn(
          `Failed to generate render function:\n\n` +
          fnGenErrors.map(({ err, code }) => `${err.toString()} in\n\n${code}\n`).join('\n'),
          vm
        )
      }
    }
    // 4. 缓存并返回res对象(render, staticRenderFns方法)
    return (cache[key] = res)
  }
}
```

createCompileToFunctionFn 函数主要做了四件事：
1. 首先会创建一个空的 cache 缓存对象，然后判断是否有 [options.delimiters](https://cn.vuejs.org/v2/api/#delimiters) 这个属性(delimiters 是一个自定义插值表达式的属性，例如你可以将双大括号修改成你想定义的值)，如果没有那就使用默认的插值表达式当做 key ，如果有缓存结果，那就直接从缓存结果中取出来编译结果，不再需要重新编译。
2. 然后通过 compile 函数将模板编译成字符串形式的 js 代码
3. 之后通过 createFunction 通过 new Function 的形式将编译后的字符串的 js 代码转换成 render 函数
4. 最后缓存并返回编译结果

#### compile

下面我们再来看看 compile 函数中做了什么
```javaScript
function compile (
  template: string,
  options?: CompilerOptions
): CompiledResult {
  const finalOptions = Object.create(baseOptions)
  const errors = []
  const tips = []

  let warn = (msg, range, tip) => {
    (tip ? tips : errors).push(msg)
  }

  if (options) {
    if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
      // $flow-disable-line
      const leadingSpaceLength = template.match(/^\s*/)[0].length

      warn = (msg, range, tip) => {
        const data: WarningMessage = { msg }
        if (range) {
          if (range.start != null) {
            data.start = range.start + leadingSpaceLength
          }
          if (range.end != null) {
            data.end = range.end + leadingSpaceLength
          }
        }
        (tip ? tips : errors).push(data)
      }
    }
    // merge custom modules
    if (options.modules) {
      finalOptions.modules =
        (baseOptions.modules || []).concat(options.modules)
    }
    // merge custom directives
    if (options.directives) {
      finalOptions.directives = extend(
        Object.create(baseOptions.directives || null),
        options.directives
      )
    }
    // copy other options
    for (const key in options) {
      if (key !== 'modules' && key !== 'directives') {
        finalOptions[key] = options[key]
      }
    }
  }

  finalOptions.warn = warn

  const compiled = baseCompile(template.trim(), finalOptions)
  if (process.env.NODE_ENV !== 'production') {
    detectErrors(compiled.ast, warn)
  }
  compiled.errors = errors
  compiled.tips = tips
  return compiled
}
```
compile 函数的主要功能就是合并 baseOptions 和 compile 传进来的 options，然后使用 baseCompile 函数进行编译创建 compiled 对象，之后将一些编译后的错误和提示收集起来，再返回这个 compiled 对象


