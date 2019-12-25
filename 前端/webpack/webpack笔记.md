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

# webapck开发环境基本配置

## 打包样式资源

### 打包 css

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
      { test: /\.css$/, use: ['style-loader','css-loader'] }
    ]
  },
  plugins: [
    // 详细 plugins 配置
  ],
  mode: 'development'
}
```

可以看到 module > rules 下的规则，他们的作用如下

+ `style-loader`：创建style标签，将 js 中的样式资源插入进行，添加到head中生效     
+ `css-loader`：将css文件变成 commonjs 模块加载到js中，里面内容是样式字符串

> 需要注意的是，规则加载顺序是反着来的

这样配置，就可以打包完成了

### 打包 less

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

### 打包 sass

1. 运行`cnpm i sass-loader node-sass -D`
2. 在`webpack.config.js`中添加处理sass文件的loader模块：

```js
{ test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }
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

### 引用在 css 文件中

1. 运行`npm i url-loader file-loader -D`  注意：url-loader 依赖与 file-loader

2. 在`webpack.config.js`中添加处理url路径的loader模块：

```js
{ test: /\.(png|jpg|gif)$/, use: 'url-loader' }
```

+ 参数
  + `limit`指定进行base64编码的图片大小；只有小于指定字节（byte）的图片才会进行base64编码，可能会处理后的图片体积变大
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
        limit: 8 * 1024, // 小于 8 kb 就会base64处理
        name: '[hash:10].[ext]' // 重命名文件名：取文件名前10位，并用原来的拓展名
    }    
}
```

### 内嵌在 html 中

1. 运行`npm i html-loader -D` 

2. 在`webpack.config.js`中添加处理url路径的loader模块：

```json
{ test: /\.html$/, use: 'html-loader' }
```

3. 这时编译成功后并不会报错，但是前往 html 页面会发现如下情况

```html
<body>
	<img src="[object Module]">
</body>
```

内嵌的 img 的 src 属性变成了 `[object Moudule]`，这是因为在==上面的配置的 url-loader 默认使用的是 es6 模块化解析，而 html-loader 默认使用 commonjs 解析==

4. 要解决这个问题需要关闭 url-loader 的 es6 模块化，使用 commonjs 解析

```json
{ 
    test: /\.(png|jpg|gif)$/, 
    use: 'url-loader',
    options: {
        limit: 8 * 1024, // 小于 8 kb 就会base64处理
        esModule: false // 关闭 url-loader 的 es6 模块化
    }    
}
```

## 打包其他资源

其他资源就是指那些不用做任何预处理，直接引用的资源，比如：字体资源

1. 引入依赖 `npm i file-loader -D`

2. 在`webpack.config.js`中添加处理，exclude 排除文件，其他文件用 file-loader 处理

```json
rules: [
    { test: /\.css$/, use: ['style-loader', 'css-loader'] },
    { 
        exclude: /\.(css|js|html|less)$/, 
        loader: 'file-loader', 
        options: {
            name: '[hash:10].[ext]'
        } 
    }
]
```

## devServer

**简单介绍**

在此之前，每次改完代码后都要重新打包，然后才能看到修改后的效果，这样会十分麻烦，于是 devServer 为我们提供了便利：用来自动化，==帮我们自动编译，自动打开浏览器，自动刷新浏览器==

他只会在内存中编译打包，没有文件输出

**安装依赖**

```shell
npm i webpack-dev-server -D
```

**配置**

```json
module.exports = {
  ...
  devServer: {
    contentBase: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true
  },
```

+ contentBase：告诉服务器内容的来源。仅在需要提供静态文件时才进行配置
+ compress：启动gzip压缩
+ port：服务器端口号
+ open：自动打开浏览器
+ hot：热更新代码 // 优化开发环境

## 不同资源输出路径

一般需要将不同后缀的资源输出到不同的文件夹下面，比如js文件放到js目录下，图片资源放在img文件夹下等等。那么就需要修改输出路径的配置

```json
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { resolve } = require('path')

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'js/bundle.js',
        path: resolve(__dirname, 'build')
    },
    ...
    module: {
        rules: [
            { test: /\.css$/, use: ['style-loader', 'css-loader'] },
            { 
                exclude: /\.(css|js|html|less)$/, 
                loader: 'file-loader', 
                options: {
                    name: '[hash:10].[ext]',
                    outputPath: 'img'
                }                
            },
            { test: /\.(ttf|eot|svg|woff|woff2)$/, use: 'url-loader' }
        ]
    },
    mode: 'development'
}
```

需要配置 options 下的 outputPath 属性

# webapck生产环境基本配置

## 提取 css 文件

**安装依赖**

```shell
 npm i mini-css-extract-plugin -D
