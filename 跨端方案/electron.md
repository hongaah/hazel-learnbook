# electron

## 概述

Electron是一个使用 JavaScript、HTML 和 CSS 构建跨平台桌面应用的开源框架。它通过将 Chromium 和 Node.js 合并到同一个运行时环境中，并为操作系统本地功能提供一套丰富的API，使得开发者可以沿用Web技术栈，轻松构建功能强大的桌面应用。

## 核心概念：主进程与渲染进程

Electron应用的核心架构基于两种进程：**主进程（Main Process）**和**渲染进程（Renderer Process）**。

🌰：以 [electron](https://github.com/hongaah/electron) 为例：

*   **主进程**：每个Electron应用有且仅有一个主进程。它作为应用的入口点，在Node.js环境中运行，负责管理应用的生命周期、创建和管理窗口（`BrowserWindow`实例）、处理原生操作系统交互（如菜单、对话框、托盘图标等）。主进程是所有渲染进程的父进程，并掌管着系统资源。

    在项目中，`index.js` 就是主进程的脚本。它负责创建应用窗口、处理窗口关闭等事件。

    ```javascript
    // index.js
    const { app, BrowserWindow } = require('electron')

    function createWindow() {
      const win = new BrowserWindow({ /* ... */ })
      win.loadFile('./dist/index.html')
      // ...
    }

    app.on('ready', createWindow)
    ```

*   **渲染进程**：每个通过`BrowserWindow`加载的网页都在其自己的渲染进程中运行。渲染进程负责UI的渲染，它运行在Chromium环境中，因此可以使用所有浏览器API，如`document`、`window`等。此外，通过Electron的特殊处理，渲染进程也能够访问Node.js模块，这极大地扩展了其能力。

    项目中的`src/feature/WriteAndRead/index.html`及其关联的`index.js`就运行在渲染进程中。

    ```html
    <!-- src/feature/WriteAndRead/index.html -->
    <script>
      // 渲染进程可以直接访问DOM
      const btn = document.getElementById('btn');
      btn.onclick = () => { /* ... */ };
    </script>
    ```

## Node.js集成：打破浏览器沙箱

Electron最强大的特性之一，就是在渲染进程中无缝集成了Node.js。这意味着，你可以在UI代码中直接调用Node.js的API，例如文件系统（`fs`）、路径处理（`path`）等，而无需搭建复杂的后端服务。

在示例中，`src/feature/WriteAndRead/index.js`就直接使用了Node.js的`fs`模块来读写本地文件：

```javascript
// src/feature/WriteAndRead/index.js
const fs = require('fs');
const path = require('path');

function readFile() {
  fs.readFile(path.join(__dirname, '/simple.txt'), (err, data) => {
    // ...
  });
}
```

**安全提示**：在渲染进程中直接使用Node.js能力虽然方便，但也带来了安全风险。如果你的应用需要加载和执行远程网页内容，请务必开启`nodeIntegration: false`和`contextIsolation: true`，并通过`preload`脚本来安全地暴露特定API给渲染进程，以防止恶意代码访问系统资源。

## 进程间通信（IPC）

虽然主进程和渲染进程各司其职，但它们之间经常需要通信。Electron提供了多种进程间通信（IPC）机制：

1.  **`ipcMain` 和 `ipcRenderer`**：这是最常用的一对模块，用于在主进程和渲染进程之间发送和接收异步或同步消息。

2.  **`remote`模块（已废弃）**：`remote`模块提供了一种便捷的方式，可以直接在渲染进程中调用主进程的模块和方法，如同本地对象一样。在项目中就使用了`remote`模块来创建新窗口：

    ```javascript
    // src/feature/WriteAndRead/index.html
    const { BrowserWindow } = require('electron').remote;
    const newWin = new BrowserWindow({ /* ... */ });
    ```

    **注意**：由于性能和安全原因，`remote`模块在Electron 12之后已被废弃，并计划在未来版本中移除。官方推荐使用`ipcMain`/`ipcRenderer`配合`contextBridge`作为替代方案。

## 调试

```bash
# 开发热更新
$ nodemon --watch . --exec electron .
```

## 应用打包与分发

开发完成后，我们需要将应用打包成可执行文件，以便分发给用户。`electron-packager`是一个流行的打包工具，可以轻松地将Electron应用打包成Windows、macOS和Linux平台的可执行程序。

- 应用程序的绿色版本：无需安装，拷贝整个文件目录之后即可使用
- 客户端应用程序安装包：安装之后通过桌面快捷方式的形式去访问（Inno Setup）

```bash
# 安装electron打包工具electron-packager
$ npm install electron-packager -g
# 编译绿色安装包
$ electron-packager . appName --win --out ./appName --arch=x64 --app-version=1.0.0 --electron-version=1.0.0 --overwrite  --icon=./src/assets/favicon.ico --ignore=node_modules
# 客户端应用程序安装包
Inno Setup 通过软件向导生成 setup.exe
```

[electron 安装+运行+打包成桌面应用+打包成安装文件+开机自启动](https://www.cnblogs.com/kakayang/p/9559777.html)

### electron-packager

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

## 总结与展望

Electron通过巧妙地结合Web技术和原生能力，为开发者打开了构建桌面应用的新大门。理解其主进程与渲染进程的分离架构，善用Node.js集成带来的便利，并掌握进程间通信的技巧，是开发高质量Electron应用的关键。

随着技术的发展，我们应该持续关注Electron的最佳实践，例如使用`contextBridge`提升应用安全性，以及探索更现代化的打包工具（如`electron-builder`）来优化分发流程。希望本文能为你探索Electron世界提供一个坚实的起点。
