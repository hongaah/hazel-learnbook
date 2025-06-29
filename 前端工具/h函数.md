# h 函数

h() 函数的使用方式非常的灵活

## 概念

Vue 提供了一个 h() 函数用于创建 vnodes，h() 是 hyperscript 的简称——意思是“能生成 HTML (超文本标记语言) 的 JavaScript”。这个名字来源于许多虚拟 DOM 实现默认形成的约定。一个更准确的名称应该是 createVNode()，但当你需要多次使用渲染函数时，一个简短的名字会更省力。

## 特点

- 编写时为对象形式
- 属性为驼峰格式或 ['xx-xx'] 格式
- 方法名变化 onClick 或对象形式

```js
h('div', null, [
  h(
    'div',
    {
      style: {
        marginTop: '10px'
      },
      props: {
        value: xx
      },
      on: {
        input: (value) => {
          this.value = xx
        },
        change: (value) => {
          xx
        }
      }
    },
    {
      default: () => [
        '审核建议：',
        h(ElInput, {
          modelValue: var1.value,
          'onUpdate:modelValue': (val: string) => {
            var1.value = val
          },
          placeholder: '请输入',
          type: 'textarea',
          minRows: 4,
          maxlength: 1000,
          showWordLimit: true
        })
      ]
    }
  )
])
```

## "Non-function value encountered for default slot."

1. Vue3 使用 h 函数推荐使用函数式插槽，以便获得更佳的性能
2. 如果组件定义了插槽，需要定义一个匿名函数返回 h 函数，即使是默认也要显性写出来 default。

```js
// 错误
return h(xxx, { xxx }, { xxx })
// 正确
return h(xxx, { xxx }, { default: () => xxx })
// 多个插槽
return h( xxx, { xxx }, { default: () => [xxx, xxx], })
```