```

**配置**

```json
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExt = require('mini-css-extract-plugin')
const { resolve } = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: resolve(__dirname, 'build')
  },
  devServer: {
    contentBase: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true
  },
  module: {
    rules: [
      { test: /\.css$/, use: [MiniCssExt.loader, 'css-loader'] }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    new MiniCssExt({
        filename: 'css/main.css'
    })
  ],
  mode: 'development'
}
```

共有两个要点

+ 在 plugins 中添加一个插件 `mini-css-extract-plugin`，如果想要修改输出的文件夹目录，可以添加 filename 属性

+ 在对 css 的规则处理中，将之前的 style-loader 改成 `MiniCssExt.loader`
  + css-loader：将css文件整合到js文件中
  + 问题：经过css-loader处理后，样式文件是在js文件中的
    + js文件体积会很大
    + 需要先加载js再动态创建style标签，样式渲染速度就慢，会出现闪屏现象
  + 解决：用MiniCssExtractPlugin.loader替代style-loader
  + MiniCssExtractPlugin.loader：提取js中的css成单独文件然后通过link加载

## css兼容性配置

css兼容性处理：postcss

**安装依赖**

```shell
npm i postcss-loader postcss-preset-env -D
```

**配置**

**webpack.config.js**

```json
// 复用loader的写法
const commonCssLoader = [
  MiniCssExtractPlugin.loader,
  'css-loader',
  {
    loader: 'postcss-loader',
    options: {
      ident: 'postcss', // 基本写法
      plugins: () => [
        // postcss的插件
        require('postcss-preset-env')(),
      ],
    },
  },
]

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: resolve(__dirname, 'build')
  },
  devServer: {
    contentBase: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true
  },
  module: {
    rules: [
      { test: /\.css$/, use: [...commonCssLoader] }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    new MiniCssExt()
  ],
  mode: 'development'
}
```

此外postcss需要通过package.json中browserslist里面的配置加载指定的css兼容性样式

在package.json中定义browserslist：

**package.json**

```json
"browserslist": {
    // 开发环境 --> 设置node环境变量：process.env.NODE_ENV = development
    "development": [ // 只需要可以运行即可
        "last 1 chrome version", // 最新1版的 chrome
        "last 1 firefox version",
        "last 1 safari version"
    ],
    // 生产环境。默认是生产环境
    "production": [ // 需要满足绝大多数浏览器的兼容
        ">0.2%",
        "not dead",
        "not op_mini all"
    ]
}
```

**如若发生报错**，需要在webpack.config.js文件所在目录下创建一个同级的postcss.config.js文件然后写入一下内容

```json
module.exports = {
    plugins: [
        //使用postcss插件
        require('postcss-preset-env')
    ]
}
```

最后将webpack文件中的其他配置去掉，改用默认配置 'postcss-loader'，如下

```json
module: {
    rules: [
        { 
            test: /\.css$/, 
            use: [
                MiniCssExt.loader,
                'css-loader',
                'postcss-loader'
            ] 
        }
    ]
}
```

==关于 node.js 的环境变量==

node.js 有个内部的环境变量，可以通过如下修改开发环境还是生产环境

```js
process.env.NODE_ENV = 'development' // 或者 production
```

node.js **默认**的环境就是 生产环境 production

环境的不同会应用不同的在 package.json 中的 browserslist 中的配置

## 压缩 css

**安装依赖**

```shell
npm i optimizi-css-assets-webpack-plugin -D
```

**配置**

```js
const OptimiziCssAssetsWebpackPlugin = require('optimizi-css-assets-webpack-plugin')

