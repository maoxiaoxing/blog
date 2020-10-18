深度优先搜索和广度优先搜索是比较常见的算法，今天我们用js来实现以下
首先我们创建一下数据
```
const tree = {
    key: '第一层-1',
    children: [
        {
            key: '第二层-1-1',
            children: [
                {
                    key: '第三层-1-1',
                    children: [],
                },
                {
                    key: '第三层-1-2',
                    children: [],
                }
            ]
        },
        {
            key: '第二层-2-1',
            children: [
                {
                    key: '第三层-2-1',
                    children: [],
                }
            ]
        },
        {
            key: '第二层-3-1',
            children: [
                {
                    key: '第三层-3-1',
                    children: [],
                }
            ]
        },
    ]
}
```
1.深度优先搜索递归实现
优点：实现简单
缺点：数据层级特别深的情况下，容易发生爆栈
```
const deepTraversal1 = (node, nodeList = []) => {
    if (node !== null) {
        nodeList.push(node)
        let children = node.children
        for (let i = 0; i < children.length; i++) {
        deepTraversal1(children[i], nodeList)
        }
    }
    return nodeList
}


console.log(deepTraversal1(tree), '深度优先搜索的递归实现')
```
来看一下结果
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200810213701456-334530902.png)

2. 深度优先搜索非递归实现
优点：搜索速度快，不会发生爆栈
```
const deepTraversal2 = (node) => {
    let stack = []
    let nodes = []

    if (node) {
        stack.push(node)
        
        while(stack.length) {
            const temp = stack.pop()
            const children = temp.children

            nodes.push(temp)

            for(let i = children.length-1; i >=0; i--) {
                stack.push(children[i])
            }
        }
    }
    return nodes
}

console.log(deepTraversal2(tree), '深度优先搜索的非递归实现')
```
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200810214011514-1311275305.png)


广度优先搜索
```
const widthTraversal = (node) => {
    let nodes = []
    let queue = []
    if (node) {
        queue.push(node)
        while (queue.length) {
        let item = queue.shift()
        let children = item.children
        nodes.push(item)
        // 队列，先进先出
        for (let i = 0; i < children.length; i++) {
            queue.push(children[i])
        }
        }
    }
    return nodes
}

console.log(widthTraversal(tree))
```
![](https://img2020.cnblogs.com/blog/1575596/202008/1575596-20200810214347372-2061372421.png)

