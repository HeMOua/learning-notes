# TypeScript笔记

## 基本概念

+ 以JavaScript为基础构建的语言
+ 一个JavaScript的超集
+ TypeScript拓展了JavaScript，并添加了类型，支持ES的新特性，添加ES不具备的新特性，丰富的配置选项
+ 可以在任何支持JavaScript的平台中执行
+ ts不能被js解析器直接执行，需要经过编译转换成javascript

## 环境搭建

1. 下载安装Node.js
2. npm全局安装 typescript `npm i -g typescript`
3. 命令行输入 `tsc`，可以校验是否安装成功
4. 创建 ts 文件，如`1.ts`
5. 命令行中输入 `tsc 1.ts`即可完成ts->js的转化

## 基本类型

+ 类型声明

  + 类型声明是TS非常重要的一个特点

  + 通过类型声明可以指定TS中变量（参数、形参）的类型

  + 指定类型后，当为变量赋值时，TS编译器会自动检查值是否符合类型声明，符合则赋值，否则报错。

  + 简而言之，类型声明给变量设置了类型，使得变量只能存储某种类型的值

  + 语法:

    ```ts
    // 声明变量a，并指定他的类型为number
    let a: number;
    
    // 申明完变量直接进行赋值
    let b : boolean = true
    
    // 限制方法参数类型
    function sum(a: number, b: number) {
        return a + b
    }
    
    // 限制方法返回值类型
    function sum(a: number, b: number): number {
        return a + b
    }
    ```

+ 自动类型判断

  + TS拥有自动的类型判断机制
  + 当对变量的声明和赋值是同时进行的，TS编译器会自动判断变量的类型
  + 所以如果你的变量的**声明和赋值时同时进行的，可以省略掉类型声明**

+ 类型

| 类型    | 例子       | 描述     |
| ------- | ---------- | :------- |
| number  | 1，-33,2.5 | 任意数字 |
| string  |    'hi', "hi",hi        |     任意字符串     |
| boolean |     true、false       |    布尔值true或false      |
| 字面量  |     其本身       |     限制变量的值就是该字面量的值，类似常量     |
| any     |    *        |       任意类型   |
| unknown |    *        |    类型安全的any      |
| void    |  空值(undefined)          | 没有值（或undefined)         |
| never   |    没有值        |    不能是任何值      |
| object  |    {name:'孙悟空'}        |    任意的JS对象      |
| array   |    [1,2,3]        |  任意JS数组        |
| tuple   |     [4.5]       |    元素，TS新增类型，固定长度数组      |
| enum    |    enum{A, B}        |    枚举，TS中新增类型      |

示例代码

```ts
/** 字面量 **/
let a: 10;
a = 10;
a = 11; // 会报错

// 使用场景
let b: 'male' | 'female'
// b 只能是这两个值中的一个
b = 'male'
b = 'female'

/* any */
// any 表示任意类型，会致使ts关闭对该变量的类型检测， 不推荐使用
let d: any 	// 显式指定变量类形为any
let d:		// 隐式指定变量类型为any


/* unkown */
// 是一个类型安全的any，不能直接赋值给其他类型的变量
let e: unkown
let f: string
e = 1
f = e // 会报错

// 使用方法
if (typeof e === 'string') f = e
// 或者
f = e as string
// 或者
f = <string>e

/* void 表示空，表示没有返回值的函数 */
function fn(): void{
}


/* never 表示永远不会返回结果 */
function fun2(): never {
    throw new Error('error')
}

/* object, 限制对象只包含那些属性，以及属性类型 */
// 属性后加一个 ?，代表该属性是可选的
// [propName: string]: string，代表这个对象中可以有任意属性，属性名是字符串类型
let b: { name: string, age?: number, [propName: string]: string }

/* array */
let a: string[]； // 字符串数组
let b: Array<number>； // 数字数组


/* tuple 元组 */
let h: [string, number]
h = ['hello', 123]


/* enum 枚举 */
let i: { name: string, gender: 0 | 1 } // 不推荐

enum Gender {
    Male, Female
}

let i: { name: string, gender: Gender }
i = { name: 'lisi', gender: Gender.Male }

/* 条件同时满足 */
let j: { name: string } & { age: number }
j = { name: 'lisi', age: 18 }

/* 类型别名 */
// 需求：限制值只能在 1~5 中取
let K: 1 | 2| 3 | 4 | 5
// 上述写法虽然可以实现，但是如果又有一个变量也是同样的限制，这样写就麻烦了
type myType = 1 | 2| 3 | 4 | 5
let K: myType
let f
```

