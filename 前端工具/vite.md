# vite

Vite，一个基于浏览器原生 ES imports 的开发服务器。利用浏览器去解析 imports，在服务器端按需编译返回，完全跳过了打包这个概念，服务器随起随用。同时不仅有 Vue 文件支持，还搞定了热更新，而且热更新的速度不会随着模块增多而变慢。针对生产环境则可以把同一份代码用 rollup 打。

## 构建

### 构建产物分析

产物分析

```json :package.json
"scripts": {
  "dev": "vite",
  "build": "tsc && vite build -w"
}
```

**pnpm dev**

pnpm dev // 直接访问 http://127.0.0.1:5173/ 更新生效
入口 index 文件是：root/index.html（Vite App ROOT）
监听的是 root/index.html 文件，该文件变化就会变化，包括引用的文件源码；结果可能不准

**pnpm build**

pnpm build （先 build，会持续监听并 build
入口 index 文件是：root/public/index.html（Vite App Public）
直接 build 完然后 open with live server，访问：http://127.0.0.1:5500/dist/index.html，如果被监听文件更新了需要刷新页面
监听的是设置的文件夹，public 下的内容会复制过去

### 环境变量

vite build 默认运行生产模式构建，你也可以通过使用不同的模式和对应的 .env 文件配置来改变它，用以运行开发模式的构建。

模式配置：NODE_ENV=development
import.meta.env.DEV: 是否运行在开发环境(NODE_ENV=development)
import.meta.env.PROD: 是否运行在生产环境(NODE_ENV=production)

## rollup 代码分割 manualChunks

1. 在 vite 配置文件，通过 build.rollupOptions.output.manualChunks 配合手动分包策略之后，vite 会自动生成 vendor 包
2. 当页面越来越多，配置了动态引入页面之后，打包出来会产生 chunk 碎片，如几个页面公用的文件 api.js sdkUtils.js http.js 等，这些独立的分包大小都很小，加起来 gzip 之后都不到 1kb，增加了网络请求

```js
/**
 * @desc 分割第三方依赖包，解决 vendor 包过大，合并多处引用的文件，解决 chunk 碎片
 * @name configManualChunk
 * @author wxh
 * @description chunk 拆包优化；依赖和构建后的文件有问题
 */
const vendorLibs: string[] = ['element-plus', 'jspdf', 'lodash', 'axios', 'vue-router']

export const configManualChunk = (id: string, { getModuleInfo }: any) => {
  // 分割第三方依赖包，解决 vendor 包过大
  if (/[\\/]node_modules[\\/]/.test(id)) {
    const matchItem = vendorLibs.find(item => {
      const reg = new RegExp(`[\\/]node_modules[\\/]_?(${item})(.*)`, 'ig')
      return reg.test(id)
    })
    return matchItem ? matchItem : 'vendor'
  }
  if (
    // 合并多处引用的文件，解决 chunk 碎片
    getModuleInfo(id).importers.length + getModuleInfo(id).dynamicImporters.length >
    2
  ) {
    return 'manifest'
  }
}
```

[解决 chunk 碎片问题](https://segmentfault.com/a/1190000041919468)
[Vite - 代码分割策略](https://juejin.cn/post/7135671174893142030#heading-0)

### 重写分包策略后报错

vendor 模块中会导入 manifest 中导出的 jsx-runtime 依赖，而 manifest 中这个依赖又是从 vendor 中导入的。

## 开发环境页面加载问题慢

解决：升级 vite3 到 4 可解决

[Vite 的首屏性能为什么不好](https://cloud.tencent.com/developer/article/2224646)
[vite 性能优化 — 增加业务代码预构建，加快首屏输出](https://juejin.cn/post/7085424254702845960#heading-4)

升级 vite 3 到 4 步骤：

1. vite 安装 4 后，执行 pnpm i 查看相对应的 peer 依赖应该是多少。
2. 联动 peer 依赖更新，比如：vitejs/plugin-vue vitejs/plugin-vue-jsx unplugin-icons unplugins-element-plus
3. 测试页面

出现的问题：
element-ui 下拉框组件出现异常，与 ui 版本有关。
element-plus el-select updateOptions emit 报错：`[Vue warn]: Extraneous non-emits event listeners (updatedcount) were passed to component but could not be automatically inherited because component renders fragment or text root nodes. If the listener is intended to be a component custom event listener only, declare it using the "emits" option`

目前解决：将 element-plus 降级到没有 updateOptions emit 即 2.2.32 恢复正常。还原相同环境测试 vue 相关的 emit 无问题，应该是 element-plus 问题，具体问题待确定。

https://cn.vitejs.dev/guide/migration.html
https://github.com/element-plus/element-plus/commits/dev/packages/components/select/src/options.ts
https://github.com/element-plus/element-plus/pull/11868
