# Python基础

## 前导知识

### 输入输出

`print()`函数的使用

+ 输出内容

  + 数字：`print(520)`
  + 字符串：`print('hello')`
  + 含有运算符的表达式：`print(3 + 1)`

+ 输出目的地

  + 显示器

  + 文件

    ```python
    fp = open('D:/text.txt', 'a+')
    print('hello', file=fp)
    fp.close()
    ```

+ 输出形式

  + 换行

  + 不换行：使用逗号分隔，就会只输出到一行，并以==空格隔开==

    ```python
    print('hello', 'world')
    ```

`input()`函数的使用

用来接受输入内容，接受输入的内容类型为`str`



### 字符编码转换

+ `ord(c)`：返回一个单字符字符串的Unicode码位，就是将字符转为2进制编码
+ `chr(i)`：返回一个单字符的Unicode字符串，就是将二进制编码转为对应字符

### 变量

变量由三部分组成

+ 唯一标识：表示对象所存储的==内存地址==，使用内置函数`id(obj)`来获取
+ 类型：表示的是对象的数据类型，使用内置函数`type(obj)`来获取
+ 值：表示对象所存储的具体数据，使用`print(obj)`可以将值进行打印输出

### 对象的布尔值

+ Python—切皆对象，所有对象都有一个布尔值
  + `bool()`：获取对象的布尔值

+ 以下对象的布尔值为False
  + 数值0 
  + None
  + 空字符串
  + 空列表
  + 空元组
  + 空字典
  + 空集合

### 注释

1. 单行注释，以`#`开头，直到换行结束

2. 多行注释，由**一对**三引号`”“”`或者`'''`之间的代码

3. 中文编码声明注释，在文件开头加上中文声明注释，用以指定源码文件编码格式，如

   ```python
   #coding:gkb
   或者
   #coding:utf-8
   ```

   当文件保存时会以相应格式保存

## 运算符

### 算术运算符

+ `+`：加
+ `-`：减
+ `*`：乘
+ `/`：除
+ `//`：整除，若运算数一正一负，则结果向下取整
+ `%`：取余
+ `**`：幂运算，如`2**3 = 2^3 = 8` 

### 赋值运算符

+ 链式赋值

  ```python
  a = b = c = 20
  print(a, id(a))
  print(b, id(b))
  print(c, id(c))
  ```

+ 参数赋值

  + +=、-=、*=、/=、//=、%=

+ 解包赋值

  ```python
  a, b, c = 20, 30, 40
  ```

+ 交换赋值

  ```python
  a, b = 10, 20
  a, b = b, a
  ```

### 比较运算符

+ <，>, >=, <=, !=
  + 比较远算符的结果都是布尔值
+ ==
  + 对象 value 的比较
+ is、is not
  + 对象 id 的比较

### 布尔运算符

+ and、or、not
+ in、not in

### 位运算符

+ &：位与，对应数位都是1,结果数位才是1,否则为0
+ |：位或，对应数位都是0,结果数位才是0，否则为1
+ <<：左移运算符，高位溢出舍弃,低位补0
+ \>>：右移运算符，低位溢出舍弃,高位补0

### 优先级

先算乘除，再算加减，但是若有幂运算就先算幂运算。（算数运算符）

先位与，再位或，但是若有位移运算符，则先位移（位运算）

比较运算符

先 and，再 or （布尔运算符）

最后赋值运算符

**总结**：算术运算符 > 位运算符 > 比较运算符 > 布尔运算符 > 赋值运算符

## 流程控制

### 分支结构

`if else elif` 语法格式

```python
if 条件表达式1:
  条件执行体1
elif 条件表达式2:
  条件执行体2
[else:]
  条件执行体N
```

==条件表达式==：

```python
x if 判断条件 else y

a = 10
b = 20
print('a', '大于等于' if a >= b else '小于', b) # a 小于 20
```

### 循环结构

`while` 语法格式

```python
while 条件表达式1:
  条件执行体1
```

`for-in` 语法格式

