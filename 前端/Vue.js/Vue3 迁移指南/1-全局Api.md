# 全局Api应用实例

## 存在的问题

之前我们定义的应用只是通过 `new Vue()` 创建的根 Vue 实例。从同一个 Vue 构造函数创建的每个根实例**共享相同的全局配置**。

因此我们定义的东西，如下：

```js
Vue.component('button-counter', {
  data: () => ({
    count: 0
  }),
  template: '<button @click="count++">Clicked {{ count }} times.</button>'
})
```

类似地，全局指令的声明方式如下：

```js
Vue.directive('focus', {
  inserted: (el) => el.focus()
})
```

它会影响到所有的实例，缺点如下：

+ 在测试期间，全局配置很容易意外地污染其他测试用例。
+ 全局配置使得在同一页面上的多个“应用”在全局配置不同时共享同一个 Vue 副本非常困难。

## 新的全局Api：`createApp`

### 1. 总览

调用 `createApp` 返回一个*应用实例*，一个 Vue 3 中的新概念。通过 `createApp` 创建的对象数据会相互隔离。

```js
import { createApp } from 'vue'

const app = createApp({})
```

应用实例暴露了 Vue 2 全局 API 的一个子集，经验法则是，*任何全局改变 Vue 行为的 API 现在都会移动到应用实例上*，以下是 Vue2 全局 API 及其相应的实例 API 列表：

| 2.x 全局 API               | 3.x 实例 API (`app`)                       |
| -------------------------- | ------------------------------------------ |
| Vue.config                 | app.config                                 |
| Vue.config.productionTip   | *移除*                                     |
| Vue.config.ignoredElements | app.config.compilerOptions.isCustomElement |
| Vue.component              | app.component                              |
| Vue.directive              | app.directive                              |
| Vue.mixin                  | app.mixin                                  |
| Vue.use                    | app.use                                    |
| Vue.prototype              | app.config.globalProperties                |
| Vue.extend                 | *移除*                                     |

### 2. `config.productionTip` 移除

在 Vue 3.x 中，“使用生产版本”提示仅在使用“dev + full build”(包含运行时编译器并有警告的构建版本) 时才会显示。

对于 ES 模块构建版本，由于它们是与打包器一起使用的，而且在大多数情况下，CLI 或脚手架已经正确地配置了生产环境，所以本提示将不再出现。

### 3. `config.ignoredElements` 替换为 `config.isCustomElement`

引入此配置选项的目的是为了支持原生自定义元素，因此重命名可以更好地传达它的意图。同时，新选项接受一个函数，相比旧的字符串或正则表达式来说能提供更高的灵活性：

```js
// 之前
Vue.config.ignoredElements = ['my-el', /^ion-/]

// 之后
const app = createApp({})
app.config.compilerOptions.isCustomElement = (tag) => tag.startsWith('ion-')
```

### 4. `Vue.prototype` 替换为 `config.globalProperties`

在 Vue 2 中， `Vue.prototype` 通常用于添加所有组件都能访问的 property。

在 Vue 3 中与之对应的是 [`config.globalProperties`](https://cn.vuejs.org/api/application-config.html#globalproperties)。这些 property 将被复制到应用中，作为实例化组件的一部分。

```js
// 之前 - Vue 2
Vue.prototype.$http = () => {}
```

```js
// 之后 - Vue 3
const app = createApp({})
app.config.globalProperties.$http = () => {}
```

使用 `provide` 时，也应考虑作为 `globalProperties` 的替代品。

### 5. `Vue.extend` 移除

在 Vue 2.x 中，`Vue.extend` 曾经被用于创建一个基于 Vue 构造函数的“子类”，其参数应为一个包含组件选项的对象。在 Vue 3.x 中，我们已经没有组件构造器的概念了。应该始终使用 `createApp` 这个全局 API 来挂载组件：

```js
// 之前 - Vue 2

// 创建构造器
const Profile = Vue.extend({
  template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
  data() {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
})
// 创建一个 Profile 的实例，并将它挂载到一个元素上
new Profile().$mount('#mount-point')
```

```js
// 之后 - Vue 3
const Profile = {
  template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
  data() {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
}
Vue.createApp(Profile).mount('#mount-point')
```

### 6. 插件

在 UMD 构建中，插件开发者使用 `Vue.use` 来自动安装插件是一个通用的做法。例如，官方的 `vue-router` 插件是这样在浏览器环境中自行安装的：

```js
var inBrowser = typeof window !== 'undefined'
/* … */
if (inBrowser && window.Vue) {
  window.Vue.use(VueRouter)
}
```

由于 `use` 全局 API 在 Vue 3 中已无法使用，因此此方法将无法正常工作，并且调用 `Vue.use()` 现在将触发一个警告。取而代之的是，开发者必须在应用实例上显式指定使用此插件：

```js
const app = createApp(MyApp)
app.use(VueRouter)
```

## 挂载App实例

使用 `createApp(/* options */)` 初始化后，应用实例 `app` 可通过 `app.mount(domTarget)` 挂载根组件实例：

```js
import { createApp } from 'vue'
import MyApp from './MyApp.vue'

const app = createApp(MyApp)
app.mount('#app')
```

## Provide / Inject（依赖注入）

与在 2.x 根实例中使用 `provide` 选项类似，Vue 3 应用实例也提供了可被应用内任意组件注入的依赖项：

```js
// 在入口中
app.provide('guide', 'Vue 3 Guide')

// 在子组件中
export default {
  inject: {
    book: {
      from: 'guide'
    }
  },
  template: `<div>{{ book }}</div>`
}
```

在编写插件时使用 `provide` 将尤其有用，可以替代 `globalProperties`。

## 在应用之间共享配置

在应用之间共享配置 (如组件或指令) 的一种方法是创建工厂函数，如下所示：

```js
import { createApp } from 'vue'
import Foo from './Foo.vue'
import Bar from './Bar.vue'

const createMyApp = (options) => {
  const app = createApp(options)
  app.directive('focus' /* ... */)

  return app
}

createMyApp(Foo).mount('#foo')
createMyApp(Bar).mount('#bar')
```

现在，`Foo` 和 `Bar` 实例及其后代中都可以使用 `focus` 指令。