==type 可以用来定义类型==

## 编译选项

1、自动编译文件

编译文件时，使用-w指令后，TS编译器会自动监视文件的变化，并在文件发生变化时对文件进行重新编译，如下

```shell
tsc xxx.ts -w
```

2、自动编译整个项目

+ 如果直接使用tsc指令，则可以自动将当前项目下的所有ts文件编译为js文件。

+ 但是能直接使用tsc命令的前提时，要先在项目根目录下创建一个ts的配置文件`tsconfig.json`

+ `tsconfig.json`是一个JSON文件，添加配置文件后，只需只需tsc命令即可完成对整个项目的编译

+ 配置选项:

  + `include`

    + 定义希望被编译文件所在的目录，默认值：`["**/*"]`

    + 示例

      ```json
      // tsconfig.json，本示例中，所有src和test目录下的文件都会被编译
      // ** 表示任意目录
      // *  表示任意文件
      {
          "include": ["src/**/*", "test/**/*"]
      }
      ```

  + `exclude`

    + 定义不需要被编译的目录，默认值：`["node_modules", "bower_components", "jspm_packages"]`

    + 示例

      ```json
      {
          "exclude": ["node_modules"]
      }
      ```

  + `extends`

    + 定义被继承的配置文件

    + 示例

      ```json
      // 下述示例中，当前配置文件会自动包含 base.json 中的所有配置信息
      {
          "extends": "./configs/base"
      }
      ```

  + `files`

    + 指定被编译文件的列表，只有需要编译的文件少时才会用到

    + 示例

      ```json
      {
      	"files": [
              "core.ts",
              "sys.ts"
          ]
      }
      ```

    + 列表中的文件都会被TS编译器所编译

3、编译器选项配置`compilerOptions`

`compilerOptions`属于tsconfig.json文件的直属属性，下面所有的配置选项全部属于compilerOptions内的配置

+ `target`：字符串，指定ts被编译的es的版本，默认`es3`，推荐`es2015`，也就是`es6`
+ `module`：字符串，指定要使用的模块化规范，推荐`es2015`
+ `lib`：数组，指定要使用的库
+ `outDir`：字符串，指定输入的文件目录，默认为源代码所在目录，推荐目录`./dist`
+ `outFile`：字符串，将所有代码合并为一个文件，它会将全局作用域的代码整合到一个文件中
+ `allowJs`：布尔值，是否将js文件进行编译，若为false则js文件就不会输出到outDir目录，默认为`false`
+ `checkJs`：布尔值，检查js代码是否符合规范，默认`false`
+ `removeComments`：布尔值，是否移除注释，默认`false`
+ `noEmit`：布尔值，不生成编译后的文件，设置为true就不会将ts转换为js文件，默认`false`
+ `noEmitOnError`：布尔值，当有错误的时候不生成编译后的文件，默认`false`

与编译检查有关的配置

+ `strict`：布尔值，严格检查的总开关，为true时，所有的严格配置都会为true

+ `alwaysStrict`：布尔值，是否启用严格模式
+ `noImplicitAny`：布尔值，不允许隐式的any类型
+ `noImplicitThis`：布尔值，不允许不明确类型的this
+ `strictNullChecks`：布尔值，是否严格检测空值，默认false

## 结合webpack

### 简单使用

1、安装依赖

```shell
npm i -D webpack webpack-cli typescript ts-loader
```

2、配置webpack.config.js

```js
const path = require('path')

const resolve = (dir) => path.resolve(__dirname, dir)

module.exports = {
  entry: './src/app.ts',
  output: {
    path: resolve('dist'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node-modules/
      }
    ]
  }
}
```

