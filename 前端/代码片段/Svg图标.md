# Svg图标

## 安装 loader

```shell
npm i svg-sprite-loader -D
或
yarn add svg-sprite-loader --dev
```

## 创建组件

在components文件夹创建名为SvgIcon的文件夹，并在其中新建index.vue，内容如下

```vue
<template>
  <div v-if="isExternal" :style="styleExternalIcon" class="svg-external-icon svg-icon" v-on="$listeners" />
  <svg v-else :class="svgClass" aria-hidden="true" v-on="$listeners">
    <use :xlink:href="iconName" />
  </svg>
</template>

<script>
import { isExternal } from '@/utils/index'

export default {
  name: 'SvgIcon',
  props: {
    iconClass: {
      type: String,
      required: true
    },
    className: {
      type: String,
      default: ''
    }
  },
  computed: {
    isExternal() {
      return isExternal(this.iconClass)
    },
    iconName() {
      return `#icon-${this.iconClass}`
    },
    svgClass() {
      if (this.className) {
        return 'svg-icon ' + this.className
      } else {
        return 'svg-icon'
      }
    },
    styleExternalIcon() {
      return {
        mask: `url(${this.iconClass}) no-repeat 50% 50%`,
        '-webkit-mask': `url(${this.iconClass}) no-repeat 50% 50%`
      }
    }
  }
}
</script>

<style scoped>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}

.svg-external-icon {
  background-color: currentColor;
  mask-size: cover!important;
  display: inline-block;
}
</style>
```

`isExternal`函数如下，在合适的地方添加

```js
export function isExternal(path) {
  return /^(https?:|mailto:|tel:)/.test(path)
}
```

## 注册组件

首先在assets文件夹中创建名为icons的文件夹，并在其中创建index.js，内容如下

```js
import Vue from 'vue'
import SvgIcon from '@/components/SvgIcon'// svg component

// register globally
Vue.component('svg-icon', SvgIcon)

const req = require.context('./svg', false, /\.svg$/)
const requireAll = requireContext => requireContext.keys().map(requireContext)
requireAll(req)
```

接着，在icons文件夹中创建svg文件夹，可以将所有的svg代码文件放在其中，目录结构如下

```
src/
|--assets/
	|--icons/
	|	|-- xxx.svg
	|	|-- xxx.svg
	|--index.js
```

然后在`main.js`中调用这个index.js文件

```js
import Vue from 'vue'
...

import './assets/icons'
```

## 配置loader规则

最后在`vue.config.js`文件中添加`chainWebpack`配置，如下

```js
'use strict'
const path = require('path')

function resolve(dir) {
  return path.join(__dirname, dir)
}

module.exports = {
  devServer: {
    ...
  },
  configureWebpack: {
    ...
  },
  chainWebpack(config) {
    // set svg-sprite-loader
    config.module
      .rule('svg')
      .exclude.add(resolve('src/assets/icons'))
      .end()
    config.module
      .rule('icons')
      .test(/\.svg$/)
      .include.add(resolve('src/assets/icons'))
      .end()
      .use('svg-sprite-loader')
      .loader('svg-sprite-loader')
      .options({
        symbolId: 'icon-[name]'
      })
      .end()
  }
}
```

## 使用

假如引入了一个名为`vip.svg`的文件放在了`assets/icons/svg`中

```html
<svg-icon icon-class="vip" />
```

