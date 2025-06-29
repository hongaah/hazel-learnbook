# webworker

一个独立于 JavaScript 主线程的独立线程，在里面执行需要消耗大量资源的操作不会堵塞主线程。

Web Worker 是 HTML5 标准的一部分，这一规范定义了一套 API，允许我们在 js 主线程之外开辟新的 Worker 线程，并将一段 js 脚本运行其中，它赋予了开发者利用 js 操作多线程的能力。

JavaScript 是单线程的语言，如果在浏览器中需要执行一些大数据量的计算，页面上的其他操作就会因为来不及响应而出现卡顿的情况，Web Worker 的出现为 js 提供了一个多线程的环境，让更多复杂计算拿到另一个环境中去完成，计算完之后再提供给主进程使用，前端页面可以只负责界面渲染，让用户体验更流畅。

Web Worker 在处理一些复杂的计算或耗时的任务时非常有用，可以提高网页的性能和响应能力。然而，它并不适用于所有情况，需要根据具体的应用场景和需求来决定是否使用 Web Worker。

web workers 已经被大多数浏览器支持，使用上基本不用考虑兼容问题。

## 使用限制

- 同源限制
- 接口限制（由于 Web Worker 运行在独立的线程中，它无法直接访问主线程的 DOM、全局变量和大多数浏览器 API。window 作用域下的部分方法不可使用，如 DOM 对象、window.alert 和 window.confirm 等方法。）
- 文件限制（无法加载本地 js 文件，必须使用线上地址。）
- 记得关闭（worker 会占用一定的系统资源，在相关的功能执行完之后，一定要记得关闭 worker，无论是在主线程关闭 worker，还是在 worker 线程内部关闭 worker，worker 线程当前的 Event Loop 中的任务会继续执行。至于 worker 线程下一个 Event Loop 中的任务，则会被直接忽略，不会继续执行。但是在主线程关闭则通信会被关闭，不会再接收到信息，worker 线程关闭则会在当前 Event Loop 中接收）
- 使用时注意 this 指向

## webworker 的使用

通过 postMessage 实现主线程和 Web Worker 双向通信

关键代码：

- const worker = new Worker('xx url')
- worker.onmessage = (e) => {e.data}
- worker.postMessage('xxx')
- worker.addEventListener(event: 'error' | 'messageerror', (err) => {})
- worker.terminate()

- self.onmessage = (e) => {e.data}
- self.postMessage('xxx')
- self.addEventListener(event: 'error' | 'messageerror', (err) => {})
- self.close()

```js
import React, { Component } from 'react'

import worker_script from './worker-script'

export default class PageHome extends Component {
  constructor(props) {
    super(props)
    this.state = {}
  }

  componentDidMount() {
    const worker = new Worker(worker_script)
    worker.postMessage('dadada')

    worker.onmessage = function (event) {
      console.log(`Received message ${event.data}`)

      // 执行完了以后关闭 webworker
      // worker.terminate()
    }
  }
}
```

```js worker-script.js
const workercode = () => {
  // 在后台执行任务
  self.onmessage = function (e) {
    console.log('Message received from main script')
    let workerResult = performData(e.data)
    self.postMessage(workerResult)
  }

  function performData(data) {
    // 执行耗时的某些操作
    return `Received from main: ${data}`
  }
}

// 在线生成同源线上地址
let code = workercode.toString()
code = code.substring(code.indexOf('{') + 1, code.lastIndexOf('}'))

const blob = new Blob([code], { type: 'application/javascript' })
const worker_script = URL.createObjectURL(blob)

export default worker_script
```

### @koale/useworker

react hooks

```js
import React from 'react'
import { useWorker } from '@koale/useworker'

const numbers = [...Array(5000000)].map(e => ~~(Math.random() * 1000000))
const sortNumbers = nums => nums.sort()

const Example = () => {
  const [sortWorker] = useWorker(sortNumbers)

  const runSort = async () => {
    const result = await sortWorker(numbers) // non-blocking UI
    console.log(result)
  }

  return (
    <button type="button" onClick={runSort}>
      Run Sort
    </button>
  )
}
```

## 参考

<https://juejin.cn/post/7139718200177983524>
