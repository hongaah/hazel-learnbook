# electron

electron 分为两个进程主进程和渲染进程

- 主线程：index.js
- Remote 模块: 在渲染进程里（比如 index.html 里面加载了一些 js 文件，那里面的 js 如果要使用到 BrowserWindow 这些属性的话就必须使用 remote）使用 remote 模块, 你可以调用 main 进程对象的方法

## 调试

```bash
# 开发热更新
$ nodemon --watch . --exec electron .
```

## electron 编译安装包

[electron 安装+运行+打包成桌面应用+打包成安装文件+开机自启动](https://www.cnblogs.com/kakayang/p/9559777.html)

应用程序的绿色版本：无需安装，拷贝整个文件目录之后即可使用

客户端应用程序安装包：安装之后通过桌面快捷方式的形式去访问（Inno Setup）

```bash
# 安装electron打包工具electron-packager
$ npm install electron-packager -g
# 编译绿色安装包
$ electron-packager . appName --win --out ./appName --arch=x64 --app-version=1.0.0 --electron-version=1.0.0 --overwrite  --icon=./src/assets/favicon.ico --ignore=node_modules
# 客户端应用程序安装包
Inno Setup 通过软件向导生成 setup.exe
```

##### electron-packager

electron-packager 的命令结构如下（根据实际情况修改）：

- .：需要打包的应用目录（即当前目录）
- appName：应用名称
- --win：打包平台（以 Windows 为例）
- --out ../appName：输出目录
- --arch=64：64 位
- --app-version=0.0.1：应用版本
- --electron-version=2.0.0：项目使用的 electron 版本

electron-packager 编译的安装包跑不起来：

- 有报错说明代码或调用出问题，会出现在线预览没事，但编译包就出问题
- 编译包定义的 electron 版本是否与项目用的一致，新版本可支持更多 api
