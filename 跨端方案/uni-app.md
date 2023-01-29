# uniapp

### 调试

[官方调试教程](https://uniapp.dcloud.net.cn/tutorial/run-and-debug.html)

#### HBuilder 内置浏览器

- 优点：在线同步运行前端项目，边改边看，接口可跨域成功，可查看请求信息
- 弊端：运行不了 html5+ 相关的

#### 真机调试

- 连接方式：usb 连接好后，通过【运行】-【运行到手机或模拟器】-【运行到 Android app 基座】
- 优点：可还原手机 app 运行环境；可以热更新
- 弊端：更新一点点都要重启 app，如果配置了账号密码、环境都要重新设置；需要抓包看网络请求；android 可用，ios 不可（截止至 22.10.25）

#### 模拟器

- 优点：可还原手机 app 运行环境；可以热更新
- 弊端：windows 只能运行安卓模拟器；需要抓包看网络请求；运行有点慢

连接教程大部分可以参考: <https://www.jianshu.com/p/f01efc0e4609>

检查模拟器连接情况：新版本 HbuilderX 行不通了，可通过【运行】-【运行到手机或模拟器】-【运行到 Android app 基座】查看是否有设备名类似为【127.0.0.1:7555】

运行成功后一直闪退问题：回退 Hb 版本 3.4.x 之后

打开调试控制台：【视图】-【显示 webview 调试控制台】

```bash
# 连接
adb connect 127.0.0.1:7555
adb devices
# HbuilderX 默认安卓端口26944
```

[模拟器端口号](https://www.jianshu.com/p/5eb851ff1a16)

模拟器通过 bin 文件可以通过终端链接手机，操作内部环境，读写删手机文件

#### chrome webview 调试

真机或模拟器调试可以用 chrome 调试页面样式，当程序运行到基座后，打开 HubuilderX 的视图 -> 显示 Webview 调试控制台。

chrome 调试 app：<https://blog.51cto.com/u_10624715/5270841>

uni-app 只有 v3 模式下的 vue 页面支持 webview 调试：<https://ask.dcloud.net.cn/article/36599>

### 离线打包

uniapp 分两种更新机制：

- 完整包更新，包括 wgt 包 + uni 原生引擎 + 原生插件（这种方式比较繁琐，即每次更新之后就要打云包，更新整包，用户体验也不好）
- 资源包更新，包括 wgt（因第一种方法想到利用资源更新，用户体验大大提升，用户更新之后无需跳转到应用市场或者浏览器去重新更包，即无感更新）。wgt 是资源升级包，完整的前端资源，将前端的全部内容编译打包成 wgt。只有前端资源 wgt 可以热更新。

uniapp 离线打包（wgt）：

1. 配置 manifeat.json 基础配置中的 appID、应用版本名称、应用版本号
2. 点击窗口的【发行】-【原生 app-制作应用 wgt 包】

[uni-app 跨平台应用开发之实现资源在线升级](https://www.php.cn/uni-app/489365.html)

### 混合开发

uniapp 编译提供的 wgt 包，宿主在原生 app 中

uniapp 向 app 发送指令，通过回调获取信息：uni.sendNativeEvent

uniapp 监听 app 的指令：uni.onNativeEventReceive((event, data) => {})

#### others

app 启用小程序时，在 onLaunch(opts) 的 opts 的可以获取到一些信息

```
获取 app 版本号：opts.referrerInfo.extraData.version
是否 debug 模式：opts.referrerInfo.extraData.isDebug
```

### HTML5+ 扩展规范

<https://www.html5plus.org/doc/h5p.html>
HTML5+是中国 HTML5 产业联盟的扩展规范，基于 HTML5 扩展了大量调用设备的能力，使得 web 语言可以想原生语言一样强大。

html5+ 是 hbulider 利用自己的 IDE 结合不同平台的接口再加上 html5 的东西开发出来的一套框架，它有自己的使用规范，它允许和提供了一些接口和函数来让 web 开发者实现原生 app 所能实现的功能，Dcloud（开发 hublider 的公司）还在 hublider 提供了云打包功能，几乎是打包发布，帮那些不熟悉原生开发的开发人员节省了很多时间，国内类似的还有 Apicloud(提供了很多原生模块，而且编译发布什么的要比 hbulider 好一些，SDK 更新的比较勤，维护还是做的很不错的，最近在和 Dcloud 打官司)。hublider 的 native.js 就是一个接口库，调里面的接口就可以利用 hbulider 为你实现的和原生 api 结合的一些功能。

### 问题

#### 项目 app 与小程序的不同点

- 分享方式改成方法 util.share
- 目录结构变更，链接本地地址需修正
- 网络请求变更域名是根据本地缓存环境，所以网络请求设置的 url 的 host 链接方式变更
- 配置新路由后`node router.js` 生效
- 有些样式错乱需要改像素单位 upx