...
plugins: [
    new HtmlWebpackPlugin({
        template: './public/index.html'
    }),
    new MiniCssExt(),
    new OptimiziCssAssetsWebpackPlugin()
]
```

要点：

+ 声明插件 require
+ 添加插件 

## js 语法检测

**安装依赖**

```shell
npm i eslint-loader eslint eslint-config-airbnb-base eslint-plugin-import -D
```

**配置**

**webpack.conf.js**

```json
module: {
    rules: [
        { 
            test: /\.js$/,
            exclude: /node_modules/, // 忽略node_modules
            enforce: 'pre', // 优先执行
            loader: 'eslint-loader',
            options: {
                // 自动修复
                fix: true,
            }
        }
    ]
}
```

**package.json**

添加配置

```json
"eslintConfig": {
    "extends": "airbnb-base"
}
```

如果在某一行 js 上面添加如下注释

```js
// eslint-disable-next-line
```

则不会对其下面一行进行语法检测

## js 兼容性处理(没实操成)

**安装依赖**

```shell
npm i babel-loader @babel/core @babel/preset-env -D
```

**配置**

第一种方法：基本js兼容性处理 --> @babel/preset-env

问题：能转换基本语法，如promise高级语法不能转换

```json
{
    test: /\.js$/,
    exclude: /node_modules/,
    enforce: 'pre' // 优先处理
    loader: 'babel-loader',
    options: {
        // 预设：指示babel做怎样的兼容性处理
        presets: ['@babel/preset-env']
    }
}
```

第二种方法

全部js兼容性处理，引入 @babel/polyfill
问题：只要解决部分兼容性问题，但是将所有兼容性代码全部引入，体积太大了

```js
import '@babel/polyfill'
```

第三种方法

```json
{
    // 第三种方式：按需加载
    test: /\.js$/,
    exclude: /node_modules/,
    loader: 'babel-loader',
    options: {
        // 预设：指示babel做怎样的兼容性处理
        presets: [
            '@babel/preset-env', // 基本预设
            {
                useBuiltIns: 'usage', //按需加载
                corejs: { version: 3 }, // 指定core-js版本
                targets: { // 指定兼容到什么版本的浏览器
                    chrome: '60',
                    firefox: '50',
                    ie: '9',
                    safari: '10',
                    edge: '17'
                },
            },
        ],
    }
}
```

## html js 压缩

在开发环境中，会自动压缩js代码

若想压缩 html 代码可以在插件中添加如下配置

```json
new HtmlWebpackPlugin({
    template: './src/index.html',
    // 压缩html代码
    minify: {
        // 移除空格
        collapseWhitespace: true,
        // 移除注释
        removeComments: true,
    },
})
```

不过据实际编写发现，4.x 版本的 HtmlWebpackPlugin 会自动在生产环境下压缩 html 和 js 代码

# webapck优化配置

## 开发环境优化

### 优化打包构建速度

#### HMR

hot module replacement 热模块替换 / 模块热替换

1、作用：当一个模块发生变化时只重新打包这一个模块，而不是将所有模块重新打包

2、使用：添加 hot 属性并置为 true

webpack.config.js

```json
devServer: {
    contentBase: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true,
    hot: true
}
```

3、适用范围

1. 样式文件：可以使用HMR功能，因为开发环境下使用的 style-loader 内部默认实现了热模块替换功能

2. js 文件：默认不能使用HMR功能（修改一个 js 模块所有 js 模块都会刷新）

   但是我们可以通过修改 js 代码来实现 HMR 需要（添加支持 HMR 功能的代码）

   ```js
   // 绑定
   if (module.hot) {
     // 一旦 module.hot 为true，说明开启了HMR功能。 --> 让HMR功能代码生效
     module.hot.accept('./print.js', function() {
       // 方法会监听 print.js 文件的变化，一旦发生变化，只有这个模块会重新打包构建，其他模块不会。
       // 会执行后面的回调函数
       print();
     });
   }
   ```
   ==注意==：HMR 功能对 js 的处理，只能处理非入口 js 文件的其他文件

3. html 文件: 默认不能使用 HMR 功能（html 不用做 HMR 功能，因为只有一个 html 文件，不需要再优化

   使用 HMR 会导致问题：html 文件不能热更新了（不会自动打包构建）

   解决：修改 entry 入口，将 html 文件引入（这样 html 修改整体刷新）

   ```json
   entry: ['./src/js/index.js', './src/index.html']
   ```

### 优化代码调试

#### srouce-map

source-map：一种提供源代码到构建后代码的映射的技术 （如果==构建后代码出错了，通过映射可以追踪源代码错误==）

参数：`[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map`

代码：

```
devtool: 'eval-source-map'
```

可选方案：[生成source-map的位置| 给出的错误代码信息 ]

| 方案                    | 位置     | 错误信息 | 源代码                                       |
| ----------------------- | -------- | -------- | -------------------------------------------- |
| source-map              | 外部     | 有       | 真实位置                                     |
| inline-source-map       | **内联** | 有       | 真实位置                                     |
| hidden-source-map       | 外部     | 有       | 构建后位置                                   |
| eval-source-map         | **内联** | 有       | 真实位置，多了个hash值                       |
| nosources-source-map    | 外部     | 有       | 知道那个代码错误，但不能看                   |
| cheap-source-map        | 外部     | 有       | 真实位置，但是错误只能定位到行               |
| cheap-module-source-map | 外部     | 有       | module 会加入 loader 的 source-map，其他一样 |

内联 和 外部的区别：1. 外部生成了文件，内联没有 2. 内联构建速度更快

**开发环境**：需要考虑速度快，调试更友好

- 速度快( eval > inline > cheap >... )
  1. eval-cheap-souce-map
  2. eval-source-map
- 调试更友好
  1. souce-map
  2. cheap-module-souce-map
  3. cheap-souce-map

最终得出最好的两种方案

+ eval-source-map（完整度高，内联速度快，脚手架默认使用）
+ eval-cheap-module-souce-map（错误提示忽略列但是包含其他信息，内联速度快）

自注：

+ 开发用 `cheap-source-map`（若依项目就是这样）

## 生产环境优化

### 优化打包构建速度

#### oneOf

oneOf：==匹配到 loader 后就不再向后进行匹配==，优化生产环境的打包构建速度

注意：不能有两个配置处理同一种类型文件（所以把eslint-loader提取出去放外面）

代码：

```json
module: {
    rules: [
        {
            test: /\.js$/,
            exclude: /node_modules/,
            enforce: 'pre',
            loader: 'eslint-loader',
            options: { fix: true }
        },
        {
            oneOf: [
                {test: /\.css$/, use: [...commonCssLoader] },
                {
                    // js 兼容性处理
                    test: /\.js$/,
                    exclude: /node_modules/,
                    loader: 'babel-loader',
                    options: {
                        ...
                    }
                }
                ...
            ]
        }
    ]
},
```

#### babel 缓存

babel 缓存：类似 HMR，将 babel 处理后的资源缓存起来（哪里的 js 改变就更新哪里，其他 js 还是用之前缓存的资源），让第二次打包构建速度更快

代码：

```json
{
  test: /\.js$/,
  exclude: /node_modules/,
  loader: 'babel-loader',
  options: {
    presets: [
      [
        '@babel/preset-env',
        {
          useBuiltIns: 'usage',
          corejs: { version: 3 },
          targets: {
            chrome: '60',
            firefox: '50'
          }
        }
      ]
    ],
    // 开启babel缓存
    // 第二次构建时，会读取之前的缓存
    cacheDirectory: true
  }
},
```

#### 多进程打包

多进程打包：某个任务消耗时间较长会卡顿，多进程可以同一时间干多件事，效率更高。

优点是提升打包速度，缺点是每个进程的开启和交流都会有开销（babel-loader消耗时间最久，所以使用thread-loader 针对其进行优化）

```json
{
  test: /\.js$/,
  exclude: /node_modules/,
  use: [
    /* 
      thread-loader会对其后面的loader（这里是babel-loader）开启多进程打包。 
      进程启动大概为600ms，进程通信也有开销。(启动的开销比较昂贵，不要滥用)
      只有工作消耗时间比较长，才需要多进程打包
    */
    {
      loader: 'thread-loader',
      options: {
        workers: 2 // 进程2个
      }
    },
    {
      loader: 'babel-loader',
      options: {
        presets: [
          [
            '@babel/preset-env',
            {
              useBuiltIns: 'usage',
              corejs: { version: 3 },
              targets: {
                chrome: '60',
                firefox: '50'
              }
            }
          ]
        ],
        cacheDirectory: true
      }
    }
  ]
}
```

#### externals

externals：让某些库不打包，通过 cdn 引入

webpack.config.js 中配置：

```js
externals: {
  // 拒绝jQuery被打包进来(通过cdn引入，速度会快一些)
  // 忽略的库名 -- npm包名
  jquery: 'jQuery'
}
```

需要在 index.html 中通过 cdn 引入：

```html
<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
```

#### dll

dll：让某些库单独打包，后直接引入到 build 中。可以在 code split 分割出 node_modules 后再用 dll 更细的分割，优化代码运行的性能。

webpack.dll.js 配置：(将 jquery 单独打包)

```js
/*
  node_modules的库会打包到一起，但是很多库的时候打包输出的js文件就太大了
  使用dll技术，对某些库（第三方库：jquery、react、vue...）进行单独打包
  当运行webpack时，默认查找webpack.config.js配置文件
  需求：需要运行webpack.dll.js文件
    --> webpack --config webpack.dll.js（运行这个指令表示以这个配置文件打包）
*/
const { resolve } = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    // 最终打包生成的[name] --> jquery
    // ['jquery] --> 要打包的库是jquery
    jquery: ['jquery']
  },
  output: {
    // 输出出口指定
    filename: '[name].js', // name就是jquery
    path: resolve(__dirname, 'dll'), // 打包到dll目录下
    library: '[name]_[hash]', // 打包的库里面向外暴露出去的内容叫什么名字
  },
  plugins: [
    // 打包生成一个manifest.json --> 提供jquery的映射关系（告诉webpack：jquery之后不需要再打包和暴露内容的名称）
    new webpack.DllPlugin({
      name: '[name]_[hash]', // 映射库的暴露的内容名称
      path: resolve(__dirname, 'dll/manifest.json') // 输出文件路径
    })
  ],
  mode: 'production'
};
```

webpack.config.js 配置：(告诉 webpack 不需要再打包 jquery，并将之前打包好的 jquery 跟其他打包好的资源一同输出到 build 目录下)

```js
// 引入插件
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');

