树的操作在前端的工作中占据了比较重要的位置，我总结了一些我平时处理树数据的方法
## 情况一 将列表转换成 tree
有的时候后端比较懒，不愿意处理树数据，给的是一个列表数据，这时候就需要前端将列表转换成 tree 数据
```js
// 列表数据
const list = [
  { id: 01, pid: null },
  { id: 02, pid: null },
  { id: 03, pid: 01 },
  { id: 04, pid: 03 },
  { id: 05, pid: 01 },
  { id: 06, pid: 03 },
  { id: 07, pid: 02 },
  { id: 09, pid: 02 },
  { id: 10, pid: 07 },
  { id: 11, pid: 07 },
]

// 转换后的数据
const tree = [
  { 
    id: 01,
    pid: null,
    children: [
      { 
        id: 03,
        pid: 01,
        children: [
          { id: 04, pid: 03 },
          { id: 06, pid: 03 },
        ]
      },
      { id: 05, pid: 01 },
    ]
  },
  { 
    id: 02, 
    pid: null,
    children: [
      { 
        id: 07,
        pid: 02,
        children: [
          { id: 10, pid: 07 },
          { id: 11, pid: 07 },
        ]
      },
      { id: 09, pid: 02 },
    ]
  },
]
```
我们还是来看一下怎么实现的
```js
/**
 * @description 查找包含自身节点的父代节点
 * @param list 列表数据
 * @param id 节点 id
 * @param pid 节点的父id
 */
function listToTree(list, id, pid) {
  list.forEach((node) => {
    const pNdoe = list.find((row) => row[id] === node[pid])

    if (pNdoe) {
      pNdoe.children = pNdoe.children || []
      pNdoe.children.push(node)
    }
  })
  return list.filter((node) => !node[pid])
}

const treeNode = listToTree(list, 'id', 'pid')
```
当然了，处理数据很简单，但是如果当你的后端不处理的话，一定要无情地怼回去，前端最好还是只展示数据，最好不要在浏览器处理一些逻辑

## 情景二 遍历树 给树中节点添加属性
像我们在用menu展示菜单或者下拉树形选择框的时候，可能后端给我们的数据并不是我们想要的结构，例如我们需要的 value 字段，后端可能返回的是 id；title 字段，后端返回的确实 label，我们需要在树中的节点添加属性，来满足我们想要的结构
- 方法1 普通递归
```js
/**
 * @description 遍历 tree 给 tree 添加数据
 * @param tree tree 数据
 * @param keyField 对应 node 节点的 value 字段
 * @param labelField 对应 node 节点的 title 字段
 */
function formatTree(tree, keyField, labelField) {
  for(const node of tree) {
    node.value = node[keyField]
    node.title = node[labelField]
    if (node.children && node.children.length) {
      formatTree(node.children, keyField, labelField)
    }
  }
  return tree
}

const newTree = formatTree(tree, 'key', 'label')
```

- 方法2 尾递归
普通递归在树的层级比较深的情况会比较慢，或者会爆栈，我们使用尾递归进行优化
```js
/**
 * @description 遍历 tree 给 tree 添加数据
 * @param tree tree 数据
 * @param keyField 对应 node 节点的 value 字段
 * @param labelField 对应 node 节点的 title 字段
 */
function formatTree(tree, keyField, labelField) {
  const bfs = (treeData, key, label) => {
    for(const node of treeData) {
      node.value = node[key]
      node.title = node[label]
      if (node.children && node.children.length) {
        formatTree(node.children, key, label)
      }
    }
    return tree
  }
  return bfs(tree, keyField, labelField)
}

const newTree = formatTree(tree, 'key', 'label')
```

## 情景三 在树中查找节点
有的时候，我们有树节点的一个 id，我们需要根据这个节点 id 到树中找到这个节点
- 方法1 递归查找
```js
/**
 * @description 查找包含自身节点的父代节点
 * @param tree 需要查找的树数据
 * @param curKey 当前节点key
 * @param keyField 自定义 key 字段
 * @param node 找到的node 可以不传
 */
function findCurNode(tree, curKey, keyField, node = null) {
  tree.forEach((item) => {
    if (item[keyField] === curKey) {
      node = item
    }
    if (item.children && item.children.length) {
      const findChildren = findCurNode(item.children, curKey, keyField, node)
      if (findChildren) {
        node = findChildren
      }
    }
  })
  return node
}

findCurNode(tree, 10, 'id')
```
- 方法2 深度优先遍历查找
方法1的递归查找实现和理解上比较简单，但是性能会差很多，而且如果树的层级特别深的话，可能会爆栈
所以我们可以使用经典算法深度优先遍历来查找树的节点
```js
/**
 * @description 查找包含自身节点的父代节点
 * @param tree 需要查找的树数据
 * @param curKey 当前节点key
 * @param keyField 自定义 key 字段
 * @param node 找到的node 可以不传
 */
function findCurNode(tree, curKey, keyField, node = null) {
  const stack = []
  for (const item of tree) {
    if (item) {
      stack.push(item)
      while (stack.length) {
        const temp = stack.pop()

        if (temp[keyField] === curKey) {
          node = temp
          break
        }

        const children = temp.children || []
        for (let i = children.length - 1; i >= 0; i--) {
            stack.push(children[i])
        }
      }
    }
  }
  return node
}

const node = findCurNode(tree, 10, 'id')
```

## 情景四 查找包含当前节点的所有父级节点
有的时候，前端在处理级联选择框的时候，需要回显选项，如果是级联菜单是懒加载的，就需要找到所有的父级节点
```js
  /**
 * @description 查找包含自身节点的父代节点
 * @param tree 需要查找的树
 * @param func 判断是否节点是否相等的函数
 * @param keyField 自定义 key 字段
 * @param isNode 是否是节点，false 为 node 节点 ； true 为 key
 * @param path 结果数组 可以不传
 */
function findTreeSelect(tree, func, keyField, isNode = false, path = []) {
  if (!tree) { return [] }
  for (const data of tree) {
    isNode ? path.push(data) : path.push(data[keyField])
    if (func(data)) { return path }
    if (data.children && data.children.length) {
      const findChildren = findTreeSelect(data.children, func, keyField, isNode, path)
      if (findChildren.length) { return findChildren }
    }
    path.pop()
  }
  return []
}

const nodes = findTreeSelect(tree, (data) => data.id === 10, 'id')
```
