# Module批量导入

## 正则表达式

+ `\w`：匹配字母、数字、下划线。等价于 `[A-Za-z0-9_]`
+ `.`：匹配除换行符 \n 之外的任何单字符。要匹配 . ，请使用 \\.
+ `*`：匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \\*

+	`+`：匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \\+

## require.context

你还可以通过 `require.context()` 函数来创建自己的 context。

可以给这个函数传入三个参数：一个要搜索的目录，一个标记表示是否还搜索其子目录， 以及一个匹配文件的正则表达式。

webpack 会在构建中解析代码中的 `require.context()` 。

**语法如下**：

```javascript
require.context(
  directory,
  (useSubdirectories = true),
  (regExp = /^\.\/.*$/),
  (mode = 'sync')
);
```

示例：

```javascript
require.context('./test', false, /\.test\.js$/);
//（创建出）一个 context，其中文件来自 test 目录，request 以 `.test.js` 结尾。
require.context('../', true, /\.stories\.js$/);
// （创建出）一个 context，其中所有文件都来自父文件夹及其所有子级文件夹，request 以 `.stories.js` 结尾。
```

**Warning**：传递给 `require.context` 的参数必须是字面量

**返回值**

一个 context module 会导出一个（require）函数，此函数可以接收一个参数：request。

此导出函数有三个属性：`resolve`, `keys`, `id`。

- `resolve` 是一个函数，它返回 request 被解析后得到的模块 id。
- `keys` 也是一个函数，它返回一个数组，由所有可能被此 context module 处理的请求（译者注：参考下面第二段代码中的 key）组成。

如果想引入一个文件夹下面的所有文件，或者引入能匹配一个正则表达式的所有文件，这个功能就会很有帮助，例如：

```javascript
function importAll(r) {
  r.keys().forEach(r);
}

importAll(require.context('../components/', true, /\.js$/));
```


```javascript
const cache = {};
function importAll(r) {
  r.keys().forEach((key) => (cache[key] = r(key)));
}

importAll(require.context('../components/', true, /\.js$/));
// 在构建时(build-time)，所有被 require 的模块都会被填充到 cache 对象中。
```

- `id` 是 context module 的模块 id. 它可能在你使用 `module.hot.accept` 时会用到。

## 练习

### 批量导入 store.js

```js
import Vue from 'vue'
import Vuex from 'vuex'
import getters from './getters'

Vue.use(Vuex)

// https://webpack.js.org/guides/dependency-management/#requirecontext
const modulesFiles = require.context('./modules', true, /\.js$/)

// you do not need `import app from './modules/app'`
// it will auto require all vuex module from modules file
const modules = modulesFiles.keys().reduce((modules, modulePath) => {
  // set './app.js' => 'app'
  const moduleName = modulePath.replace(/^\.\/(.*)\.\w+$/, '$1')
  const value = modulesFiles(modulePath)
  modules[moduleName] = value.default
  return modules
}, {})

const store = new Vuex.Store({
  modules,
  getters
})

export default store
```

### 批量导入图标

```js
const req = require.context('../../assets/icons/svg', false, /\.svg$/)
const requireAll = requireContext => requireContext.keys()

const re = /\.\/(.*)\.svg/

const icons = requireAll(req).map(i => {
  return i.match(re)[1]
})

export default icons
```

