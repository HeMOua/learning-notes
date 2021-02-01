# Webpack简介

## webpack 是什么

Webpack是一种前端资源构建工具，一个静态模块打包器（module bundler）。在Webpack看来，前端的所有资源文件（js/json/css/img/less/..）都会作为模块处理。它将根据模块的依赖关系进行静态分析，打包生成对应的静态资源（bunlle）。

1、webpack先找到一个入口文件，比如 index.js，然后根据这个入口引入不同的依赖

2、将所以代码引进后形成一个代码块，称为 chunk 代码块

3、然后对chunck内不同的资源，比如 less -> css, js 等进行处理

4、最后对这些处理后的文件进行打包，形成 bundle.js

## webpack 五个核心概念

### Entry

==入口（Entry）指示Webpack以哪个文件为入口起点开始打包，分析构建内部依赖图。==

入口起点 (entry point) 指示 webpack 应该使用哪个模块，来作为构建其内部 依赖图(dependency graph) 的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

默认值是 `./src/index.js`，但你可以通过在 webpack configuration 中配置 entry 属性，来指定一个（或多个）不同的入口起点。例如：

**webpack.config.js**

```js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

### Output

==输出（Output）指示Webpack打包后的资源bundles输出到哪里去，以及如何命名。==

output 属性告诉 webpack 在哪里输出它所创建的 bundle，以及如何命名这些文件。主要输出文件的默认值是 `./dist/main.js`，其他生成文件默认放置在 `./dist `文件夹中。

你可以通过在配置中指定一个 output 字段，来配置这些处理过程：

**webpack.config.js**

```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

在上面的示例中，我们通过 `output.filename` 和 `output.path` 属性，来告诉 webpack bundle 的名称，以及我们想要 bundle 生成(emit)到哪里。

### Loader

==Loader让Webpack能够去处理那些非JavaScript文件（webpack自身只理解JavaScript 和 json）==简单来说loader就像一个翻译官，将 webpack 不认识的非 js 文件翻译成它认识的

webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 模块，以供应用程序使用，以及被添加到依赖图中。

> loader 能够 import 导入任何类型的模块（例如 .css 文件），这是 webpack 特有的功能，其他打包程序或任务执行器的可能并不支持。我们认为这种语言扩展是很有必要的，因为这可以使开发人员创建出更准确的依赖关系图。

在更高层面，在 webpack 的配置中，loader 有两个属性：

1. test 属性，识别出哪些文件会被转换。
2. use 属性，定义出在进行转换时，应该使用哪个 loader。

**webpack.config.js**

```js
const path = require('path');

module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};
```


以上配置中，对一个单独的 module 对象定义了 rules 属性，里面包含两个必须属性：test 和 use。这告诉 webpack 编译器(compiler) 如下信息：

> “嘿，webpack 编译器，当你碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先 use(使用) raw-loader 转换一下。”

> 重要的是要记住，在 webpack 配置中定义 rules 时，要定义在 module.rules 而不是 rules 中。为了使你便于理解，如果没有按照正确方式去做，webpack 会给出警告。


> 请记住，使用正则表达式匹配文件时，你不要为它添加引号。也就是说，/\.txt$/ 与 '/\.txt$/' 或 "/\.txt$/" 不一样。前者指示 webpack 匹配任何以 .txt 结尾的文件，后者指示 webpack 匹配具有绝对路径 '.txt' 的单个文件; 这可能不符合你的意图。

### Plugins

==插件（Plugins）可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量等。==

loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。

想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。

多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建一个插件实例。

**webpack.config.js**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```


在上面的示例中，html-webpack-plugin 为应用程序生成一个 HTML 文件，并自动注入所有生成的 bundle。

### Mode

通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 `production`。

```javascript
module.exports = {
  mode: 'production'
};
```

或者从 [CLI](https://webpack.docschina.org/api/cli/) 参数中传递：

```bash
webpack --mode=development
```

支持以下字符串值：

| 选项          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| `development` | 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 `development`. <br />为模块和 chunk 启用有效的名。 |
| `production`  | 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 `production`。<br />为模块和 chunk 启用确定性的混淆名称，`FlagDependencyUsagePlugin`，<br />`FlagIncludedChunksPlugin`，`ModuleConcatenationPlugin`，<br />`NoEmitOnErrorsPlugin` 和 `TerserPlugin` 。 |
| `none`        | 不使用任何默认优化选项                                       |

如果没有设置，webpack 会给 `mode` 的默认值设置为 `production`。

# webpack使用

## webpack初体验

0、安装依赖并编写代码

```shell
webpack i webpack-cli@3.3.11 webpack@4.41.6
```

```js
function add(x, y){
  return x + y;
}

console.log(add(2, 3))
```

1、打包

```shell
webpack ./src/index.js -o ./build/bundle.js --mode=devlopment
```

webpack会以 `./src/index.js` 为入口文件开始打包，打包后输出到 `./build/bundle.js`，整体打包环境就是

2、输出

```shell
# 文件的唯一标识
Hash: 1fd1660ae1dd74b65ce1
Version: webpack 4.41.6
Time: 77ms
Built at: 2021-01-30 1:52:41 ├F10: PM┤
    Asset      Size  Chunks             Chunk Names
bundle.js  3.81 KiB    main  [emitted]  main
Entrypoint main = bundle.js
# 下面就是参与打包的文件
[./src/index.js] 39 bytes {main} [built]
```

3、测试打包包含 json 或 css 文件

3.1 当打包 json 文件时并没有发生错误，如下

```shell
Hash: d75b68ed2fbd1ee3d29c
Version: webpack 4.41.6
Time: 305ms
Built at: 2021-01-30 2:03:06 ├F10: PM┤
    Asset      Size  Chunks             Chunk Names
