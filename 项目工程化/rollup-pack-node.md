# rollup 打包 node commonjs 项目

[使用 rollup 打包启动项目](https://pengfeixc.com/blogs/javascript/rollup-with-typescript)

```js :rollup.config.mjs
// pnpm add -D @rollup/plugin-commonjs @rollup/plugin-node-resolve

import resolve from '@rollup/plugin-node-resolve'
import commonjs from '@rollup/plugin-commonjs'

export default [
  {
    plugins: [resolve(), commonjs()],
    input: './packages/Json2Interface/index.js',
    output: {
      file: './dist/index.js',
      format: 'cjs',
    },
    external: ['chalk', 'commander', 'inquirer', 'ora', 'request', 'fs', 'quicktype-core'],
  },
]
```

```json :package.json
{
  "scripts": {
    "rollup:Json2Interface": "rollup -c rollup.config.mjs"
  }
}
```

```js
// 使用
const { output } = require('../dist/index.js')
```
