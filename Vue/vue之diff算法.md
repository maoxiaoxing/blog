# vue之diff算法
----------------

## 前言

我们知道 Vue 使用的是虚拟 DOM 去减少对真实 DOM 的操作次数，来提升页面运行的效率。今天我们来看看当页面的数据改变的时候，Vue 是如何来更新 DOM 的。Vue和React在更新dom时，使用的算法基本相同，都是基于 [snabbdom](https://github.com/snabbdom/snabbdom)。
当页面上的数据发生变化时，Vue 不会立即渲染。而是经过 diff 算法，判断出哪些是不需要变化的，哪些是需要变化更新的，只需要更新那些需要更新的 DOM 就可以了，这样就减少了很多不需要的 DOM 操作，大大提升了性能。
Vue就使用了这样的抽象节点VNode，它是对真实DOM的一层抽象，而不依赖某个平台，它可以是浏览器平台，也可以是weex，甚至是node平台也可以对这样一棵抽象DOM树进行创建删除修改等操作，这也为前后端同构提供了可能。

## Vue 更新视图

我们知道在 Vue 1.x 中，每一个数据都对应一个 Watcher；而在 Vue 2.x 中，一个对应一个 Watcher，这样当我们的数据改变的时候，在 set 函数中会触发 Dep 的 notify 函数去通知 Watcher 去执行 vm._update(vm._render(), hydrating) 方法去更新视图，下面我们来看看 _update 方法
```javaScript
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
  const vm: Component = this
  const prevEl = vm.$el
  const prevVnode = vm._vnode
  const restoreActiveInstance = setActiveInstance(vm)
  vm._vnode = vnode
  // Vue.prototype.__patch__ is injected in entry points
  // based on the rendering backend used.
  /*基于后端渲染Vue.prototype.__patch__被用来作为一个入口*/
  if (!prevVnode) {
    // initial render
    vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
  } else {
    // updates
    vm.$el = vm.__patch__(prevVnode, vnode)
  }
  restoreActiveInstance()
  // update __vue__ reference
  /*更新新的实例对象的__vue__*/
  if (prevEl) {
    prevEl.__vue__ = null
  }
  if (vm.$el) {
    vm.$el.__vue__ = vm
  }
  // if parent is an HOC, update its $el as well
  if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
    vm.$parent.$el = vm.$el
  }
  // updated hook is called by the scheduler to ensure that children are
  // updated in a parent's updated hook.
}
```
很明显，我们能看到 _update 方法会将传入的 Vnode 将老的 Vnode 进行 patch 操作。
下面我们再来看看在 patch 函数中都发生了什么。

## patch

patch 函数将新老两个节点进行比较，然后判断出哪些是需要修改的节点，只需要修改这些节点即可，这样可以比较高效地更新 DOM，我们先来看一下代码
```javaScript
return function patch (oldVnode, vnode, hydrating, removeOnly) {
  /*vnode不存在调用销毁钩子删除节点*/
  if (isUndef(vnode)) {
    if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
    return
  }

  let isInitialPatch = false
  const insertedVnodeQueue = []

  /*oldVnode不存在，直接创建新节点*/
  if (isUndef(oldVnode)) {
    // empty mount (likely as component), create new root element
    isInitialPatch = true
    createElm(vnode, insertedVnodeQueue)
  } else {
    /*标记旧的VNode是否有nodeType*/
    const isRealElement = isDef(oldVnode.nodeType)
    if (!isRealElement && sameVnode(oldVnode, vnode)) {
      // patch existing root node
      /*是同一个节点的时候直接修改现有的节点*/
      patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
    } else {
      if (isRealElement) {
        // mounting to a real element
        // check if this is server-rendered content and if we can perform
        // a successful hydration.
        if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
          /*当旧的VNode是服务端渲染的元素，hydrating记为true*/
          oldVnode.removeAttribute(SSR_ATTR)
          hydrating = true
        }
        if (isTrue(hydrating)) {
          /*需要合并到真实Dom上*/
          if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
            /*调用insert钩子*/
            invokeInsertHook(vnode, insertedVnodeQueue, true)
            return oldVnode
          } else if (process.env.NODE_ENV !== 'production') {
            warn(
              'The client-side rendered virtual DOM tree is not matching ' +
              'server-rendered content. This is likely caused by incorrect ' +
              'HTML markup, for example nesting block-level elements inside ' +
              '<p>, or missing <tbody>. Bailing hydration and performing ' +
              'full client-side render.'
            )
          }
        }
        // either not server-rendered, or hydration failed.
        // create an empty node and replace it
        /*如果不是服务端渲染或者合并到真实Dom失败，则创建一个空的VNode节点替换它*/
        oldVnode = emptyNodeAt(oldVnode)
      }

      // replacing existing element
      /*取代现有元素*/
      const oldElm = oldVnode.elm
      const parentElm = nodeOps.parentNode(oldElm)

      // create new node
      createElm(
        vnode,
        insertedVnodeQueue,
        // extremely rare edge case: do not insert if old element is in a
        // leaving transition. Only happens when combining transition +
        // keep-alive + HOCs. (#4590)
        oldElm._leaveCb ? null : parentElm,
        nodeOps.nextSibling(oldElm)
      )

      // update parent placeholder node element, recursively
      if (isDef(vnode.parent)) {
        /*组件根节点被替换，遍历更新父节点element*/
        let ancestor = vnode.parent
        const patchable = isPatchable(vnode)
        while (ancestor) {
          for (let i = 0; i < cbs.destroy.length; ++i) {
            cbs.destroy[i](ancestor)
          }
          ancestor.elm = vnode.elm
          if (patchable) {
            /*调用create回调*/
            for (let i = 0; i < cbs.create.length; ++i) {
              cbs.create[i](emptyNode, ancestor)
            }
            // #6513
            // invoke insert hooks that may have been merged by create hooks.
            // e.g. for directives that uses the "inserted" hook.
            const insert = ancestor.data.hook.insert
            if (insert.merged) {
              // start at index 1 to avoid re-invoking component mounted hook
              for (let i = 1; i < insert.fns.length; i++) {
                insert.fns[i]()
              }
            }
          } else {
            registerRef(ancestor)
          }
          ancestor = ancestor.parent
        }
      }

      // destroy old node
      if (isDef(parentElm)) {
        /*移除老节点*/
        removeVnodes([oldVnode], 0, 0)
      } else if (isDef(oldVnode.tag)) {
        /*调用destroy钩子*/
        invokeDestroyHook(oldVnode)
      }
    }
  }

  /*调用insert钩子*/
  invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
  return vnode.elm
}
```

![img](https://i.loli.net/2017/08/27/59a23cfca50f3.png)

![img](https://i.loli.net/2017/08/27/59a2419a3c617.png)

Vue 的 diff 算法是将同层的节点进行比较，所以它的时间复杂度只有 O(n)，它的算法非常的高效。
从代码中我们也能看出，patch 中会用 sameVnode 判断老节点和新节点是否是同一个节点，如果是的话才会进行进一步的 patchVnode，否则就会创建新的 DOM，移除旧的 DOM。

## sameVnode

下面我们再来看看 sameVnode 中是如何来判定两个节点是同一个节点的。
```javaScript
/*
  判断两个VNode节点是否是同一个节点，需要满足以下条件
  key相同
  tag（当前节点的标签名）相同
  isComment（是否为注释节点）相同
  是否data（当前节点对应的对象，包含了具体的一些数据信息，是一个VNodeData类型，可以参考VNodeData类型中的数据信息）都有定义
  当标签是<input>的时候，type必须相同
*/
function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}

// Some browsers do not support dynamically changing type for <input>
// so they need to be treated as different nodes
/*
  判断当标签是<input>的时候，type是否相同
  某些浏览器不支持动态修改<input>类型，所以他们被视为不同类型
*/
function sameInputType (a, b) {
  if (a.tag !== 'input') return true
  let i
  const typeA = isDef(i = a.data) && isDef(i = i.attrs) && i.type
  const typeB = isDef(i = b.data) && isDef(i = i.attrs) && i.type
  return typeA === typeB || isTextInputType(typeA) && isTextInputType(typeB)
}
```

## patchVnode

```javaScript
 // diff算法 比较节点
function patchVnode (
  oldVnode,
  vnode,
  insertedVnodeQueue,
  ownerArray,
  index,
  removeOnly
) {
  /*两个VNode节点相同则直接返回*/
  if (oldVnode === vnode) {
    return
  }

  if (isDef(vnode.elm) && isDef(ownerArray)) {
    // clone reused vnode
    vnode = ownerArray[index] = cloneVNode(vnode)
  }

  const elm = vnode.elm = oldVnode.elm

  if (isTrue(oldVnode.isAsyncPlaceholder)) {
    if (isDef(vnode.asyncFactory.resolved)) {
      hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
    } else {
      vnode.isAsyncPlaceholder = true
    }
    return
  }

  // reuse element for static trees.
  // note we only do this if the vnode is cloned -
  // if the new node is not cloned it means the render functions have been
  // reset by the hot-reload-api and we need to do a proper re-render.
  /*
    如果新旧VNode都是静态的，同时它们的key相同（代表同一节点），
    并且新的VNode是clone或者是标记了once（标记v-once属性，只渲染一次），
    那么只需要替换elm以及componentInstance即可。
  */
  if (isTrue(vnode.isStatic) &&
    isTrue(oldVnode.isStatic) &&
    vnode.key === oldVnode.key &&
    (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
  ) {
    vnode.componentInstance = oldVnode.componentInstance
    return
  }

  // 执行一些组件钩子
  /*如果存在data.hook.prepatch则要先执⾏*/
  let i
  const data = vnode.data
  if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
    /*i = data.hook.prepatch，如果存在的话，见"./create-component componentVNodeHooks"。*/
    i(oldVnode, vnode)
  }

  // 查找新旧节点是否存在孩子
  const oldCh = oldVnode.children
  const ch = vnode.children

  // 属性更新
  if (isDef(data) && isPatchable(vnode)) {
    // cbs中关于属性更新的数组拿出来[attrFn, classFn, ...]
    /*调用update回调以及update钩子*/
    for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
    if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
  }

  // 判断是否元素
  if (isUndef(vnode.text)) { /*如果这个VNode节点没有text文本时*/
    // 双方都有孩子
    if (isDef(oldCh) && isDef(ch)) {
      /*新老节点均有children子节点，则对子节点进行diff操作，调用updateChildren*/
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
    } else if (isDef(ch)) {
      /*如果老节点没有子节点而新节点存在子节点，先清空elm的文本内容，然后为当前节点加入子节点*/
      if (process.env.NODE_ENV !== 'production') {
        checkDuplicateKeys(ch)
      }
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
    } else if (isDef(oldCh)) {
      /*当新节点没有子节点而老节点有子节点的时候，则移除所有ele的子节点*/
      removeVnodes(oldCh, 0, oldCh.length - 1)
    } else if (isDef(oldVnode.text)) {
      /*当新老节点都无子节点的时候，只是文本的替换，因为这个逻辑中新节点text不存在，所以直接去除ele的文本*/
      nodeOps.setTextContent(elm, '')
    }
  } else if (oldVnode.text !== vnode.text) {
    /*当新老节点text不一样时，直接替换这段文本*/
    nodeOps.setTextContent(elm, vnode.text)
  }
  /*调用postpatch钩子*/
  if (isDef(data)) {
    if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
  }
}
```

patchVnode 的过程是这样的：
1. 如果 oldVnode 和 Vnode 是同一个对象，那么久直接返回，不需要再更新
2. 如果新旧VNode都是静态的，同时它们的key相同（代表同一节点），并且新的VNode是clone或者是标记了once（标记v-once属性，只渲染一次），那么只需要替换elm以及componentInstance即可。
3. 如果vnode.text不是文本节点，新老节点均有children子节点，且新老节点的子节点不相同的时候，则对子节点进行diff操作，调用updateChildren，这个updateChildren也是diff的核心。
4. 如果老节点没有子节点而新节点存在子节点，先清空老节点DOM的文本内容，然后为当前DOM节点加入子节点
5. 当新节点没有子节点而老节点有子节点的时候，则移除该DOM节点的所有子节点。
6. 当新老节点都无子节点的时候，只是文本的替换。

## updateChildren

我们页面的dom是一个树状结构，上面所讲的patchVnode方法，是复用同一个dom元素，而如果新旧两个VNnode对象都有子元素，我们应该怎么去比较复用元素呢？这就是我们updateChildren方法所要做的事儿
```javaScript
/*
  diff核心方法，比较优化
*/
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
  let oldStartIdx = 0
  let newStartIdx = 0
  let oldEndIdx = oldCh.length - 1
  let oldStartVnode = oldCh[0]
  let oldEndVnode = oldCh[oldEndIdx]
  let newEndIdx = newCh.length - 1
  let newStartVnode = newCh[0]
  let newEndVnode = newCh[newEndIdx]
  let oldKeyToIdx, idxInOld, vnodeToMove, refElm

  // removeOnly is a special flag used only by <transition-group>
  // to ensure removed elements stay in correct relative positions
  // during leaving transitions
  const canMove = !removeOnly

  if (process.env.NODE_ENV !== 'production') {
    checkDuplicateKeys(newCh)
  }

  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    if (isUndef(oldStartVnode)) {
      /*向右靠拢*/
      oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
    } else if (isUndef(oldEndVnode)) {
      /*向左靠拢*/
      oldEndVnode = oldCh[--oldEndIdx]
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
      /*前四种情况其实是指定key的时候，判定为同一个VNode，则直接patchVnode即可，分别比较oldCh以及newCh的两头节点2*2=4种情况*/
      patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
      oldStartVnode = oldCh[++oldStartIdx]
      newStartVnode = newCh[++newStartIdx]
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
      patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
      oldEndVnode = oldCh[--oldEndIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
      patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
      canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
      oldStartVnode = oldCh[++oldStartIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
      patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
      canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
      oldEndVnode = oldCh[--oldEndIdx]
      newStartVnode = newCh[++newStartIdx]
    } else {
      /*
        生成一个key与旧VNode的key对应的哈希表（只有第一次进来undefined的时候会生成，也为后面检测重复的key值做铺垫）
        比如childre是这样的 [{xx: xx, key: 'key0'}, {xx: xx, key: 'key1'}, {xx: xx, key: 'key2'}]  beginIdx = 0   endIdx = 2  
        结果生成{key0: 0, key1: 1, key2: 2}
      */
      if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)

      /*如果newStartVnode新的VNode节点存在key并且这个key在oldVnode中能找到则返回这个节点的idxInOld（即第几个节点，下标）*/
      idxInOld = isDef(newStartVnode.key)
        ? oldKeyToIdx[newStartVnode.key]
        : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
      if (isUndef(idxInOld)) { // New element
        /*newStartVnode没有key或者是该key没有在老节点中找到则创建一个新的节点*/
        createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
      } else {
        /*获取同key的老节点*/
        vnodeToMove = oldCh[idxInOld]
        if (sameVnode(vnodeToMove, newStartVnode)) {
          /*如果新VNode与得到的有相同key的节点是同一个VNode则进行patchVnode*/
          patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
          /*因为已经patchVnode进去了，所以将这个老节点赋值undefined，之后如果还有新节点与该节点key相同可以检测出来提示已有重复的key*/
          oldCh[idxInOld] = undefined
          /*当有标识位canMove实可以直接插入oldStartVnode对应的真实Dom节点前面*/
          canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
        } else {
          // same key but different element. treat as new element
          /*当新的VNode与找到的同样key的VNode不是sameVNode的时候（比如说tag不一样或者是有不一样type的input标签），创建一个新的节点*/
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        }
      }
      newStartVnode = newCh[++newStartIdx]
    }
  }
  if (oldStartIdx > oldEndIdx) {
    /*全部比较完成以后，发现oldStartIdx > oldEndIdx的话，说明老节点已经遍历完了，新节点比老节点多，所以这时候多出来的新节点需要一个一个创建出来加入到真实Dom中*/
    refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
    addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
  } else if (newStartIdx > newEndIdx) {
    /*如果全部比较完成以后发现newStartIdx > newEndIdx，则说明新节点已经遍历完了，老节点多余新节点，这个时候需要将多余的老节点从真实Dom中移除*/
    removeVnodes(oldCh, oldStartIdx, oldEndIdx)
  }
}
```
乍一看这一块代码，可能有点儿懵。具体内容其实不复杂，我们先大体看一下整个判断流程，之后通过几个例子来详细过一下。

`oldStartIdx`、`newStartIdx`、`oldEndIdx`、`newEndIdx`都是指针，具体每一个指什么，相信大家都很明了，我们整个比较的过程，会不断的移动指针。

`oldStartVnode`、`newStartVnode`、`oldEndVnode`、`newEndVnode`与上面的指针一一对应，是它们所指向的`VNode`结点。

`while`循环在`oldCh`或`newCh`遍历结束后停止，否则会不断的执行循环流程。整个流程分为以下几种情况：

1、 如果`oldStartVnode`未定义，则`oldCh`数组遍历的起始指针后移一位。

```JavaScript
  if (isUndef(oldStartVnode)) {
    oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
  }
```

注：见第七种情况，`key`值相同可能会置为undefined

2、 如果`oldEndVnode`未定义，则`oldCh`数组遍历的起始指针前移一位。

```JavaScript
  else if (isUndef(oldEndVnode)) {
    oldEndVnode = oldCh[--oldEndIdx]
  } 
```

注：见第七种情况，`key`值相同可能会置为undefined

3、`sameVnode(oldStartVnode, newStartVnode)`，这里判断两个数组起始指针所指向的对象是否可以复用。如果返回真，则先调用`patchVnode`方法复用`dom`元素并递归比较子元素，然后`oldCh`和`newCh`的起始指针分别后移一位。

```JavaScript
  else if (sameVnode(oldStartVnode, newStartVnode)) {
    patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
    oldStartVnode = oldCh[++oldStartIdx]
    newStartVnode = newCh[++newStartIdx]
  }
```

4、`sameVnode(oldEndVnode, newEndVnode)`，这里判断两个数组结束指针所指向的对象是否可以复用。如果返回真，则先调用`patchVnode`方法复用`dom`元素并递归比较子元素，然后`oldCh`和`newCh`的结束指针分别前移一位。

```JavaScript
  else if (sameVnode(oldEndVnode, newEndVnode)) {
    patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
    oldEndVnode = oldCh[--oldEndIdx]
    newEndVnode = newCh[--newEndIdx]
  } 
```

5、`sameVnode(oldStartVnode, newEndVnode)`，这里判断`oldCh`起始指针指向的对象和`newCh`结束指针所指向的对象是否可以复用。如果返回真，则先调用`patchVnode`方法复用`dom`元素并递归比较子元素，因为复用的元素在`newCh`中是结束指针所指的元素，所以把它插入到`oldEndVnode.elm`的前面。最后`oldCh`的起始指针后移一位，`newCh`的起始指针分别前移一位。

```JavaScript
  else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
    patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
    canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
    oldStartVnode = oldCh[++oldStartIdx]
    newEndVnode = newCh[--newEndIdx]
  }
```

6、`sameVnode(oldEndVnode, newStartVnode)`，这里判断`oldCh`结束指针指向的对象和`newCh`起始指针所指向的对象是否可以复用。如果返回真，则先调用`patchVnode`方法复用`dom`元素并递归比较子元素，因为复用的元素在`newCh`中是起始指针所指的元素，所以把它插入到`oldStartVnode.elm`的前面。最后`oldCh`的结束指针前移一位，`newCh`的起始指针分别后移一位。

```JavaScript
  else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
    patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
    canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
    oldEndVnode = oldCh[--oldEndIdx]
    newStartVnode = newCh[++newStartIdx]
  }
```

7、如果上述六种情况都不满足，则走到这里。前面的比较都是头尾组合的比较，这里的情况，稍微更加复杂一些，其实主要就是根据`key`值来复用元素。

① 遍历`oldCh`数组，找出其中有`key`的对象，并以`key`为键，索引值为`value`，生成新的对象`oldKeyToIdx`。

```JavaScript
if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
function createKeyToOldIdx (children, beginIdx, endIdx) {
  let i, key
  const map = {}
  for (i = beginIdx; i <= endIdx; ++i) {
    key = children[i].key
    if (isDef(key)) map[key] = i
  }
  return map
}
```

② 查询`newStartVnode`是否有`key`值，并查找`oldKeyToIdx`是否有相同的`key`。

```JavaScript
  idxInOld = isDef(newStartVnode.key) ? oldKeyToIdx[newStartVnode.key] : null
```

③ 如果`newStartVnode`没有`key`或`oldKeyToIdx`没有相同的`key`，则调用`createElm`方法创建新元素，`newCh`的起始索引后移一位。

```JavaScript
  if (isUndef(idxInOld)) { // New element
    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
    newStartVnode = newCh[++newStartIdx]
  } 
```

④ `elmToMove`保存的是要移动的元素，如果`sameVnode(elmToMove, newStartVnode)`返回真，说明可以复用，这时先调用`patchVnode`方法复用`dom`元素并递归比较子元素，重置`oldCh`中相对于的元素为`undefined`，然后把当前元素插入到`oldStartVnode.elm`前面，`newCh`的起始索引后移一位。如果`sameVnode(elmToMove, newStartVnode)`返回假，例如`tag`名不同，则调用`createElm`方法创建新元素，`newCh`的起始索引后移一位。

```JavaScript
  elmToMove = oldCh[idxInOld]
  if (sameVnode(elmToMove, newStartVnode)) {
    patchVnode(elmToMove, newStartVnode, insertedVnodeQueue)
    oldCh[idxInOld] = undefined
    canMove && nodeOps.insertBefore(parentElm, newStartVnode.elm, oldStartVnode.elm)
    newStartVnode = newCh[++newStartIdx]
  } else {
    // same key but different element. treat as new element
    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
    newStartVnode = newCh[++newStartIdx]
  }
```

参考文章：
- [拉勾教育](https://kaiwu.lagou.com/)
- [VirtualDOM与diff(Vue实现)](https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown)
- [patch——diff](https://github.com/liutao/vue2.0-source/blob/master/patch%E2%80%94%E2%80%94diff.md)
