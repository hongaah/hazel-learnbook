# h 函数

## 概述

Vue 提供了一个 `h()` 函数用于创建虚拟节点（vnodes）。`h()` 是 hyperscript 的简称，意思是"能生成 HTML (超文本标记语言) 的 JavaScript"。这个名字来源于许多虚拟 DOM 实现默认形成的约定。虽然一个更准确的名称应该是 `createVNode()`，但在需要频繁使用渲染函数时，简短的名字更具优势。

## 核心特点

1. **高度灵活性**：`h()` 函数可以适应各种组件和元素创建需求
2. **多种参数格式**：
   - 支持对象形式编写
   - 属性可以写成驼峰式或 kebab-case 格式（使用引号包裹）
   - 事件监听器可以写成 `onClick` 形式或对象形式

## 详细用法示例

```js
h('div', null, [
  h(
    'div',
    {
      // 样式对象（驼峰式）
      style: {
        marginTop: '10px',
        backgroundColor: '#f5f5f5'
      },
      // DOM 属性
      props: {
        value: someValue,
        id: 'container'
      },
      // 事件监听器
      on: {
        input: (value) => {
          this.value = value
        },
        change: (value) => {
          handleChange(value)
        },
        // 也可以使用 kebab-case 格式
        'custom-event': handleCustomEvent
      },
      // 其他特殊属性
      attrs: {
        'data-test': 'test-id'
      },
      // 类名
      class: {
        'active': isActive,
        'disabled': isDisabled
      }
    },
    {
      // 插槽内容必须使用函数形式
      default: () => [
        '审核建议：',
        h(ElInput, {
          // 双向绑定
          modelValue: var1.value,
          'onUpdate:modelValue': (val: string) => {
            var1.value = val
          },
          // 其他属性
          placeholder: '请输入',
          type: 'textarea',
          minRows: 4,
          maxlength: 1000,
          showWordLimit: true,
          // 原生事件
          onClick: handleClick,
          // 自定义指令
          directives: [
            {
              name: 'focus',
              value: true
            }
          ]
        })
      ]
    }
  )
])
```

## 插槽处理最佳实践

### 常见错误："Non-function value encountered for default slot."

1. **性能优化**：Vue 3 推荐使用函数式插槽以获得更好的性能
2. **显式声明**：即使只是默认插槽，也需要显式写出 `default: () => ...` 的形式
3. **多插槽处理**：当组件有多个插槽时，需要为每个插槽提供函数

```js
// 错误写法 - 直接传递内容
return h(Component, { props }, content)

// 正确写法 - 默认插槽
return h(Component, { props }, { 
  default: () => content 
})

// 多个插槽
return h(Component, { props }, {
  default: () => [content1, content2],
  header: () => h('div', 'Header'),
  footer: () => h('div', 'Footer')
})
```

## 进阶技巧

1. **动态组件**：
   ```js
   h(resolveComponent(componentName), { ...props })
   ```

2. **递归组件**：
   ```js
   h('div', { key: node.id }, [
     node.children ? node.children.map(child => h(recursiveComponent, { node: child })) : null
   ])
   ```

3. **JSX 替代**：
   ```jsx
   const vnode = <div id="container">Hello</div>
   ```

4. **指令使用**：
   ```js
   h('div', {
     directives: [
       {
         name: 'my-directive',
         value: someValue,
         arg: 'arg',
         modifiers: { modifier: true }
       }
     ]
   })
   ```

## 性能优化

1. **避免不必要的重新渲染**：为列表项提供稳定的 `key`
2. **合理使用函数式组件**：对于无状态的展示组件
3. **缓存静态内容**：将不变化的 vnodes 缓存起来
4. **合理使用片段**：减少不必要的包装元素

## 版本差异说明

1. **Vue 2 vs Vue 3**：
   - Vue 2 使用 `createElement` 而非 `h`
   - Vue 3 的插槽必须使用函数形式
   - Vue 3 对属性处理更加严格

2. **Composition API**：
   ```js
   import { h } from 'vue'
   
   export default {
     setup() {
       return () => h('div', 'Hello')
     }
   }
   ```
