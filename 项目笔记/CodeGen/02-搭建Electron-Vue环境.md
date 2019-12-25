# 搭建Electron-Vue环境

## 安装全局依赖

安装 Electron

```
npm i -g electron
```

安装 Vue-cli

```
npm i -g @vue/cli
```

## 构建项目

首先用 vue-cli 创建项目

```
vue create electron-vue
```

安装完毕后可以先校验 vue 页面是否能运行（也可省略该步骤）

```
cd electron-vue
npm run serve
```

在 vue 项目搭建完毕后安装 Electron

```
npm i electron --dev
// or
yarn add electron --dev
```

## 程序入口

我们需要创建一个 `background.js` 文件，**注意**文件名字要一致，文件放置在如下位置

```
|-electron-vue
	|-src
	  |-background.js *
	  |-App.vue
	|-...
```

文件内容如下

```js
'use strict'
/* global __static */
import path from 'path'
import { app, BrowserWindow } from 'electron'
import { createProtocol } from 'vue-cli-plugin-electron-builder/lib'

function createWindow () {   
  // 创建浏览器窗口
  let win = new BrowserWindow({
    width: 900,
    height: 750,
    webPreferences: {
      nodeIntegration: true
    },
    
    // eslint-disable-next-line no-undef
    icon: path.join(__static, 'icon.png')
  })

  if (process.env.WEBPACK_DEV_SERVER_URL) {
    // Load the url of the dev server if in development mode
    win.loadURL(process.env.WEBPACK_DEV_SERVER_URL)
    if (!process.env.IS_TEST) win.webContents.openDevTools()
  } else {
    createProtocol('app')
    // Load the index.html when not in development
    win.loadURL('app://./index.html')
  }

  win.on('closed', () => {
    win = null
  })
}

app.on('ready', async () => {
  try {
    createWindow()
  } catch (err) {
    console.log(err)
  }
})
```

## 相关依赖

如果想要程序正常运行还需要安装下列依赖

```
yarn add vue-cli-plugin-electron-builder --dev
```

## 运行

修改以下package.json

```json
  "scripts": {
    "serve": "vue-cli-service serve",
    "lint": "vue-cli-service lint",
    "electron": "vue-cli-service electron:serve",
    "electron:build": "vue-cli-service electron:build"
  },
  ...
```

然后运行 `yarn electron`