// plugins中配置：
plugins: [
  new HtmlWebpackPlugin({
    template: './src/index.html'
  }),
  // 告诉webpack哪些库不参与打包，同时使用时的名称也得变
  new webpack.DllReferencePlugin({
    manifest: resolve(__dirname, 'dll/manifest.json')
  }),
  // 将某个文件打包输出到build目录下，并在html中自动引入该资源
  new AddAssetHtmlWebpackPlugin({
    filepath: resolve(__dirname, 'dll/jquery.js')
  })
],
```

### 优化代码运行性能

#### 文件资源缓存

文件名不变，就不会重新请求，而是再次用之前缓存的资源

1. hash： 每次 wepack 打包时会生成一个唯一的 hash 值。

   + 配置：`filename: '[hash:10].[ext]'`

   + 问题：重新打包，所有文件的 hash 值都改变，会导致所有缓存失效。（可能只改动了一个文件）

2. chunkhash：根据 chunk 生成的 hash 值。来源于同一个 chunk的 hash 值一样 

   + 配置：`filename: '[chunkhash:10].[ext]'`

   + 问题：js 和 css 来自同一个chunk，hash 值是一样的（因为 css-loader 会将 css 文件加载到 js 中，所以同属于一个chunk）

3. contenthash： 根据文件的内容生成 hash 值。不同文件 hash 值一定不一样(文件内容修改，文件名里的 hash 才会改变)

   + 配置：`filename: '[contenthash:10].[ext]'`

   + 修改 css 文件内容，打包后的 css 文件名 hash 值就改变，而 js 文件没有改变 hash 值就不变，这样 css 和 js 缓存就会分开判断要不要重新请求资源 --> 让代码上线运行缓存更好使用

#### tree shaking

tree shaking：去除无用代码

前提：1. 必须使用 ES6 模块化 2. 开启 production 环境 （这样就自动会把无用代码去掉）

作用：减少代码体积

在 package.json 中配置：

`"sideEffects": false` 表示所有代码都没有副作用（都可以进行 tree shaking）

这样会导致的问题：可能会把 css / @babel/polyfill 文件干掉（副作用）

所以可以配置：`"sideEffects": ["*.css", "*.less"]` 不会对css/less文件tree shaking处理

#### code split

代码分割。将打包输出的一个大的 bundle.js 文件拆分成多个小文件，这样可以并行加载多个文件，比加载一个文件更快。

方法一：多入口拆分

webpack.config.js

```json
module.exports = {
    entry: {
        // 多入口：有一个入口，最终输出就有一个bundle
        index: './src/js/index.js',
        test: './src/js/test.js'
    },
    output: {
        // [name]：取文件名
        filename: 'js/[name].[contenthash:10].js',
        path: resolve(__dirname, 'build')
    }
    ...
}
```

方法二：

webpack.config.js

```json
module.exports = {
    ...
    optimization: {
        splitChunks: {
            chunks: 'all'
        }
    }
}
```

- 将 node_modules 中的代码单独打包（大小超过30kb）
- 自动分析多入口chunk中，有没有公共的文件。如果有会打包成单独一个chunk(比如两个模块中都引入了jquery会被打包成单独的文件)（大小超过30kb）

方法三：import 动态导入语法：

index.js

```js
/*
  通过js代码，让某个文件被单独打包成一个chunk
  import动态导入语法：能将某个文件单独打包(test文件不会和index打包在同一个文件而是单独打包)
  webpackChunkName:指定test单独打包后文件的名字
*/
import(/* webpackChunkName: 'test' */'./test')
  .then(({ mul, count }) => {
    // 文件加载成功~
    // eslint-disable-next-line
    console.log(mul(2, 5));
  })
  .catch(() => {
    // eslint-disable-next-line
    console.log('文件加载失败~');
  });
