# webpack开发速记

## 必备操作

1、使用webpack开发项目第一件事，把webapck、webpack-cli装了

```shell
npm i -D webpack webpack-cli
```

2、配置webpack.config.js

这里强行分为3个步骤，方便记忆

```js
// 第一步： 引入 path 依赖，编写resolve函数
const path = require('path')

const resolve = (dir) => path.resolve(__dirname, dir)

// 第二三步：配置程序入口与出口点
module.exports = {
    entry: './src/main.js',
    output: {
        path: resolve('dist'),
        filename: 'bundle.js'
    }
}
```

## 使用html自定义模板

1、如果需要html的自定义模板，那么第一步`html-webpack-plugin`插件的安装是少不了的

```shell
npm i -D html-webapck-plugin
```

2、引入插件

```js
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
```

3、应用插件并配置html模板

```js
// webpack.config.js
module.exports = {
  ...
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    })
  ]
}
```

## webapck-server快速开发

1. 安装插件

```shell
npm i -D webpack-dev-server
```

2. 添加命令

```json
"start": "webpack serve --open"
```

然后运行 `npm start` 即可

3. 更多配置

```json
module.exports = {
  ...
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
}
```

+ contentBase：告诉服务器内容的来源。仅在需要提供静态文件时才进行配置
+ compress：启动gzip压缩
+ port：服务器端口号
+ open：自动打开浏览器
+ hot：热更新代码 // 优化开发环境

## 处理css

1. 运行`cnpm i style-loader css-loader --save-dev`
2. 修改`webpack.config.js`这个配置文件：

```js
module: { // 用来配置第三方loader模块的
        rules: [ // 文件的匹配规则
            { test: /\.css$/, use: ['style-loader', 'css-loader'] }//处理css文件的规则
        ]
    }
```

3. 注意：`use`表示使用哪些模块来处理`test`所匹配到的文件；`use`中相关loader模块的调用顺序是从后向前调用的；

## 处理sass

1. 运行`cnpm i sass-loader node-sass --save-dev`
2. 在`webpack.config.js`中添加处理sass文件的loader模块：

```js
{ test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }
```

## 处理less文件

1. 运行`cnpm i less-loader less -D`
2. 修改`webpack.config.js`这个配置文件：

```js
{ test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader'] },
```

## babel处理语法兼容

1. 安装插件

```shell
npm i -D @babel/core @babel/preset-env babel-loader core-js
```

2. 配置

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

const resolve = (dir) => path.resolve(__dirname, dir)

module.exports = {
  entry: './src/app.ts',
  output: {
    path: resolve('dist'),
    filename: 'bundle.js',
    // 取消箭头函数
    environment: {
      arrowFunction: false
    }
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              // 设置预定义的插件
              presets: [
                [
                  // 设置预定义的环境
                  "@babel/preset-env",
                  {
                    // 要兼容的目标浏览器
                    targets: {
                      "chrome": "88"
                    },
                    // 指定corejs版本
                    "corejs": "3",
                    // 指定corejs的使用方式为按需加载
                    "useBuiltIns": "usage"
                  }
                ]
              ]
            }
          },
          'ts-loader'
        ],
        exclude: /node-modules/
      }
    ]
  }
  。。。
}
```

## 模块化

当新建一个ts文件，并在入口ts文件中引入它时，webpack会报错，则需要在webpack.config.js添加以下配置

```js
module.exports = {
  ...
  resolve: {
    extensions: ['.ts', '.js']
  }
}
```