3、tsconfig.js

```js
{
    "compilerOptions": {
        "target": "ES6",
        "module": "ES6",
        "strict": true
    }
}
```

4、package.json

添加一个指令

```json
"build": "webpack"
```

最后在命令行输入`npm run build`即可

### 引入插件

#### html-webapck-plugin

1. 安装插件

```shell
npm i -D html-webapck-plugin
```

2. 引入插件

```js
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
```

3. 应用插件并配置html模板

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



#### webpack-dev-server

1. 安装插件

```shell
npm i -D webpack-dev-server
```

2. 添加命令

```json
"start": "webpack serve --open"
```

然后运行 `npm start` 即可

#### babel

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

### 模块化

当新建一个ts文件，并在入口ts文件中引入它时，webpack会报错，则需要在webpack.config.js添加以下配置

```js
module.exports = {
  ...
  resolve: {
    extensions: ['.ts', '.js']
  }
}
```

## 面向对象

面向对象是程序中一个非常重要的思想，它被很多同学理解成了一个比较难，比较深奥的问题，其实不然。面向对象很简单，简而言之就是程序之中所有的操作都需要通过对象来完成。

举例来说:

+ 操作浏览器要使用window对象
+ 操作网页要使用document对象
+ 操作控制台要使用console对象

一切操作都要通过对象，也就是所谓的面向对象，那么对象到底是什么呢?这就要先说到程序是什么，计算机程序的本质就是对现实事物的抽象，抽象的反义词是具体，比如:照片是对一个具体的人的抽象，汽车模型是对具体汽车的抽象等等。程序也是对事物的抽象，在程序中我们可以表示一个人、一条狗、一把枪、一颗子弹等等所有的事物。一个事物到了程序中就变成了一个对象。

在程序中所有的对象都被分成了两个部分数据和功能，以人为例，人的姓名、性别、年龄、身高、体重等属于数据，人可以说话、走路、吃饭、睡觉这些属于人的功能。数据在对象中被成为属性，而功能就被称为方法。所以简而言之，在程序中一切皆是对象。

### 类的简介

```ts
class Person {
    // 直接定义的属性是实例属性，需要通过对象的实例去访问
    name: string = '孙悟空'
    
    // 通过static定义的属性为静态属性，需要通过类来访问
    static age: number = 18
    
    // readonly 开头的表示一个只读属性，无法修改
    readonly father: string = '石头'
    
    // 定义方法
    sayHello() {
    }
}
```

### 构造函数

```ts
class Person {
	name: string
    age: number
    
    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }
}
```

### 继承

```ts
class Animal {
	name: string
    age: number
    
    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }
    sayHello() {
        console.log('动物在叫~~~')
    }
}

class Cat extends Animal {
    sayHello() {
        console.log('喵喵在叫~~~')
    }
}
```

关于super关键字同java，不再介绍

### 抽象类

```ts
abstract class Animal {
    name: string
    constructor(name: string) {
        this.name = name
    }
    abstract sayHello(): void
}
```

### 接口

```ts
interface interf {
    name: string
    age: number
}

interface interf {
    gender: string
    
    sayHello(): void
}

let a: interf = {
    age: 18,
    gender: 'nan',
    name: '大黄',
    sayHello() {
        console.log('123')
    }
}

class MyClass implements interf {
    name: string
    age: number
    gender: string

    constructor(name: string, age: number, gender: string) {
        this.name = name
        this.age = age
        this.gender = gender
    }

    sayHello(): void {
        throw new Error('Method not implemented.')
    }
}
```

==与java的不同在与，接口可以多次申明，最后这个接口是所有同名接口合并的效果==

### 封装

```ts
class MyClass {
    name: string // 默认为public
    private _gender: string

    public get gender(): string {
        return this._gender
    }
    public set gender(value: string) {
        this._gender = value
    }

    constructor(name: string, gender: string) {
        this.name = name
        this._gender = gender
    }
}
```

protected关键字同java

### 泛型

```ts
function fn<T>(a: T): T {
    return a
}

fn(10) // 可以自动推断类型
fn<string>('hello') // y
```

