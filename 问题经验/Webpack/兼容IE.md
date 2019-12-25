# 兼容IE

> ie不识别箭头函数，以及es6语法

1、修改webpack配置

第一步取消掉webapck的箭头函数，因为新版本的webpack打包时默认使用的就是箭头函数，babel也处理不了，这里就要修改webpack的配置了，在webpack.config.js添加如下配置

```js
module.exports = {
  output: {
	...
    // 取消箭头函数
    environment: {
      arrowFunction: false
    }
  }
}
```



2、添加babel

安装依赖

```shell
npm i -D @babel/core @babel/preset-env babel-loader core-js
```

修改webapck配置

```js
module.exports = {
  ...
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
                      "chrome": "58",
                      "ie": "11"
                    },
                    // 指定corejs版本
                    "corejs": "3",
                    // 指定corejs的使用方式为按需加载
                    "useBuiltIns": "usage"
                  }
                ]
              ]
            }
          }
        ],
        exclude: /node-modules/
      }
    ]
  }
}
```