```python
for 自定义的变量 in 可迭代对象:
  循环体
```

+ in：表达从(字符串、序列等)中依次取值，又称为遍历
+ 目标对象：遍历的对象必须是可迭代对象
+ 若循环体不需要访问自定义变量，可以将自定义变量写为下划线`_`



### pass语句

语句什么都不做，只是一个占位符，用在语法上需要语句的地方

+ 使用时机
  + 当没想好代码怎么写的时候可以使用
+ 使用场景
  + if 语句的条件执行体
  + for-in 语句的循环体
  + 定义函数的函数体

## 数据类型

### Number 数字

+ int：98

+ float：3.14159

  ```python
  # 浮点类型进行计算时，可能会不准确，如下
  print(1.1 + 2.2) # 3.3000000000000003
  print(1.1 + 2.1) # 3.2
  
  # 因此可以采取下面的措施来解决
  from decimal import Decimal
  print(Decimal('1.1') + Decimal('2.2')) # 3.3
  ```

+ bool：True、False

  ```python
  f1 = True
  f2 = False
  
  # 布尔值可以转成整数运算
  print(f1 + 1) # 2
  print(f2 + 1) # 1
  ```

+ complex：

> 进制转换：
>
> + 十进制：无前缀
> + 二进制：前缀为`0b`，如`0b10101111`
> + 八进制：前缀为`0o`
> + 十六进制：前缀为`0x`

### String 字符串

字符串又称为不可变的字符序列，只要加上`'`或者`"`的字符串就是 str 类型，而`‘’‘`或者`“”“`可多行书写，并保留`/n`。注意，str 类型是==可迭代对象==

```
str1 = 'hello world!'
str2 = "hello world!"
str3 = """hello
world!"""

str4 = '''hello
world!'''
```

> 类型转换
>
> ```python
> name = '张三'
> age = 20
> 
> print('我叫' + name + '今年' + age + '岁') # 当将str类型与int类型进行连接时，会报错，但是用逗号连接就不会报错
> ```
>
> + 使用`str()`可以将其他数据类型转为 string 类型
>
> ```python
> a = 10
> b = 198.8
> c = False
> print(str(a), str(b), str(c))
> ```
>
> + 使用`int()`可以将其他数据类型转为 int 类型
>
> ```python
> s1 = '128' 
> f1 = 98.7
> s2 = '76.77' # 转化会报错，因为是浮点数串
> ff = True
> s3 = 'hello'
> 
> print(type(s1)，type(fl)，type(s2)，type(ff)，type(s3))
> print(int(s1)，type(int(s1))) # 将str转成int类型，字符串为数字串
> print(int(f1)，type(int(f1))) # float转成int类型，截取整数部分，舍掉小数部分
> print(int(s2)，type(int(s2))) # 将str转成int类型，报错，因为字符串为小数串
> print(int(ff)，type(int(ff))) # 输出 1 <class 'int'>
> print(int(s3), type(int(s3))) # 报错，将str转为int类型时，必须是整数串，不然会报错
> ```
>
> + 使用`float()`可以将其他数据类型转成 float 类型
>
> ```
> s1 = '128' 
> print(float(s1)) # 输出 128.0
> ```

### list 列表

变量可以存储一个元素，而列表是一个“大容器”可以存储N多个元素，程序可以方便地对这些数据进行整体操作列表相当于其它语言中的数组

+ 列表的创建

  + 使用中括号`[]`， 元素之间用英文的逗号进行分隔

  + 调用内置函数`list()`

    ```python
    # 方法一
    lst = ['hello', 'world', 98]
    print(lst) # ['hello', 'world', 98]
    
    # 方法二
    lst2 = list(['hello', 'world', 98])
    ```

+ 列表的特点

  + 列表元素按顺序有序排序
  + 索引映射唯一一个数据，如 `lst[0] = 'hello'`，`lst[-3] = 'hello'`
  + 列表可以存储重复数据
  + 任意数据类型混存
  + 根据需要动态分配和回收内存，不用考虑存放数据过多的问题

