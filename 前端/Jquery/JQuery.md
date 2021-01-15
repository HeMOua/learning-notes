# JQuery DOM操作

## 获取内容与属性

### 获取元素内容

基本的`html`结构如下：

```html
<p><strong>我是 srtong 标签</strong>我是文本内容</p>
```

如何获取`p`标签的内容呢？这里有两种方法，要注意区分，代码如下：

```js
$("p").text();
$("p").html();
```

- `text()`是获取所选元素的文本内容，`html` 标签的内容已经转化；
- `html()`是获取所选元素的内容，包括 `html` 标签。

### 获取元素的属性

基本的`html`结构如下：

```html
<div class="box" id="wrap" name="容器" data-count="none"></div>
```

如何获取该`div`的各种属性呢？这里用`attr()`，代码如下：

```js
console.log($(".box").attr("class"));
console.log($(".box").attr("id"));
console.log($(".box").attr("name"));
console.log($(".box").attr("data-count"));
```

## 设置内容和属性

### 设置元素的内容

基本的`html`结构如下：

```html
<p class="first"></p>
<p class="second">我是第二个</p>
<input class="third" type="text"/>
```

```js
$(".first").text("<h3>我是第一个</h3>");
$(".second").html("<h3>我是第二个</h3>");
$(".third").val("我是第三个");
```

以`text()`为例，`text()`表示获取内容，`text(content)`表示设置内容

### 设置元素的属性

基本的`html`结构如下： 

```html
<div class="box" id="wrap" name="容器" data-count="none"></div>
```

如何设置该`div`的各种属性呢？这里用`attr(att, newValue)`，代码如下：

```js
$("div").attr("class","alterClass");
$("div").attr("id","alterId");
$("div").attr("name","alterName");
$("div").attr("data-count","alterDataCount");
```

上面的写法是用来设置一种属性的，可不可以设置多种属性呢？当然是可以的，上面的代码简写如下：

```js
$("div").attr({
  "class": "alterClass",
  "id": "alterId",
  "name": "alterName",
  "data-count": "alterDataCount"
});
```

## 添加元素

1.`append()，prepend()`

2.`before()，after()`

基本的`html`结构如下：

```html
<button class="btn1">添加内容 </button>
<button class="btn2">添加内容 </button>
<div class="box">
     <h3>jquery可以链式调用</h3>
</div>
```

效果如下： 

![img](img/206581)

### 元素内添加

如何给类名为`box`的元素添加内容呢？ 这里介绍两个方法：`append()`和`prepend()`，实现代码如下：

```js
$(".btn1").click(function(){
     $(".box").prepend("<p>头部添加内容</p>") 
})
$(".btn2").click(function(){
     $(".box").append("<p>尾部添加内容</p>")
})
```

![img](img/206582)

可以看出：

- `append()`在被选元素的结尾插入内容；
- `prepend()`在被选元素的头部插入内容。

### 元素外添加

需要实现的效果如下：

![img](img/206601)

如何实现上面效果呢？ 这里介绍两个方法：`before()`和`after()`，实现代码如下：

```js
$(".btn1").click(function(){
     $(".box").before("<p>头部添加内容</p>") 
})
$(".btn2").click(function(){
     $(".box").after("<p>尾部添加内容</p>")
})
```

可以看出：

- `before()`在被选元素之前插入内容；
- `after()`在被选元素之后插入内容。

注意：要和 `append()`，`prepend()`区分开来。

## 删除元素

1.`remove()`

2.`empty()`

基本的`html`结构如下：

```html
<button class="btn">删除内容</button>
<div class="box">
     <p class="com first">我是第一行</p>
     <p class="com second">我是第二行</p>
     <p class="third" id="item">我是第三行</p>
</div>
```

效果图如下：

![img](img/206584)

### remove()

需要实现的效果如下：

![img](img/206585)

如何删除类名为`.box`的元素呢？这里用`remove()`方法，实现代码如下： 