```

####  lazy loading

1.懒加载：当文件需要使用时才加载（需要代码分割）。但是如果资源较大，加载时间就会较长，有延迟。

2.正常加载：可以认为是并行加载（同一时间加载多个文件）没有先后顺序，先加载了不需要的资源就会浪费时间。

3.预加载 prefetch（兼容性很差）：会在使用之前，提前加载。等其他资源加载完毕，浏览器空闲了，再偷偷加载这个资源。这样在使用时已经加载好了，速度很快。所以在懒加载的基础上加上预加载会更好。

代码：

```js
document.getElementById('btn').onclick = function() {
  // 将import的内容放在异步回调函数中使用，点击按钮，test.js才会被加载(不会重复加载)
  // webpackPrefetch: true表示开启预加载
  import(/* webpackChunkName: 'test', webpackPrefetch: true */'./test').then(({ mul }) => {
    console.log(mul(4, 5));
  });
  import('./test').then(({ mul }) => {
    console.log(mul(2, 5))
  })
};
```

#### pwa

#### 离线可访问技术

pwa：离线可访问技术（渐进式网络开发应用程序），使用 serviceworker 和 workbox 技术。优点是离线也能访问，缺点是兼容性差。

webpack.config.js 

```js
const WorkboxWebpackPlugin = require('workbox-webpack-plugin'); // 引入插件

