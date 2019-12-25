# NVM安装以及使用

https://github.com/coreybutler/nvm-windows/wiki#manual-installation

## 安装

1. 在 github 下载 nvm-noinstall.zip
2. 设置`NVM_HOME`环境变量，就是 NVM 的所在地址，比如`F:\ServerTool\nvm-window-1.1.7`
3. 设置`NVM_SYMLINK`环境变量，模拟的 nodejs 所在位置，比如`D:\Program Files\Nodejs`
4. 在 `PATH` 中添加上面两个环境变量

## 使用

```shell
nvm use 14.0.0
npm install -g yarn
nvm use 12.0.1
npm install -g yarn
```

