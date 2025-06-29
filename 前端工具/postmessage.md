# postMessage

[跨窗口通信](https://zh.javascript.info/cross-window-communication)

1.父窗口给子窗口发送消息的方式：
iFrame.contentWindow.postMessage('MessageFromIndex1','\_');
其实就是在父窗口中操作子窗口发消息，然后让子窗口接收自己刚才发的消息。

2.子窗口给父窗口发送消息的方式：
parent.postMessage({msg: 'MessageFromIframePage'}, '\_');
注意：此处 parent === window.parent，即子窗口的父窗口
其实就是在子窗口中操作父窗口发消息，然后让父窗口接收自己刚才发的消息。

总结：所谓的跨窗口发送消息，就是通过在别的窗口操作本窗口发送消息，然后本窗口再自己接收的方式实现。

```js
// 子页面
// 子窗口传父
function sendMessage() {
  // top = window.parent = window.top
  top?.postMessage(
    {
      data: {
        code: 'success',
        text: '子頁面的test',
      },
    },
    '*'
  )
}

// 子窗口接收父
const result = ref({})
window.addEventListener(
  'message',
  (event: any) => {
    result.value = event.data.data
  },
  false
)
```

```js
// 父窗口
// 父接收子
const result = ref()
onMounted(() => {
  window.addEventListener('message', (event: any) => {
    const data = event.data.data
    if (data?.code === 'success') {
      ElMessage(data?.text)
      result.value = data?.text
    }
  })

  // 父发子先定义
  myFrameInstance.value = myFrameRef.value?.contentWindow
})

// 父发子
const myFrameRef = ref()
const myFrameInstance = ref()
function sendMessage() {
  myFrameInstance.value.postMessage(
    {
      data: 'Hello Message From Bi!',
    },
    '*'
  )
}
```

可参考库：https://github.com/dollarshaveclub/postmate

## window.top.postmessage 和 window.postmessage

window.top.postmessage 才能传值

```js
window.top.postMessage({
  name: 'test'
}, '*')
```

## 多窗口嵌套传值

```js
const sendData = {}
const windowList = []
function getWindow (currentWindow) {
  if (currentWindow.top.opener) { // 多窗口层级
    if (currentWindow.top.vm) { // 新后台
      return currentWindow.top
    } else {
      windowList.push(currentWindow.top)
      return getWindow(currentWindow.top.opener)
    }
  } else { // 只有一个层级
    return currentWindow.top
  }
}

function closeWindows() {
  windowList.forEach(item => {
    if (item === window) {
      setTimeout(() => {
        item.close()
      }, 1)
    } else {
      item.close()
    }
  })
}

getWindow(window).postMessage(sendData, '*')
closeWindows()
```

## window.opener.xx

父：window.xx = () => {}

子：
window.opener.xx()
widnow.close()

## MessageChannel

https://www.jianshu.com/p/4f07ef18b5d7