可以看出：`remove()`是删除被选元素及其子元素。

### empty()

需要实现的效果如下： 

![img](img/206586)

如何只删除类名为`.box`的**子元素**呢？这里用`empty()`方法，实现代码如下： 

```js
$(".btn").click(function(){
     $(".box").empty();
})
```

可以看出：`empty()`是只删除被选元素的子元素。

# Jquery事件

1. `click()` 事件方法；
2. `dblclick()`事件方法；
3. `mouseenter()`事件方法；
4. `mouseleave()`事件方法；
5. 其他鼠标事件

### click事件

当**单击**元素时，发生 `click` 事件。`click()` 方法触发 `click` 事件，或规定当发生 `click` 事件时运行的函数。

一次点击事件`click`是指鼠标指针悬停在元素上，点击（**按下并松开**）鼠标左键一次。

基本语法：

```js
//function 为处理函数
$(selector).click(function(){
    ......
})
```

说明:`selector`表示要操作的`dom`元素

示例：点击按钮时，按钮背景颜色变为灰色。

```html
<input type="button" value="click me" class="button">
<script>
$(".button").click(function () {
    $(this).css("background","grey");
});
</script>
```

效果图如下：

![img](img/206517)

### dblclick事件

当**双击**元素时，触发`dblclick`事件。`dblclick()`方法触发`dblclick`事件，或规定当发生`dblclick`事件时运行的函数。

一次双击事件`dblclick`是指鼠标指针悬停在元素上，快速连续点击鼠标左键**两次**。

基本语法：

```js
//function 为处理函数
$(selector).dblclick(function(){
    ......
});
```

示例：双击按钮时，弹出框显示“鼠标双击事件`dblclick()`”。

```html
<input type="button" value="dblclick me" class="button">
$(".button").dblclick( function(){
    alert("鼠标双击事件dblclick()");
};
```

效果图如下：

![img](img/206518)

### mouseenter事件

当鼠标**指针穿过（进入）**被选元素时，会发生 `mouseenter` 事件。`mouseenter()` 方法触发 `mouseenter` 事件，或添加当发生 `mouseenter` 事件时运行的函数。

基本语法：

```js
//function 为处理函数
$(selector).mouseenter(function(){
    ......
})
```

示例：当鼠标移入`btn`时，`btn`背景颜色变为蓝色。

```html
<input type="button" value="mouseenter me" class="button">
$(".button").mouseenter(function () {
    $(this).css("background","blue");
});
```

效果图如下：

![img](img/206522)

### mouseleave事件

当鼠标**指针离开**被选元素时，会发生`mouseleave`事件。`mouseleave()` 方法触发 `mouseleave` 事件，或添加当发生 `mouseleave` 事件时运行的函数。

基本语法：

```
//function 为处理函数
$(selector).mouseleave(function(){
    ......
})
```

示例：当鼠标移出`btn`时，`btn`背景颜色变为绿色。

```
<input type="button" value="mouseleave me" class="button">
$(".button").mouseleave(function () {
    $(this).css("background","green");
});
```

效果图如下：

![img](img/206521)

### 其他鼠标事件方法

- `mouseover()`当鼠标指针位于**元素上方**时触发。与 `mouseenter` 事件不同，`mouseover`事件在鼠标指针**进入被选元素**或**任意子元素**时都会被触发；
- `mouseout()`当鼠标指针**离开**被选元素时触发。与 `mouseleave` 事件不同，`mouseout` 事件在鼠标指针**离开被选元素**或**任意子元素**时都会被触发；
- `mousedown()`当鼠标**按下时**触发，如果一直按着鼠标不放，只触发一次`mousedown`事件；
- `mouseup()`当鼠标**松开**按键时触发；
- `hover()`方法规定当鼠标指针**悬停**到**移开**过程中触发 `mouseenter` 和 `mouseleave` 事件。

