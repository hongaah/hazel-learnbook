# node 内存溢出问题

Node 进程的内存限制是有限的，当前在默认情况下，V8 在 32 位系统上的内存限制为 512mb，在 64 位系统上的内存限制为 1gb，所以有些项目在加了 Babel 后在构建时可能会出现内存溢出的问题。可以通过将 --max-old-space-size 的值提高，但是如果达到内存限制，还可以将单个进程拆分为多个工作进程。

### 解决方法

#### windows

方式一：VSCode 工作区设置

```js: .vscode/settings.json

{
  "terminal.integrated.env.windows": {
    "node_options": "--max_old_space_size=3072"
  }
}

```

方式二：在 package.json 中设置

```js: package.json

"scripts": {
    "serve": "vue-cli-service serve --max_old_space_size=3072",
}

```

方式三：在终端进程中设置

```bash
set NODE_OPTIONS=--max-old-space-size=<size in bits> // 单开终端
export NODE_OPTIONS="--max-old-space-size=8192" // 永久终端
```

#### macos

macos 在.bash_profile 下加入

```sh
export NODE_OPTIONS=--max_old_space_size=3072
```

<https://www.cnblogs.com/caofeng11/p/13160416.html>

#### windows 和 macos

**通用的方式，使用 cross-env 跨平台设置环境变量**

```json
npm install -D cross-env

"scripts": {
    "serve": "cross-env NODE_OPTIONS=--max_old_space_size=3072 vue-cli-service serve",
  },
```

<https://blog.csdn.net/wuyujin1997/article/details/122869951>

更多可参考：

[详细配置](https://qa.icopy.site/questions/56982005/where-do-i-set-node-options-max-old-space-size-2048)

[nodejs 设置应用的内存上限 NODE_OPTIONS=--max-old-space-size=8192](https://blog.csdn.net/wuyujin1997/article/details/122869951)

[node 内存溢出了？ 看看这篇。](https://juejin.cn/post/6965382058973593636)

### 其他

#### Node.js heap out of memory

1. 升级 node 版本
2. 运行以下命令

```bash
npm install -g increase-memory-limit
increase-memory-limit
```

<https://stackoverflow.com/questions/38558989/node-js-heap-out-of-memory>
