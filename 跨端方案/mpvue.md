## mpvue 起步

1. 安装和注册微信小程序
   https://developers.weixin.qq.com/miniprogram/dev/framework/

2. mpvue 脚手架
   http://mpvue.com/mpvue/quickstart.html

#### mpvue 问题

1. 组件 onshow 获取父组件的传值会有误，会缓存并取上一个父组件传的值
2. 组件的 onshow 第一次进入页面不会触发，只有隐藏页面再打开才会触发。组件的 mounted 只有第一次进入页面才会触发
3. 组件传值，props 传的是对象 A，父组件对 A 做出修改，子组件获得 A 可以变化，但修改的 A 只能是**对象的第一层元素**。如果修改该对象的元素是数组，子组件不会同步变化该数组内的元素。
4. mpvue 传值其他方法
   - 必须是在 onLoad 钩子函数往后的生命周期中获取
   - 在所有 页面 的组件内可以通过 this.$root.$mp.query 进行获取小程序在 pageonLoad 时候传递的 options。要注意：写在 mounted 函数里，写到 created 报错。
   - 在所有的组件内可以通过 this.$root.$mp.appOptions 进行获取小程序在 apponLaunch/onShow 时候传递的 options。
   - a 标签也可以跳转，用 href 属性

### 其他框架

WePY：是在代码开发风格上借鉴了 Vue，本身和 Vue 没有什么关系，腾讯"类 Vue"的框架

<https://github.com/Tencent/wepy>

<https://wepyjs.github.io/wepy-docs/>