// plugins中加入：
new WorkboxWebpackPlugin.GenerateSW({
  /*
    1. 帮助serviceworker快速启动
    2. 删除旧的 serviceworker

    生成一个 serviceworker 配置文件
  */
  clientsClaim: true,
  skipWaiting: true
})
```

index.js 中还需要写一段代码来激活它的使用：

```js
/*
  1. eslint不认识 window、navigator全局变量
    解决：需要修改package.json中eslintConfig配置
    "env": {
      "browser": true // 支持浏览器端全局变量
    }
  2. sw代码必须运行在服务器上
    --> nodejs
    或-->
      npm i serve -g
      serve -s build 启动服务器，将打包输出的build目录下所有资源作为静态资源暴露出去
*/
if ('serviceWorker' in navigator) { // 处理兼容性问题
  window.addEventListener('load', () => {
    navigator.serviceWorker
      .register('/service-worker.js') // 注册serviceWorker
      .then(() => {
        console.log('sw注册成功了~');
      })
      .catch(() => {
        console.log('sw注册失败了~');
      });
  });
}
```

# webapck详细配置

## entry

entry: 入口起点

1. string --> './src/index.js'，单入口

   打包形成一个 chunk。 输出一个 bundle 文件。此时 chunk 的名称默认是 main

2. array --> ['./src/index.js', './src/add.js']，多入口

   所有入口文件最终只会形成一个 chunk，输出出去只有一个 bundle 文件。

   （一般只用在 HMR 功能中让 html 热更新生效）

3. object，多入口

   有几个入口文件就形成几个 chunk，输出几个 bundle 文件，此时 chunk 的名称是 key 值

--> 特殊用法：

```json
entry: {
  // 形成一个chunk, 合并输出出去只有一个bundle文件。
  index: ['./src/index.js', './src/count.js'], 
  // 形成一个chunk，输出一个bundle文件。
  add: './src/add.js'
}
```

## output

```json
output: {
  // 文件名称（指定名称+目录）
  filename: 'js/[name].js',
  // 输出文件目录（将来所有资源输出的公共目录）
  path: resolve(__dirname, 'build'),
  // 所有资源引入公共路径前缀 --> 'imgs/a.jpg' --> '/imgs/a.jpg'
  publicPath: '/',
  chunkFilename: 'js/[name]_chunk.js', // 指定非入口chunk的名称
  library: '[name]', // 打包整个库后向外暴露的变量名
  libraryTarget: 'window' // 变量名添加到哪个上 browser：window
  // libraryTarget: 'global' // node：global
  // libraryTarget: 'commonjs' // conmmonjs模块 exports
  // 结果就是： window['name'] = ...
},
```

## module

```json
module: {
  rules: [
    // loader的配置
    {
      test: /\.css$/,
      // 多个loader用use
      use: ['style-loader', 'css-loader']
    },
    {
      test: /\.js$/,
      // 排除node_modules下的js文件
      exclude: /node_modules/,
      // 只检查src下的js文件
      include: resolve(__dirname, 'src'),
      enforce: 'pre', // 优先执行
      // 不写就是中间执行
      // enforce: 'post', // 延后执行
      // 单个loader用loader
      loader: 'eslint-loader',
      options: {} // 指定配置选项
    },
    {
      // 以下配置只会生效一个
      oneOf: []
    }
  ]
},
```

## resolve

```json
// 解析模块的规则
resolve: {
  // 配置解析模块路径别名: 优点：当目录层级很复杂时，简写路径；缺点：路径不会提示
  alias: {
    $css: resolve(__dirname, 'src/css')
  },
  // 配置省略文件路径的后缀名（引入时就可以不写文件后缀名了）
  extensions: ['.js', '.json', '.jsx', '.css'],
  // 告诉 webpack 解析模块应该去找哪个目录
  modules: [resolve(__dirname, '../../node_modules'), 'node_modules']
}
```

这样配置后，引入文件就可以这样简写：`import '$css/index';`

## dev server

```json
devServer: {
  // 运行代码所在的目录
  contentBase: resolve(__dirname, 'build'),
  // 监视contentBase目录下的所有文件，一旦文件变化就会reload
  watchContentBase: true,
  watchOptions: {
    // 忽略文件
    ignored: /node_modules/
  },
  // 启动gzip压缩
  compress: true,
  // 端口号
  port: 5000,
  // 域名
  host: 'localhost',
  // 自动打开浏览器
  open: true,
  // 开启HMR功能
  hot: true,
  // 不要显示启动服务器日志信息
  clientLogLevel: 'none',
  // 除了一些基本信息外，其他内容都不要显示
  quiet: true,
  // 如果出错了，不要全屏提示
  overlay: false,
  // 服务器代理，--> 解决开发环境跨域问题
  proxy: {
    // 一旦devServer(5000)服务器接收到/api/xxx的请求，就会把请求转发到另外一个服务器3000
    '/api': {
      target: 'http://localhost:3000',
      // 发送请求时，请求路径重写：将/api/xxx --> /xxx （去掉/api）
      //proxy的changeOrigin是false：请求头中host仍然是浏览器发送过来的host；如果设置成true：发送请求头中host会设置成target
      changeOrigin: true,
      pathRewrite: {
        '^/api': ''
      }
    }
  }
}
```

其中，跨域问题：同源策略中不同的协议、端口号、域名就会产生跨域。

正常的浏览器和服务器之间有跨域，但是服务器之间没有跨域。代码通过代理服务器运行，所以浏览器和代理服务器之间没有跨域，浏览器把请求发送到代理服务器上，代理服务器替你转发到另外一个服务器上，服务器之间没有跨域，所以请求成功。代理服务器再把接收到的响应响应给浏览器。这样就解决开发环境下的跨域问题。

# webapck5 介绍