+ 常用方法

  + 查询操作

    + `index(find, start[, stop])`：获取某元素的索引
      + 如查列表中存在N个相同元素，只返回相同元素中的第一个元素的索引
      + 如果查询的元素列表中不存在，则会抛出`ValueError`
      + 可以在指定的`start`和`stop`中查找，左闭右开区间
    + `lst[index]`：获取指定索引值的元素值
      + 正向索引从`0`到`N-1`，举例`lst[0]`
      + 逆向索引从`-N`到`-1`，举例`lst[-N]`
      + 指定索引不存在，会抛出`IndexError`

  + 切片操作，语法格式 `列表名[start: stop: step]`，==切片结果为源列表片段的一个拷贝==

    + 切片范围：`[start, stop)`
    + 步长默认为1：`list1[1:3]`   把第二个元素到第三个元素个提出来，
    + step 为正数
      + `[:stop:step]`：切片的第一个元素默认是列表的第一个元素
      + `[start::step]`：切片的最后一个元素默认是列表的最后一个元素
    + step 为负数
      + `[:stop:step]`：切片的第一个元素默认是列表的最后一个元素
      + `[start::step]`：切片的最后一个元素默认是列表的第一个元素
    + `list1[:]`提取所有的元素

  + 比较操作，可以用`is`还有`not in`判断元素是否在列表中

  + 增删改操作

    + `append()`向列表尾部添加元素，并不会创建新的对象

    + `extend()`向列表末尾至少添加一个元素

    + `insert(index, dat)`添加元素，第一个参数为要添加的索引值，第二个为要添加的元素

    + `remove(dat)`参数为需要删除的元素，如果有重复元素，只移除重复元素的第一个

    + `pop(index)`删除最后一个元素，并且返回删除的元素，如果带上参数，则移除指定索引元素，若指定索引元素不存在，则抛出`IndexError`

    + `clear()`清空列表

    + `del list1`参数为需要删除的列表

    + 切片修改元素

      ```python
      # 修改
      list1 = [1, 2, 3, 4]
      list1[1:] = [True, False, 'hello']
      print(list1) # [1, True, False, 'hello']
      
      # 删除
      list1[1:3] = []
      print(list1) # [1, 'hello']
      ```

  + 乘操作、加操作

    ```python
    list1 = [1, 'hello']
    print(list1 * 2) # [1, 'hello', 1, 'hello']
    print(list1 + list) # [1, 'hello', 1, 'hello']
    ```

  + 排序

    + 修改原列表

      ``` python
      # 升序
      list1 = [20, 40, 10, 92, 51]
      print(list1, id(list1)) # [20, 40, 10, 92, 51] 2791287105288
      
      list1.sort()
      print(list1, id(list1)) # [10, 20, 40, 51, 92] 2791287105288
      
      # 降序
      list1.sort(reverse=True)
      print(list1) # [92, 51, 40, 20, 10]
      ```

    + `sorted()`不修改原列表，产生新列表

      ```python
      # 升序
      list1 = [20, 40, 10, 92, 51]
      new_list = sorted(list1)
      print(list1, new_list) # [20, 40, 10, 92, 51] [10, 20, 40, 51, 92]
      
      # 降序
      new_list = sorted(list1, reverse=True)
      ```

  + ==列表生成式==

    ```python
    list1 = [i for i in range(1, 10)]
    print(list1) # [1, 2, 3, 4, 5, 6, 7, 8, 9]
    
    list1 = [i*i for i in range(1, 10)]
    print(list1) # [1, 4, 9, 16, 25, 36, 49, 64, 81]
    ```


### dict 字典

Python内置的数据结构之一，与列表一样是一个可变序列以键值对的方式存储数据，字典是一个**无序**的序列

字典的特点：

1. 字典中的所有元素都是一个key-value对， key不允许重复, value可以重复
2. 字典中的元素是无序的
3. 字典中的key必须是不可变对象
4. 字典也可以根据需要动态地伸缩 
5. 字典会浪费较大的内存，是一种使用空间换时间的数据结构

