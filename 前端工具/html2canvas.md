# html2canvas

读取已经渲染好的 DOM 元素的结构和样式信息，然后基于这些信息去构建截图，呈现在 canvas 画布中。

## 使用场景

生成 canvas 下载图片
页面截图打印

## 安装

npm i html2canvas

## 使用

```html
<!-- 海报 -->
<div id="banner">
     
  <div>
    ...
    <!-- data-html2canvas-ignore：忽略不需要转化canvas的元素-->
    <el-button data-html2canvas-ignore @click="download">下载</el-button>
  </div>
</div>
```

```js
import html2canvas from 'html2canvas'

download() {
  html2canvas(document.querySelector('#banner'), {
    useCORS: true, // 是否尝试使用CORS从服务器加载图像
    allowTaint: false, // 是否允许跨域图像污染画布
    scale: 4, // 放大倍数 提高图片质量
    width: dom.offsetWidth, // 直接取得需要转为图片的dom元素的宽高
    height: dom.offsetHeight,
  }).then(canvas => {
    // 去除锯齿
    let context = canvas.getContext('2d')
    context.mozImageSmoothingEnabled = false
    context.webkitImageSmoothingEnabled = false
    context.msImageSmoothingEnabled = false
    context.imageSmoothingEnabled = false
    // 加载动画
    this.posterLoading = Loading.service({
      lock: true,
      text: '下载海报中……',
      background: 'rgba(0, 0, 0, 0.5)',
    })
    let self = this
    let url = canvas.toDataURL()
    let xhr = new XMLHttpRequest()
    xhr.open('GET', url, true)
    xhr.responseType = 'blob'
    xhr.onload = function () {
      if (this.status === 200) {
        let blob = this.response
        let fileName = self.posterTitle.includes('.') ? self.posterTitle + '.png' : self.posterTitle
        if (navigator.msSaveBlob == null) {
          const link = document.createElement('a')
          const event = new MouseEvent('click')
          link.download = fileName
          link.href = URL.createObjectURL(blob)
          link.dispatchEvent(event)
        } else {
          navigator.msSaveBlob(blob, fileName)
        }
      }
      setTimeout(() => {
        self.posterLoading.close()
      }, 200)
    }
    xhr.send()


    // 模拟下载
    const link = document.createElement('a')
    const event = new MouseEvent('click')
    link.download = this.activityTitle
    link.href = canvas.toDataURL()
    link.dispatchEvent(event)
  })
},
```

## 分辨率

scale：
scalewindow.devicePixelRatio 用于渲染的缩放比例，默认为浏览器设备像素比

## 问题

1. 图片不显示

   可能原因：

   - html2canvas 没有配置 useCORS: true // 是否尝试使用 CORS 从服务器加载图像
   - 后台没有配置跨域
   - 后台配置跨域了但控制台还是显示跨域错误

     原因：
     浏览器本地缓存错误，即重复请求了相同的接口，导致跨域错误。
     因为 html2canvas 只会根据 dom 结果请求并且对图片来源有限制。当 dom 有图片就会请求网络，所以 canvas 第二次请求实际上是从缓存获取，如果从缓存读取类似本地读取照片则会有跨域限制。

     验证：
     开启控制台网络的禁止缓存选项

     解决：

     - 加 cdn，但要加钱
     - 配置 CORS 规则，附加相关 Header，选中返回 vary: origin 头以避免本地缓存错乱。但是选中后可能会造成浏览器访问次数或 CDN 回源次数增加。

2. 样式错乱，与网页不一

   部分 css 样式不兼容，比如 clac 计算样式等等

3. 下载不了或无反应

   可能性：
   需要在页面渲染完毕再下载

4. 报错 TypeError: Failed to execute 'addColorStop' on 'CanvasGradient'

   可能性：
   页面渲染后存在两个相同 canvasId 的 canvas

## 参考

[html2canvas 实现浏览器截图的原理（包含源码分析的通用方法）](https://www.1024sou.com/article/209309.html)

[html2canvas 兼容性](https://html2canvas.hertzen.com/features)
