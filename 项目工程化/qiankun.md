# [qiankun](https://qiankun.umijs.org/)

- main：主应用，作为一个外壳，集合多个子应用
- sub：子应用，每个子应用都是独立的项目，独立配置且需要独立部署

🌰：https://github.com/umijs/qiankun/examples

```sh
# 当时测试用的 node 版本
nvm use 16.15.0
# 根目录
npm i
# 每个 examples 的应用
npm i

npm run examples:start
```

## qiankun 样式隔离

默认的 qiankun 案例项目是没开启 css 隔离，所有的 css 都在全局，这样各应用的样式会相互影响，比如主应用和子应用。

qiankun 提供了两种样式隔离方案：shadow dom 和自己实现的 scoped。

### shadow dom

shadow dom 是 web components 技术的一部分，其实就一个 attachShadow 的 api。web components 添加内容的时候，不直接 appendChild，而是先 attach 个 shadow，然后再在下面 appendChild。

开启 shadow dom 后，shadow dom 内的样式和外界互不影响。但由于弹窗默认是挂在 body 上的，也就不在 shadow dom 里了，那 shadow dom 里给它加的样式自然就不生效了，所以弹窗的样式就丢失了。

```js :multiple.js qiankun 开启 shadow dom
loadMicroApp(
  { name: 'vue', entry: '//localhost:7101', container: '#vue' },
  { sandbox: { strictStyleIsolation: true } },
);
```

### qiankun 实现的 scoped

scoped 的方案是给选择器加了一个 data-qiankun='应用名' 的选择器，这样父应用能设置子应用样式，这样能隔离样式，但是同样有挂在 body 的弹窗样式设置不上的问题，因为 qiankun 的 scoped 不支持全局样式

```js :multiple.js qiankun 开启 scoped
loadMicroApp(
  { name: 'vue', entry: '//localhost:7101', container: '#vue' },
  { sandbox: { experimentalStyleIsolation: true } },
);
```

### 解决方案

react 和 vue 项目本身都会用 scoped css 或者 css modules 的组件级别样式隔离方案，这俩方案都支持传递样式给子元素、设置全局样式等，只是实现和使用方式不同。
