# js添加和删除class

下面介绍一下如何给一个节点添加和删除class名

> 添加：节点.classList.add("类名")；
>
> 删除：节点.classList.remove("类名")；

以tab切换为例：

在写tab切换的时候，通常我们会给选中的tab设置不同的样式，常用的方法是给被选中的tab新增一个class名，然后改这个class名的样式。

比如 起一个class名叫“active”

设置样式

```
.active{
    color: #FFD113 !important;
}
```

在html代码中给首页（默认选中）加上class名active

```
<a class="tab_item active">
    <span class="iconfont icon-shouye"></span>
    <span class="tab2">首页</span>
</a>
```

在点击切换的过程中，我们需要给被选中的子选项添加“active”，然后让其它子选项删除“active”名。

该怎么做呢？

首先，通过JS取到所有tab的节点

```
var arr = document.getElementsByClassName("tab_item");
```

然后只需在被选中的子节点加上.classList.add("类名")，比如：

```
arr[i].classList.add("active");
```

这样就给当前子选项卡添加了“active”类名。

然后我们把其他的选项卡的“active”移除，只需要在其他子节点的后面加上.classList.remove("类名")，比如：

```
arr[j].classList.remove("active");
```

这样就实现了我们想要的功能。