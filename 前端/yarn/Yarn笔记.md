# Yarn手册

## 快速入门

### 使用方法

**初始化一个新项目**

```
yarn init
```

**添加依赖包**

```shell
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]
```

**将依赖项添加到不同依赖项类别中**

分别添加到 `devDependencies`、`peerDependencies` 和 `optionalDependencies` 类别中：

```
yarn add [package] --dev
yarn add [package] --peer
yarn add [package] --optional
```

**升级依赖包**

```
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]
```

**移除依赖包**

```
yarn remove [package]
```

**安装项目的全部依赖**

```
yarn
```

或者

```shell
yarn install
```

## 配置全局安装路径

### 查看路径

**查看全局bin位置**

```
yarn global bin
```

**查看全局安装位置**

```
yarn global dir
```

**查看全局cache位置**

```
yarn cache dir
```

### 修改路径

**修改全局bin路径**

```
yarn config set prefix E:\Cache\yarn
```

**修改全局安装路径**

```
yarn config set global-folder E:\Cache\yarn\Data\global
```

**修改全局Cache路径**

```
yarn config set cache-folder E:\Cache\yarn\Cache
```

**修改全局link路径**

```
yarn config set link-folder E:\Cache\yarn\Data\link
```

## 换源

**查看源**

```
yarn config get registry
```

**换taobao源**

```
yarn config set registry https://registry.npm.taobao.org
```

**通过yrm**

首先安装`yrm`，它可以帮助我们找到镜像源，并切换npm和yarn的源

```sh
npm install yrm -g
```

查看所有镜像

```sh
yrm ls
```

切换镜像

```sh
yrm use taobao
```

## 设置代理

```shell
# http
yarn config set proxy http://proxy_host:port  

# https
yarn config set https-proxy http://proxy_host:port  
```



取消

```
yarn config delete proxy
yarn config delete https-proxy
```

