[toc]

# Node.js 笔记

# 一、安装配置

# 二、NPM的使用

> NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：
>
> + 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
> + 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
> + 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

## 1、安装

新版的nodejs已经集成了npm，所以之前npm也一并安装好了。同样可以通过输入 **"npm -v"** 来测试是否成功安装。命令如下，出现版本提示表示安装成功:

```shell
$ npm -v
2.3.0
```

如果安装的是旧版的NPM可以使用如下命令**升级**

```shell
npm install npm -g
```

或使用**淘宝镜像**
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

这样就可以使用 cnpm 命令来安装模块了：

```
$ cnpm install [name]
```

## 2、安装模块

npm 安装 Node.js 模块语法格式如下：

```shell
$ npm install <Module Name>
```

**全局安装与本地安装**

npm 的包安装分为本地安装（local）、全局安装（global）两种，从敲的命令行来看，差别只是有没有-g而已，比如

```shell
npm install express      # 本地安装
npm install express -g   # 全局安装
```

### 安装缓慢解决

**npm install 安装过慢的解决方案：**
方法一：

该命令可以改变装包用的资源点，使访问外网变为访问国内网

```shell
npm install -gd express --registry=http://registry.npm.taobao.org
```

需要使用–registry参数指定镜像服务器地址，为了避免每次安装都需要–registry参数，可以使用如下命令进行永久设置：

```shell
npm config set registry http://registry.npm.taobao.org
```

方法二：

```shell
npm install -g cnpm -registry=https://registry.npm.taobao.org
```

该代码用来安装淘宝团队的npm。安装完成后遇到需要npm install的时候直接cnpm install 即可

方法三：

首先安装`nrm`，它可以帮助我们找到镜像源，并切换它

```sh
npm install nrm -g
```

查看所有镜像

```sh
nrm ls
```

切换镜像

```sh
nrm use taobao
```

## 3、查看安装信息

你可以使用以下命令来查看所有全局安装的模块：

```shell
$ npm list -g
```

如果要查看某个模块的版本号，可以使用命令如下：

```shell
$ npm list grunt
```

## 4、package.json

> package.json 位于模块的目录下，用于定义包的属性。

**属性说明**如下

+ **name** - 包名。
+ **version** - 包的版本号。
+ **description** - 包的描述。
+ **homepage** - 包的官网 url 。
+ **author** - 包的作者姓名。
+ **contributors** - 包的其他贡献者姓名。
+ **dependencies** - 依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下。
+ **repository** - 包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上。
+ **main** - main 字段指定了程序的主入口文件，require('moduleName') 就会加载这个文件。这个字段的默认值是模块根目录下面的 index.js。
+ **keywords** - 关键字

## 5、其他常用命令

我们可以使用以下命令来**卸载** Node.js 模块。

```shell
$ npm uninstall express
```

我们可以使用以下命令**更新**模块：

```
$ npm update express
```

使用以下来**搜索**模块：

```
$ npm search express
```

**创建模块**，package.json 文件是必不可少的。我们可以使用 NPM 生成 package.json 文件，生成的文件包含了基本的结果。

```shell
$ npm init
```

+ 使用`npm help `可查看某条命令的详细帮助，例如`npm help install`。
+ 在`package.json`所在目录下使用`npm install . -g`可先在本地安装当前命令行程序，可用于发布前的本地测试。
+ 使用`npm update `可以把当前目录下`node_modules`子目录里边的对应模块更新至最新版本。
+ 使用`npm update -g`可以把全局安装的对应命令行程序更新至最新版。
+ 使用`npm cache clear`可以清空NPM本地缓存，用于对付使用相同版本号发布新版本代码的人。
+ 使用`npm unpublish @`可以撤销发布自己发布过的某个版本代码。



# 三、命令

+ npm i jquery --save-dev == npm i jquery -D