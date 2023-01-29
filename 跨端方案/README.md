[[译] React Native vs. Cordova、PhoneGap、Ionic，等等](https://juejin.cn/post/6844903684032167944)

### 原生的定义

软件是关于如何操作大量晶体管和电路 (两者统称为硬件) 的指令的集合。直接运行在硬件上的原始指令对我们人类来说是几乎无法理解的, 特别是考虑到当今计算机的复杂性和规模。

要使得软件可以理解和操作的话，计算机科学家将其划分为多个层，这些层均是由框架构成的，每个框架都运行在另一个框架之上。在所有框架中，越接近硬件的框架，我们就说它更“原生”。

严格来说，我们无法说一个应用本身是否是原生的。我们只能说，相比于另一个应用，它是更原生的。

### 框架

在 React Native 出现之前，移动端框架一般分为两个阵营。

- 首先是原生阵营，例如安卓的 Java/Kotlin 和 IOS 的 Objective-C/Swift 。

- 另外一个阵营就是以 Cordova/PhoneGap 和 Ionic 为代表的 WebView 框架。WebView 框架是在原生框架之上构建的，类似于 web 浏览器，可以同时运行在安卓和 IOS 平台上(还可以有更多平台)。这些框架可以让 Web 开发人员使用他们已经具备的 HTML、CSS 和 JavaScript 技能来开发应用。但是，相比于原生应用，这类应用会没有那么流畅，能访问的硬件功能也有限。

- React Native 代表的是移动端框架的第三阵营。React Native 直接运行在原生框架里，它的 UI 层要比 WebView 框架更原生，而其余部分处于模拟层，以实现其易用性。

### WebView 框架

基于 HTML5 的跨平台技术比较出名的有 PhoneGap、Cordova，常常用于开发 webapp，还有 Egret、Cocos-creator、Unity、weex、uniapp 等，常用于开发游戏；还有基于 node 的 nw.js，用于开发桌面应用，以及 Electron，一款比 nw.js 还强大的用网页技术来开发桌面应用的神器。
