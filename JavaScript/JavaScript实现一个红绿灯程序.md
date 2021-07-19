# JavaScript实现一个红绿灯程序

这个题算是一个比较经典的面试题了，代码实现也很简单，下面我们就来看看是怎样实现的

## setTimeout 版本

setTimeout 是 JavaScript 中的炸弹定时器，setTimeout 执行完就会立即被回收，那么我们就可以利用这个特点，在回收之前再去调用一个 setTimeout ，这样就能输出下一个灯的信息，当所有的灯都输出完毕后，我们就可以再递归调用一下程序，这样就会一直循环输出红绿灯了

```js
function trafficLights() {
  console.log('绿灯')
  setTimeout(() => {
    console.log('黄灯')
    setTimeout(() => {
      console.log('红灯')
      setTimeout(() => {
        fn()
      }, 2000)
    }, 2000)
  }, 2000)
}
trafficLights()
```

## Promise 版本

setTimeout 版本实现起来不是那么优雅，ES6的出现，可以让我们使用 async 和 Promise 去实现这个操作

```js
function red() {
  console.log('红灯')
}

function green() {
  console.log('绿灯')
}

function yellow() {
  console.log('黄灯')
}

function wait(time, cb) {
  return new Promise((resolve) => {
    setTimeout(() => {
      cb()
      resolve()
    }, time)
  })
}

async function main() {
  while(true) {
    await wait(3000, red)
    await wait(2000, green)
    await wait(2000, yellow)
  }
}

main()
```

可以看到 Promise 实现起来代码逻辑比较清晰，而且我们可以使用 while 循环来代替递归
