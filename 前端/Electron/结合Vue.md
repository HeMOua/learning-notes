# Electron 结合 Vue

## 项目创建

1. 创建 vue 项目，执行 `vue create xxx`
1. cd 进入 xxx 文件夹
2. 安装 `electron` 依赖，执行 `yarn add -D electron`
3. 安装 `electron-builder`，执行 `vue add electron-builder`

会创建一个 `background.js` 文件，它的名字不可更改，并且会在 package.json 文件中多添加 4 个命令