bundle.js  1.03 KiB       0  [emitted]  main
Entrypoint main = bundle.js
[0] ./src/data.json 25 bytes {0} [built]
[1] ./src/index.js 125 bytes {0} [built]
```

3.2 现在加载一个 css 文件时，代码如下

```js
import data from './data.json'
import './index.css'

function add(x, y){
  return x + y;
}

console.log(add(2, 3))

console.log(data.name)
```

输出如下，即打包失败

```shell
Hash: c6e980fae6967ffc517a
Version: webpack 4.41.6
Time: 92ms
Built at: 2021-01-30 2:08:51 ├F10: PM┤
    Asset      Size  Chunks             Chunk Names
bundle.js  5.34 KiB    main  [emitted]  main
Entrypoint main = bundle.js
[./src/data.json] 25 bytes {main} [built]
[./src/index.css] 270 bytes {main} [built] [failed] [1 error]
[./src/index.js] 147 bytes {main} [built]

ERROR in ./src/index.css 1:0
Module parse failed: Unexpected token (1:0)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders     
> *{
|     padding: 0;
|     margin: 0;
 @ ./src/index.js 2:0-20
npm ERR! code ELIFECYCLE
npm ERR! errno 2
npm ERR! webpack_test@1.0.0 build: `webpack ./src/index.js -o ./build/bundle.js --mode=development`
npm ERR! Exit status 2
```

> **总结**
>
> 1. webpack能处理 js/json 资源，不能处理 css/img 等其他资源
> 2. 生产环境和开发环境将ES6模块化编译成浏览器能识别的模块化~
> 3. 生产环境比开发环境多一个压缩 js 代码的步骤。

## 打包样式资源

0、安装依赖

```shell
npm i webpack@4 webpack-cli@3 -D    
npm i css-loader style-loader -D
```

1、先在与 src 同级的目录下创建 `webapck.config.js`文件，它可以进行 webpack 的相关配置

> 注意：所有构建工具都是基于nodejs平台运行的~模块化默认采用commonjs。
>
> 所以 src 中的代码是 ES6 语法，而 webpack.config.js 是commonjs

**webpack.config.js**

```js
const { resolve } = require('path')

module.exports = {
  // 入口起点
  entry: './src/index.js',
  output: {
    // 输出文件名
    filename: 'bundle.js',
    // 输出路径
    path: resolve(__dirname, 'build')
  },
  module: {
    rules: [
      // 详细 loader 配置
      {
        test: /\.css$/,
        use: [    
          //use数组中1oader执行顺序：从右到左，从下到上依次执行
          //创建style标签，将s中的样式资源插入进行，添加到head中生效     
          'style-loader',
          //将css文件变成commonjs模块加载到js中，里面内容是样式字符串
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    // 详细 plugins 配置
  ],
  mode: 'development'
}
```

可以看到 module > rules 下的规则，他们的作用如下

+ style-loader：创建style标签，将 js 中的样式资源插入进行，添加到head中生效     
+ css-loader：将css文件变成 commonjs 模块加载到js中，里面内容是样式字符串

> 需要注意的是，规则加载顺序是反着来的

这样配置，就可以打包完成了

3、打包 less

需要新增如下依赖

```shell
npm i less less-loader -D
```

并添加规则

**webpack.config.js**

```js
module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader','css-loader']
      },{
        test: /\.less/,
        use:['style-loader','css-loader','less-loader']
      }
    ]
  },
```

## 打包HTML资源

1、新增依赖

```shell
npm i html-webpack-plugin -D
```

2、修改配置

**webpack.config.js**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    ...
    plugins: [
        new HtmlWebpackPlugin()
    ]
}
```

3、运行 webpack 指令后输入如下

```shell
Hash: c8a21934fda90404c817
Version: webpack 4.46.0
Time: 104ms
Built at: 2021-01-30 3:04:37 ├F10: PM┤
     Asset       Size  Chunks             Chunk Names
 bundle.js   3.77 KiB    main  [emitted]  main
index.html  227 bytes          [emitted]
Entrypoint main = bundle.js
[./src/index.js] 0 bytes {main} [built]
```

可以发现，多输出了一个 html 文件

==所以 webpack 指令的作用就是自动生成一个 html 文件，并在最后一行自动添加 bundle.js==

4、但是新输出的文件里面是空的，所以进行以下修改

```js
plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    })
  ],
```

然后在 `./public/index.html` 中编写自己的 html 骨架，然后这个插件就会以它为模板，在最底下添加 script 标签

## 打包图片资源

1. 运行`npm i url-loader file-loader -D`
2. 在`webpack.config.js`中添加处理url路径的loader模块：

```js
{ test: /\.(png|jpg|gif)$/, use: 'url-loader' }
```

+ 参数
  + `limit`指定进行base64编码的图片大小；只有小于指定字节（byte）的图片才会进行base64编码：
  + `name`打包后图片的名字

```js
{ test: /\.(png|jpg|gif)$/, use: 'url-loader?limit=410&name=[hash:8]-[name].[ext]'},
{ test: /\.(ttf|eot|svg|woff|woff2)$/, use: 'url-loader' },
```

或者

```json
{ 
    test: /\.(png|jpg|gif)$/, 
    use: 'url-loader',
    options: {
        limit: 8 * 1024 // 小于 8 kb 就会base64处理
    }    
}
```

