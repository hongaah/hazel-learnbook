# monorepo 方案

- pnpm workspaces
- yarn workspaces
- lerna + yarn workspaces

## pnpm workspaces

🌰：
- hazel-plus(ui)
- 两个官网重构合并（nuxt2.17）

### 两个官网重构合并（nuxt2.17）

pnpm 配置，使用旧 npm 安装依赖的方式

```json :.npmrc
shamefully-hoist=true
auto-install-peers=true
strict-peer-dependencies=false
```

项目中的别名 alias 重新定义引用路径

```js
import { resolve } from 'path'

const config = {
  alias: {
    '@': resolve(__dirname, '../..'),
  },
}
```

配置 ts 类型解析路径

```json :tsconfig.json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "baseUrl": "../../",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

### pnpm workspaces 项目部署

经常会有异常报错，如本地编译构建正常，发线上部署则报错，坑点可能是依赖问题，lock 文件没有锁好，某个依赖被升级了然后导致问题

[workspaces](https://pnpm.io/zh/workspaces)
[Quick Start：用 pnpm 管理 Monorepo 项目](https://zhuanlan.zhihu.com/p/422740629)

#### link 机制

在 monorepo 中，我们往往需要解决 package 间的引用，比如 @panda/tools 会被 @panda/server 和 @panda/web 依赖。

```json
"dependencies": {
  "@panda/tools": "workspace:^1.0.0", // 通过 workspace 为本地引用
},
```

通过软链接引用了 tools package，可以在 web 项目下的 node_modules 看到对应的代码已经提升了。

#### 发布 workspace 包

当这样的工具包被发布到平台后，如何识别其中的 workspace 呢？

需要 workspace 包打包到归档，可以通过`pnpm pack` 或 `pnpm publish` 之类的发布命令，将动态替换这些 workspace 依赖，把基于的 workspace 的依赖变成外部依赖，解决了开发环境和生产环境对依赖的问题。

注意：必须是用 pnpm 的命令才能起效

## yarn workspaces

package.json 配置

```json
{
  "private": true,
  "license": "MIT",
  "version": "1.0.0",
  "workspaces": ["apps/*"],
  "scripts": {
    "xxx:dev": "yarn workspace xxx run dev"
  }
}
```

## lerna + yarn workspaces

https://blog.csdn.net/frontend_frank/article/details/115344129

```bash
# 全局安装，lerna 测试版本为 5.6.2
$ npm install -g lerna@5.6.2

# 空项目初始化 lerna 项目
$ lerna init

# 执行 apps 项目下 xxx-a 项目的 script 指令 dev，如果不加 --stream，运行结果只会提示最后一句 run dev 就什么信息都不会报
$ lerna run dev --scope xxx-a --stream

# 查看 lerna 指令
$ lerna run --help
```

如果在旧项目重构成 lerna monorepo 项目则手动加 lerna.json 文件

```json :lerna.json
{
  "$schema": "node_modules/lerna/schemas/lerna-schema.json",
  "version": "independent", // lerna默认使用的是集中版本，所有的package共用一个version,如果需要packages下不同的模块 使用不同的版本号，需要配置Independent模式
  "packages": ["apps/*"],
  "useWorkspaces": true,
  "npmClient": "yarn",
  "useNx": false
}
```

## turborepo

https://turbo.build/
