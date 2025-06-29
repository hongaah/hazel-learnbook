# fetch

[Fetch API 教程](https://www.ruanyifeng.com/blog/2020/12/fetch-tutorial.html)

## features

fetch() 的功能与 XMLHttpRequest 基本相同，但有三个主要的差异

- promise
- 模块化设计，API 分散在多个对象上（Response、Request、Headers）
- 数据流（stream 对象）处理数据，可以分块读取，有利于提高网站性能表现，减少内容占用；XMLHTTPRequest 对象不支持数据流，所有的数据必须放在缓存里，不支持分块读取，必须等待全部拿到后，再一次性吐出来
- 请求是否成功标识（只有网络错误，或者无法连接时，fetch()才会报错，其他情况都不会报错，而是认为请求成功）
- 捕获错误 try catch

### 模块化对象

#### Response 对象

fetch() 请求成功以后，得到的是一个 Response 对象，response 是一个 Stream 对象。

```js
const response = await fetch(url)

// 同步属性
console.log('response.ok', response.ok) // 返回一个布尔值，表示请求是否成功，true对应 HTTP 请求的状态码 200 到 299，false对应其他的状态码
console.log('response.status', response.status) // 回一个数字，表示 HTTP 回应的状态码
console.log('response.statusText', response.statusText) // 返回一个字符串，表示 HTTP 回应的状态信息（例如请求成功以后，服务器返回"OK"）
console.log('response.url', response.url) // 返回请求的 URL。如果 URL 存在跳转，该属性返回的是最终 URL
console.log('response.type', response.type) // 返回请求的类型
console.log('response.redirected', response.redirected) // 返回一个布尔值，表示请求是否发生过跳转

// 异步方法
// Response对象根据服务器返回的不同类型的数据，提供了不同的读取方法。
await response.text() // 得到文本字符串，比如 HTML 文件。
await response.json() // 得到 JSON 对象。
await response.blob() // 得到二进制 Blob 对象，比如图片文件。
await response.formData() // 得到 FormData 表单对象，主要用在 Service Worker 里面，拦截用户提交的表单，修改某些数据以后，再提交给服务器。
await response.arrayBuffer() // 得到二进制 ArrayBuffer 对象，比如音频文件。

// Stream 对象只能读取一次，读取完就没了，再读取会报错，可以通过创建Response对象的副本，实现多次读取。
const response2 = response.clone()
```

#### Headers 对象

Response 对象还有一个Response.headers属性，指向一个 Headers 对象，对应 HTTP 回应的所有标头。

```js
const headers = response.headers

Headers.get() // 根据指定的键名，返回键值。
Headers.has() //  返回一个布尔值，表示是否包含某个标头。
Headers.set() // 将指定的键名设置为新的键值，如果该键名不存在则会添加。
Headers.append() // 添加标头。
Headers.delete() // 删除标头。
Headers.keys() // 返回一个遍历器，可以依次遍历所有键名。
Headers.values() // 返回一个遍历器，可以依次遍历所有键值。
Headers.entries() // 返回一个遍历器，可以依次遍历所有键值对（[key, value]）。
Headers.forEach() // 依次遍历标头，每个标头都会执行一次参数函数。
```

#### ReadableStream 对象

Response 对象有一个 Response.body.getReader() 属性，指向一个 ReadableStream 对象，可以用来分块读取内容，应用之一就是显示下载的进度

```js
const response = await fetch('flower.jpg')
const reader = response.body.getReader() // 返回一个遍历器。这个遍历器的read()方法每次返回一个对象，表示本次读取的内容块。

while(true) {
  // done属性是一个布尔值，用来判断有没有读完；value属性是一个 arrayBuffer 数组，表示内容块的内容，而value.length属性是当前块的大小。
  const {done, value} = await reader.read()

  if (done) {
    break
  }

  console.log(`Received ${value.length} bytes`)
}
```

#### AbortController 对象

取消fetch()请求需要使用 AbortController 对象

```js
// 
  const controller = new AbortController()
  const signal = controller.signal

  const postreq = await fetch(getContentByTypeCodeUrl, {
    method: 'post',
    headers: {
      'Content-Type': 'application/json;charset=utf-8'
    },
    body: JSON.stringify({
      platformSource: 0,
      typeCode: 'orderMust'
    })
    signal: signal // 指定一个 AbortSignal 实例，用于取消fetch()请求
  })
  signal.addEventListener('abort', (e) => {
    console.log('abort', e)
  })

  // 用于取消 fetch 请求
  setTimeout(() => {
    controller.abort()
  }, 2000)
```

## 请求数据类型

```js
let params = {}
const response = await fetch('xxx', {
  method: 'POST',
  body: params
})
```

1. 提交 JSON 数据
```js
params = JSON.stringify(user)
```

2. 提交表单
```js
const form = document.querySelector('form')
params = new FormData(form)
```

3. 文件上传
```js
const input = document.querySelector('input[type="file"]')
const data = new FormData()
data.append('file', input.files[0])
data.append('user', 'foo')
params = data
```

4. 直接上传二进制数据
```js
let blob = await new Promise(resolve =>   
  canvasElem.toBlob(resolve, 'image/png')
)
params = blob
```

5. get 请求的参数转对象传递

直接传：❎ Failed to execute 'fetch' on 'Window': Request with GET/HEAD method cannot have body
借助 Qs：✅

```js
const params = {
  path: url,
  color: 128,
  level: 9
}

fetch(`/sharp-compress-gif/compression?${Qs.stringify(params)}`, {
  method: 'GET',
})
```

## usage

```js
try {
  /** GET 请求 */
  // fetch() 接收到的 response 是一个 Stream 对象
  const response = await fetch(queryCouponConfigListUrl)
  const resData = await response.json()

  /** POST 请求 */
  const postreq = await fetch(getContentByTypeCodeUrl, {
    method: 'post',
    headers: {
      'Content-Type': 'application/json;charset=utf-8'
    },
    body: JSON.stringify({
      platformSource: 0,
      typeCode: 'orderMust'
    }),
    referrer: 'about:client',
    referrerPolicy: 'no-referrer-when-downgrade',
    mode: 'cors', // 允许跨域请求
    credentials: 'same-origin', // 同源请求时发送 Cookie，跨域请求时不发送
    cache: 'default', // 先在缓存里面寻找匹配的请求
    redirect: 'follow', // fetch()跟随 HTTP 跳转
    integrity: '', // 指定一个哈希值，用于检查 HTTP 回应传回的数据是否等于这个预先设定的哈希值。比如，下载文件时，检查文件的 SHA-256 哈希值是否相符，确保没有被篡改。
    keepalive: true, // 该属性用于页面卸载时，告诉浏览器在后台保持连接，继续发送数据
    signal: signal // 指定一个 AbortSignal 实例，用于取消fetch()请求
  })
  const postresult = await postreq.json()
  console.log('result', postresult)

} catch (e) {
  console.log('error', e)
}
```