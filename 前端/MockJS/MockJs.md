# MockJs

> + mockJs生成随机数据语法
>
> + mockJs拦截请求
> + [Mock文档](https://github.com/nuysoft/Mock/wiki)、[Mock示例](http://mockjs.com/examples.html)

## 为什么使用MockJS

在工作开发中，如果后端接口还未开发完成，难道我们就只能写静态页面了吗？所以前端为了不影响工作效率，我们自己手动模拟后端接口返回随机数据。

1. 采用json数据模拟，生成数据比较繁琐，也比较有局限性，没办法达到增删改查
2. 采用mockjs进行模拟数据，可以模拟各种场景 [get、post] 生成接口，并且随机生成所需数据，还可以对数据进行增删改查

## 使用MockJS

先通过vue-cli创建项目

1. 在项目中安装mock

   ```shell
   npm i mockjs
   ```

2. 在项目中新建Mock模块

   ```js
   import Mock from ‘mockjs’
   ```

3. 将mock文件在main.js中导入

   ```js
   import Vue from 'vue'
   import App from './App.vue'
   import './mock/index'
   ...
   ```

## MockJS语法

### 生成字符串

1. 指定范围

`'name|min-max': string`

```js
Mock.mock({
  "string|1-10": "★"
})
```

Result

```
{
  "string": "★★★★★★★★"
}
```

2. 指定重复次数

`'name|count': string`

```js
Mock.mock({
  "string|3": "★★★"
})
```

Result

```
{
  "string": "★★★★★★★★★"
}
```

3. 生成随机文本

`'name': '@cword'`

```js
Mock.mock({
    'string': '@cword'
})
```

4. 指定随机文本长度

`'name': '@cword(len)'`

`'name': '@cword(min, max)'`

```js
Mock.mock({
	'username': '@cword(5)',
    'title': '@cword(5, 10)'
})
```

5. 生成标题和句子

`'name': '@ctitle(len)'`

`'name': '@csentence(len)'`

```js
Mock.mock({
    'title': '@ctitle(8)',
    'sentence': '@csentence(50)'
})
```

6. 生成段落

```js
Mock.mock({
	'content': '@cparagraph(5)'
})
```

### 生成数字

1. 生成指定数字

```js
Mock.mock({
	'number|80': 1
})
// 或者
Mock.mock({
    'age': 20
})
```

2. 生成指定范围数字

```js
Mock.mock({
    'age|20-30': 1
})
```

### 生成增量id

```js
Mock.mock({
    id: '@increment'
})
```

### 生成姓名-地址-身份证号

```js
Mock.mock({
	name: '@cname()',
    idCard: '@id()',
    address: '@city(true)' // 带true生成的更具体
})
```

### 生成图片

`@image([size], [backcolor], [forecolor], [format], [text])`

+ size：图片大小，如 200x300
+ backcolor：背景色
+ forecolor：前景色
+ format：图片格式
+ text：图片文字

```js
Mock.mock({
	img_url: '@image("300x300", "#00a1d6", "#fff", "png", "登录")'
})
```

其实原理就是借助这个网站[Dynamic Dummy Image Generator - DummyImage.com](https://dummyimage.com/)

### 生成时间

```js
Mock.mock({
	date: '@date()'
})

Mock.mock({
    date: '@date(yyyy:MM:dd HH:mm:ss)'
})
```

### 生成数组

```js
Mock.mock({
    'list|8': [
        {
            name: '@cname',
            address: '@city',
            id: '@increment'
        }
    ]
})
```

## 模拟请求

### get

```js
Mock.mock('/api/news', 'get', {
    status: 200,
    msg: '获取数据成功'
})
```

### post

```js
Mock.mock('/api/news', 'post', {
	status: 200,
    msg: '获取数据成功'
})
```