+ 字典的创建

  + 最常用的方式：

    ```python
    scores = {'zhangsan': 100, 'lisi': 90, 'wangwu': 45}
    
    # 创建空字典
    d = {}
    ```

  + 使用内置函数 `dict()`

    ```python
    user = dict(name='jack', age=20)
    print(user) # {'name': 'jack', 'age': 20}
    ```

+ 元素获取

  + `dict[key]`，例如`scores['zhansan']`，若指定 key 不存在，则抛出`KeyError`异常
  + `get(key[, defaultValue])`方法，例如`scores.get('zhangsan')`，若指定 key 不存在，会返回 Node，也可以设置参数返回默认的 value 值

+ 判断操作

  + 使用`in`或`not in`判断键是否在字典中存在

+ 增删改操作

  + 添加修改字典元素：`scores['lisi'] = 30`
  + 删除指定键值对：`del scores['张三']`
  + 清空字典元素：`scores.clear()`

+ 视图操作

  + `keys()`：获取字典中所有 key，返回值为`dict_keys`类型，但可以通过`list()`函数转为 list 类型
  + `values()`：获取字典中所有 value，返回值为`dict_values`类型
  + `items()`：获取字典中所有 key-value 对，返回值为`dict_items`类型

+ 字典元素遍历

  ```python
  for item in scores: # item 为字典的键
      print(item)
  ```

+ 字典生成式

  + 使用内置函数`zip()`，用于将可迭代的对象作为参数，将对象中对应的元素打包成一个元组，然后返回由这些元组组成的列表

    ```python
    # 语法格式，若zip的两个参数长度不相同，则以短的为基准生成
    # { item : price for item, price in zip(items, prices) }
    name = ['cc', 'hemou']
    age = ['99', '20']
    {n: a for n, a in zip(name, age)} # {'cc': '99', 'hemou': '20'}
    ```

    

## 拓展

### 常用内建函数

#### range()

+ `range(stop)`：创建一个 (0, stop) 之间的整数序列，步长为1，左闭右开，后同
+ `range(start, stop)`：创建一个 (start, stop) 之间的整数序列，步长为1
+ `range(start, stop, step)`：创建一个 (start, stop) 之间的整数序列，步长为 step

返回值是一个迭代器对象

优点：

1、不管range对象表示的整数序列有多长，所有range对象占用的内存空间都是相同的，因为仅仅需要存储start,stop和step，只有当用到range对象时，才会去计算序列中的相关元素

2、`in`与`not in`可以判断整数序列中是否存在(不存在)指定的整数

```python
r = range(10)
print(list(r)) # [2, 3, 4, 5, 6, 7, 8, 9]
```



### import用法

| 方式                        | 引用方式                    |
| --------------------------- | --------------------------- |
| import <库名>               | <库名>.<函数名>(<函数参数>) |
| from <库名> import <函数名> | <函数名>(<函数参数>)        |
| from <库名> import *        | <函数名>(<函数参数>)        |
| import <库名> as 别名       | <别名>.<函数名>(<函数参数>) |

### turtle库

画布以窗体正中心为(0, 0)

| 函数                                 | 解释                           |
| ------------------------------------ | ------------------------------ |
| setup(width, height, startx, starty) | 绘制窗体                       |
| goto(x, y)                           | 绝对移动                       |
| fd(d)                                | 向前运动                       |
| bk(d)                                | 向后运动                       |
| circle(r, angle)                     | 以画笔左侧的一个点进行曲线运动 |
| seth(angle)                          | 设置画笔偏移角度               |
| left(angle)                          | 设置画笔向左偏移角度           |
| right(angle)                         | 设置画笔向左偏移角度           |
| penup                                | 抬起画笔                       |
| pendown()                            | 落下画笔                       |
| pencolor()                           | 画笔颜色                       |
| done()                               | 停止绘画，并保持窗体不关闭     |







