---
title: "pinia使用手册"
date: 2023-01-14T09:51:32+08:00
draft: false
hiddenFromHomePage: false
categories: ["note"]
tags: ["pinia"]
---
### 前言
#### Store 是什么
Store是一个保存状态和业务逻辑的实体，它并不与你的组件树绑定，它承载着全局的状态

#### 什么时候使用Store
一个Store应该包含可以在整个应用中访问的数据，例如显示在导航栏中的用户信息，以及需要通过页面保存的数据，例如一个非常复杂的多步骤表单。应该避免在Store中引入那些本可以在组件中保存的本地数据，例如，一个元素在页面中的可见性。


### 使用

1. 创建pinia实例(根store)，然后将其挂载到app上
```javascript
// mian.ts
import { createPinia } from 'pinia'
const pinia = createPinia()
app.use(pinia)
```

2. 声明store
```typescript
// useTodoList.ts
import { defineStore } from 'pinia'
export const useTodoList = defineStore('todoList', {
  // state
  // getters
  // actions
})
```

3. 使用store
```typescript
import { useTodoList } from '@/store/useTodoList'
const todoListStore = useTodoList()
```

4. 修改state
+ actions中修改
+ 直接修改
+ $patch
+ $reset

5. 订阅state
+ $subscribe
```typescript
store.$subscribe((mutation, state) => {

})
```

+ watch 监听

5. 插件的使用
```typescript
import { createPinia } from 'pinia'

// 创建的每个 store 中都会添加一个名为 `secret` 的属性。
// 在安装此插件后，插件可以保存在不同的文件中
function SecretPiniaPlugin() {
  return { secret: 'the cake is a lie' }
}

const pinia = createPinia()
// 将该插件交给 Pinia
pinia.use(SecretPiniaPlugin)

// 在另一个文件中
const store = useStore()
store.secret // 'the cake is a lie'
```

插件中的参数
```typescript
export function myPiniaPlugin(context) {
  context.pinia // 用 `createPinia()` 创建的 pinia。 
  context.app // 用 `createApp()` 创建的当前应用(仅 Vue 3)。
  context.store // 该插件想扩展的 store
  context.options // 定义传给 `defineStore()` 的 store 的可选对象。
  // ...
}
```