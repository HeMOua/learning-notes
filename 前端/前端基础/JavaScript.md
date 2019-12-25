# JavaScript

## Array对象

+ find
+ findInex
+ forEach
+ every
+ some
+ includes
+ map
+ reduce

### 数组转字符串

1、`String(arr)`

将arr中每个元素转为字符串，用逗号分隔

2、`arr.join("连接符")`

### 拼接和选取

1、`concat()`

拼接两个或更多个数组

2、`slice(i, n)`

返回现有数组的一个子数组

### 修改数组

`arr.splice(i, n)`

删除arr中i位置开始的n个元素

![image-20200309153730226](img/image-20200309153730226.png)

![image-20200309153843062](img/image-20200309153843062.png)

![image-20200309153903872](img/image-20200309153903872.png)

![image-20200309153921042](img/image-20200309153921042.png)

## Object对象

### Object.assign()

**语法**

```
Object.assign(target, ...sources)
```

如果目标对象中的属性具有相同的键，则属性将被源对象中的属性覆盖。后面的源对象的属性将类似地覆盖前面的源对象的属性。

**参数**

+ `target`：目标对象
+ `sources`：源对象

## DOM操作

### DOM查找

![image-20200309154457801](img/image-20200309154457801.png)

![image-20200309154436594](img/image-20200309154436594.png)

![image-20200309154546831](img/image-20200309154546831.png)

![image-20200309154603829](img/image-20200309154603829.png)

![image-20200309154631560](img/image-20200309154631560.png)

### DOM修改

![image-20200311081846792](img/image-20200311081846792.png)

![image-20200311082050245](img/image-20200311082050245.png)

![image-20200311082518706](img/image-20200311082518706.png)

![image-20200311084333838](img/image-20200311084333838.png)

![image-20200311084439399](img/image-20200311084439399.png)

![image-20200311084506125](img/image-20200311084506125.png)

### DOM添加

步骤

1. 创建空元素
2. 设置关键属性
3. 将元素添加到DOM树中



![image-20200311091901639](img/image-20200311091901639.png)
