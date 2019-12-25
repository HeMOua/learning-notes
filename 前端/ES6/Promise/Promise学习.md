# Promise学习

> ES6中一个非常重要和好用的特性就是Promise，它是异步编程的一种解决方案。一种很常见的场景应该就是网络请求了。

## 1、简单案例

Promise 本身接受一个函数 `function(resolve, reject)`作为参数，而接受的这个匿名函数的两个参数 `resolve`和 `reject` 本身也是一个函数

```js
new Promise((resolve, reject) => {
    console.log('request1')
    setTimeout(() => {
        resolve()
    }, 1000)            
}).then(() => {
    console.log('handle1')
    return new Promise((resolve, reject) => {
        console.log('request2');
        resolve()
    })
}).then(() => {
    console.log('handle2')
}).catch(data => {
    console.log(data)
})
```

## 2、catch的另一种写法

正常情况下我们应该这么写捕获 reject

```js
new Promise((resolve, reject) => {
    console.log('request1')
    setTimeout(() => {
        reject('error message')
    }, 1000)            
}).then(data => {

}).catch(err => {
    console.log(err);
})
```

而另一种就是下面的方法

```js
new Promise((resolve, reject) => {
    console.log('request1')
    setTimeout(() => {
        reject('error message')
    }, 1000)            
}).then(data => {

}, err => {
    console.log(err);
})
```

输出结果

```
request1
error message
```

## 3、简写

### 3.1 什么时候可以简写

现在回头看开头的简单案例，发现只有第一个 Promise 是异步请求，而剩下的几个都是获取前一步的操作结果，面对这种情况就可以简写了，如下

```js
new Promise((resolve, reject) => {
    console.log('request1')
    setTimeout(() => {
        resolve('data')
    }, 50)
}).then(data => {
    console.log('handle1: data =', data);
    return Promise.resolve(data + '111')
}).then(data => {
    console.log('handle2: data =', data);
    return Promise.reject('err msg')
}).then(data => {
    console.log('handle3: data =', data);
    return Promise.resolve(data + '333')
}).catch(res => {
    console.log(res)
})
```

输出结果

```
request1
handle1: data = data
handle2: data = data111
err msg
```

当不进行异步操作，而直接传递数据时可以直接用 `Promise.resolve()` 或 `Promise.reject()`传递数据，就不用再麻烦的封装 `new Promise((resolve.reject))`了

### 3.2 更进一步简写

对于`Promise.resolve()`还有更进一步的简写，如下

```js
new Promise((resolve, reject) => {
    console.log('request1')
    setTimeout(() => {
        resolve('data')
    }, 50)            
}).then(data => {
    console.log('handle1: data =', data);
    return data + '111'
}).then(data => {
    console.log('handle2: data =', data);
    return data + '222'
}).then(data => {
    console.log('handle3: data =', data);
    throw 'err msg'
}).catch(err => {
    console.log(err);
})
```

输出结果

```
request1
handle1: data = data
handle2: data = data111
handle3: data = data111222
err msg
```

直接返回即可，无需用 `Promise.resolve()`封装。而对于 `Promise.reject()` ，可以用 `throw`将错误信息抛出

## 4、Promise.all(iterator)

现在有这样的一个需求，就是当 url1 请求和 url2 请求全部完成后，才能进行操作

### 4.1 普通写法

```js
var res1 = false
var res2 = false

$.ajax({
    url: 'url1',
    success: function(res){
		res1 = true
        handleResult(res)
    }
})
$.ajax({
    url: 'url2',
    success: function(res){
		res2 = true
        handleResult(res)
    }
})

function handleResult(res){
    if(res1 && res2){
        ....
    }
}
```

这样虽然可行，但是还需定义2个变量，1个函数，十分的别扭

### 4.2 Promise.all() 写法

```js
Promise.all([
    new Promise((resolve, reject) => {
        $.ajax({
            url: 'url1',
            success: function(res){
                resolve(res) // 设res为 {name: 'bob', age: 18}
            }
        })
    }), 
    new Promise((resolve, reject) => {
        $.ajax({
            url: 'url2',
            success: function(res){
                resovle(res)// 设res为 {name: 'tim', age: 22}
            }
        })
    })
]).then(result => {
    console.log(result)
})
```

那么可以得到下面的结果

```json
 [{name: 'bob', age: 18}, {name: 'tim', age: 22}]
```

