# uniapp

## 常用

### uniapp 生命周期执行顺序

```js
// 进入应用
App Launch
App Show
page onLoad
page onShow
component beforeCreate
component created
component mounted
page onReady

// 应用后台
App Hide
page onHide

// 应用关闭
page onUnload
component destory

// 后台重新进入
App Show
page onLoad
page onShow
```

带来的问题：

1. 每次切换页面的时候，组件的生命周期 created mounted，只能触发一次，解决方法：ref，在父组件的 onShow 调用子组件的方法

2. 生命周期：
   如果页面没有销毁但隐藏会触发 onHide，销毁也隐藏了触发 beforeDestroyed，onHide 和 beforeDestroyed 二者只会触发一次

3. onLoad 和 onShow

   navigateTo 调转，下一页会触发 onload 和 onshow，返回会触发 beforeDestroy

   小程序缩小到后台，切换 app 再返回小程序会触发 onhide onshow

   总结：除想要每次显示小程序都处理某些逻辑要使用 onShow，否则最好用 onload

### uniapp 跨组件触发事件

uni.$emit('xxx')
....

## 小程序分包

小程序主包最多只能 2M 的限制，当一个主包文件过多时，肯定面临着性能低，卡顿的情况。所以采用了分包的加载机制，将各个模块解耦。总包 20M

### 降低主包大小

- 外部根目录的 components 优化，只放使用率高的组件
- 尽量多分包，分包数量可支持 10 个左右，总包大小 20M
- 图片除底部栏外尽量使用网络图片
- 升级微信开发者工具，可提高打包效率压缩代码

### 打包原则

- app（主包）也可以有自己的 pages（即最外层的 pages 字段）
- tabBar 页面必须在 app（主包）内
- subpackages 配置路径外的目录将被打包到 app（主包） 中
- 声明 subpackages 后，将按 subpackages 配置路径进行打包
- subpackage 的根目录不能是另外一个 subpackages 内的子目录

### app.json 目录结构

微信原生是在同级分包，但 mpvue 同级会报错的，只能在 pages 中进行分包，在 subpackages 声明项目分包结构

```json: app.json
{
  "pages": ["pages/home/main", "pages/tabbarpage.main"],
  "subPackages": [
    {
      "root": "pages/subPage", //分包根目录
      "name": "", // 分包别名，分包预下载时可以使用
      "pages": [
        // 分包页面路径，相对于分包根目录
        "page1/main",
        "page2/main"
      ],
      "independent": "" // 分包是否是独立分包
    }
  ]
}
```

## uniapp 小程序微信和支付宝端差异

### class 和 style 语法

:class="xx: Boolean"

注意事项：

1. Boolean 不能是要通过复杂计算出的布尔，也不能是要经过默认转化的非布尔值
2. class 为多个值时可能会异常，判断不正确

### 支付宝兼容问题

1. 组件传值为数组对象时，子组件修改值后支付宝界面未同步更新

   子组件通过直接修改数组对象内容，数据都可以修改，微信可以渲染界面，但支付宝不行

   可以将值通过 computed 复制一份，在该备份改完以后，forceupdate

2. span 标签注册 click 事件失败，需要转为 div

3. 样式空白屏

   当使用 position: absolute;top: 0;bottom: 0; 撑开元素 A 时，需要注意 A 的父元素是否为根元素 body，如果不是需要给一个高度，否则会出现子是撑不开父的。

   微信端容错性高，可以展示效果，支付宝不行。

4. 样式边距错乱

- 支付宝端有时会对 box-sizing: border-box;不生效。如

```css
/* 此时swiper-box的宽是铺满屏幕的，没有边距 */
.main-content-banner {
  width: 100%;
  height: 98px;
  padding-bottom: 16px;
  position: relative;

  .swiper-box {
    width: 100%;
    height: 98px;
    box-sizing: border-box;
    padding: 0 12px;
    text-align: center;
  }
}
```

- 支付宝图片标签如果定义了 padding-bottom，会拉长图片的高，尽量用 margin
- 图片的背景如果用了 background-size: contain; 会使支付宝端高度不可控，与微信端不一致

5. vue 数据联动没反应

   原则上子组件不能对 props 传值进行修改，包括对象。但微信端可直接引用该 props 对象，再 emit update 修改。支付宝端不行。

   所以可采用 computed 复制 props 再引用，修改可用 emit update。
