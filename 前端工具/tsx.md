# tsx

SFC：Vue 单文件组件

TSX：TypeScript 文件，包含 JavaScript XML（JSX）

## 配置

```ts :vite.config.ts
// pnpm install @vitejs/plugin-vue-jsx -D

import { defineConfig } from "vite"
import vue from "@vitejs/plugin-vue"
import vueJsx from "@vitejs/plugin-vue-jsx"

export default defineConfig({
  plugins: [
    vue(),
    vueJsx() //插件使用
  ],
  css: {
    modules: {
      // 规定css类名的命名规则为小驼峰，即 child-item css 类在 js 中会变成 childItem 变量
      localsConvention: 'camelCase'
    }
  },
})
```

## CSS Modules

css module 把 css 作为模块引入到 js 中，是一种技术流的组织css代码的策略，它将为css提供默认的局部作用域。

css module 对 css 文件命名有要求，必须在后缀名前面是module，例如xxx.module.css、xxx.module.less、xxx.module.scss。

```js :vite.config.ts
css: {
  modules: {
    // 规定css类名的命名规则为小驼峰，即 child-item css 类在 js 中会变成 childItem 变量
    localsConvention: 'camelCase'
  }
},
```

```tsx
const TsxDemo = defineComponent({
  setup() {
    return () => {
      return (
        <>
          <div class={styles.tableWrap}></div>
          {/* OR */}
          <div class={styles['table-wrap']}></div>
        </>
      )
    }
  },
})

export default TsxDemo
```

### 编译和使用

css module 会在编译的时候自动把类名加上一个哈希字符串，而使用 global 声明的 class 不会在编译的时候被加上哈希字符串。

```scss :index.module.scss
/** 形式一 */
.demo-normal {
  .normal-test {
    background-color: red !important;
  }
}
// 形式一会编译成：
._react-demo_1e21d_6 ._demo-test_1e21d_6 {
  background-color: red !important;
}

/** 形式二 */
.demo-global {
  :global {
    .global-test {
      background-color: red !important;
    }
  }
}
// 形式二会编译成：
._react-demo_86lo7_6 .demo-test {
  background-color: red !important;
}
```

类名使用引用的方式会被编译成哈希字符串的，global 声明则不能用引用的方式。

```tsx
// 形式一
<div class={styles['demo-normal']}>
  <div class={styles['normal-test']}>
    normal
  </div>
</div>

// 形式二
<div class={styles['demo-global']}>
  <div class="global-test">
    global
  </div>
</div>
```

### global 全局作用选择器

```scss :statistics.module.scss
// react，如果我们想要覆盖第三方的组件样式，用自己写的选择器名称是覆盖不了的，因为加了哈希字符串之后与组件默认样式的选择器名称不匹配；而使用 global 声明的 class，不会在编译的时候被加上哈希字符串，从而可以实现覆盖默认样式的效果。
.table-wrap {
  :global {
    .el-table td.el-table__cell,
    .el-table thead.is-group th.el-table__cell {
      padding: 5px !important;
    }
  }
}

// 区别 vue，即使用一个唯一的类名包着也会作用到有用到该类名的全局样式，慎用
// 比如在下方例子中，其他文件用了 .test 类名也会被影响到
.notice-dialog {
  :global(.el-dialog.is-fullscreen) {
    min-width: 900px !important;
  }
  :global(.test) {
    background-color: red !important;
  }
}
```

## [classnames](https://www.npmjs.com/package/classnames)

classNames('foo', 'bar') // => 'foo bar'
