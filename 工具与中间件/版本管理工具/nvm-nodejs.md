[nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions (github.com)](https://github.com/nvm-sh/nvm)

1. 在 github 上下载 nvm-install.zip，解压到想要的地方

2. 设置环境变量

   1. `NVM_HOME`：就是上面解压的位置，表示 nvm 的目录所在

   2. `NVM_SYMLINK`：软链接位置，用来表示 nodejs 所在文件夹
      + 该位置文件夹不能存在，它会由 nvm 自动创建

   3. 将 `NVM_HOME` 以及 `NVM_SYMLINK` 添加到 `Path` 中

3. 在 nvm 目录也就是 `NVM_HOME` 创建配置文件 `setting.txt`，参考如下

```
root: E:\ServerTools\nvm-window
path: D:\Program Files\nodejs
arch: 64
proxy: none
originalpath: 
originalversion: 
node_mirror: http://npmmirror.com/mirrors/node/
npm_mirror: http://registry.npmmirror.com/mirrors/npm/
```

