# \$attrs 和 $listeners

在研究 Vue 的组件库，之前也用过 $attrs 和 $listeners，官方文档描述的不太详细，也没有太好的例子，就没有深入的研究过这两个属性。最近生病在家，正好有时间好好研究一下 Vue 的高阶用法，写了几个 demo，下面我们来看看这两个属性到底有什么奥秘。

## \$attrs 

我们先来看看官方文档的 api 描述是怎样描述 $attrs 的

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210418160929983-1644196118.png)

乍一看可能有点懵，下面我们结合下面的例子来看看 $attrs 的作用

### 普通的 props 传值

先来看看下面的例子，我自己写了一个双向绑定的 input 的组件，来验证一下。

```html
<!-- 父组件 -->
<mxx-input v-model="user.username" type="text" foo="foo"></mxx-input>
```

```html
<!-- 子组件 -->
<div>
  <input v-bind="$attrs" :type="type" :value="value">
</div>
```

```js
// 子组件
export default {
  name: 'MxxInput',
  props: {
    type: {
      type: String,
      default: 'text'
    },
    value: {
      type: String,
    }
  },
}
```

我们来看看子组件渲染的 DOM 结构

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210418161150653-1283990622.png)

我们会发现 foo 这个属性加在了 MxxInput 组件的最外层。
如果你仔细看过 Vue 的官方文档的话，你会发现我们在使用 props 的方式向子组件传值的时候，子组件没有使用 props 作为接受的话，那么这个属性会自动设置在<mark>子组件的最外层的 HTML 标签</mark>上。如果是 class 和 style 的话，会合并最外层标签的 class 和 style。

## \$attrs 的作用

如果子组件中不想继承父组件传入的非 prop 属性，可以使用 inheritAttrs 禁用继承，然后通过 v-bind="$attrs" 把外部传入的非 prop 属性设置给希望的标签上。
也就是你不想将没有设置 props 的属性自动继承到组件最外层的标签上那么你就需要将 inheritAttrs 这个属性设置为 false，但是这不会改变 class 和 style。


```js
// 子组件
export default {
  name: 'MxxInput',
  inheritAttrs: false,
  props: {
    type: {
      type: String,
      default: 'text'
    },
    value: {
      type: String,
    }
  },
}
```

我们再来看看新的 DOM 结构

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210418164018582-256372363.png)

我们可以看到最外层的 DOM 结构上没有 foo 这个属性了！
而 $attrs 的值就是 { "foo": "foo" }，这样我们就可以利用这个特性实现组件的属性透传，更多有趣的用法，你可以自己去摸索一下。



## $listeners

![](https://gitee.com/maoxiaoxing/mxx-blog/raw/master/Img/1575596-20210418164945891-1935931411.png)

### 普通 $emit 传值

我们先来看看如果我们想从一个组件向组件外传值，一种方法就是使用 $emit 向组件外暴露一个事件，然后通过事件方法的参数传值。

```html
<input
  type="text"
  v-bind="$attrs"
  @focus="$emit('focus', $event)"
  @input="$emit('input', $event)"
>
```

### $listeners 的作用

其实 $attrs 的一个作用就是可以批量向组件内传值，但是如果我们想批量向组件外传值怎么办呢，这个时候我们就可以使用 $listeners

```html
<input
  type="text"
  v-bind="$attrs"
  v-on="$listeners"
>
```

$listeners 实际就相当于上面的多个 $emit 了
