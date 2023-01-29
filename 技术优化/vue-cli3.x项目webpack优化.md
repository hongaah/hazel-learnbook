# vue-cli3.x 项目 webpack 优化

### terser-webpack-plugin

> 该插件可以压缩 JavaScrip 和在构建时一键去掉调试打印语句

- 安装：

```bash
$ npm install terser-webpack-plugin@3.1.0 --save-dev
```

- 配置：

```js: vue.config.js
const TerserPlugin = require('terser-webpack-plugin')

configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      return {
        optimization: {
          minimize: true,
          minimizer: [
            new TerserPlugin({
              cache: true,
              parallel: true,
              sourceMap: false,
              terserOptions: {
                compress: {
                  drop_console: true,
                  drop_debugger: true,
                  // pure_funcs: ['console.log'], // 移除console
                },
              },
            }),
          ],
        },
    }
}
```

参考链接：
<https://v4.webpack.docschina.org/configuration/optimization/#optimization-minimizer>
<https://github.com/webpack-contrib/terser-webpack-plugin>
<https://github.com/terser/terser#minify-options>

### compression-webpack-plugin

> 压缩静态资源，服务端在 Nginx 开启 Gzip 属性。Nginx 在访问资源时，如果该资源有 gz 文件，则会请求 gz 文件。

- 安装：

```bash
$ npm install compression-webpack-plugin --save-dev
```

- 配置：

```js: vue.config.js
const CompressionPlugin = require('compression-webpack-plugin') // 引入gzip

configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      return {
        plugins: [
          // 添加gzip
          new CompressionPlugin({
            test: /\.js$|\.html$|\.css/,
            threshold: 10240,
            deleteOriginalAssets: false
          }),
        ]
    }
}
```

参考：
<https://v4.webpack.js.org/plugins/compression-webpack-plugin/>

### 分析打包速度

**speed-measure-webpack-plugin**

https://github.com/stephencookdev/speed-measure-webpack-plugin

只针对 plugins
