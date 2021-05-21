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

#### 驻留机制

仅保存一份相同且不可变字符串的方法，不同的值被存放在字符串的驻留池中， Python的驻留机制对相同的字符串只保留一份拷贝，后续创建相同字符串时，不会开辟新空间，而是把该字符串的地址赋给新创建的变量

+ 驻留机制的几种情况（交互模式）
  + 字符串的长度为0或1时
  + 符合标识符的字符串，如`abc%`就不符合标识符命名规则，所以不会驻留
  + 字符串只在编译时进行驻留，而非运行时
  + [-5, 256]之间的整数数字

+ sys中的intern方法强制2个字符串指向同一个对象

+ PyCharm对字符串进行了优化处理

  ```python
  a = 'abc'
  b = 'abc'
  print(a, id(a)) # abc 2251299979824
  print(b, id(b)) # abc 2251299979824
  
  a = 'abc%'
  b = 'abc%'
  print(a, id(a)) # abc 2251336056688
  print(b, id(b)) # abc 2251336064560 因为不满足命名规则，所以不会驻留
  
  # 不过可以使用 sys 的 intern 方法强制指向同一个对象
  a = 'abc%'
  b = 'abc%'
  a = sys.intern(b)
  a is b # True
  
  a = 'abc'
  b = 'ab' + 'c'
  a is b # True, 因为编译时 a 和 b 都是‘abc’，所以符合驻留机制
  
  a = 'abc'
  b = ''.join(['ab', 'c'])
  a is b # False, 上面是运行时拼接的，所以不符合驻留机制
  ```

+ 驻留机制的优缺点

  + 当需要值相同的字符串时，可以直接从字符串池里拿来使用，避免频繁的创建和销毁，提升效率和节约内存，因此拼接字符串和修改字符串是会比较影响性能的。
  + 在需要进行字符串拼接时建议使用str类型的`join`方法，而非`+` ，因为`join()`方法是先计算出所有字符中的长度，然后再拷贝，只new一次对象，效率比`+`高

#### 查找方法

| 方法名称   | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| `index()`  | 查找子串substr第一次出现的位置,如果查找的子串不存在时，则抛出ValueError |
| `rindex()` | 查找子串substr最后一次出现的位置，如果查找的子串不存在时，则抛出ValueError |
| `find()`   | 查找子串substr第一次出现的位置,如果查找的子串不存在时，则返回-1 |
| `rfind()`  | 查找子串substr最后一次出现的位置,如果查找的子串不存在时，则返回-1 |

#### 大小写转化

| 方法名称       | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| `upper()`      | 把字符串中所有字符都转成大写字母                             |
| `lower()`      | 把字符串中所有字符都转成小写字母                             |
| `swapcase()`   | 把字符串中所有大写字母转成小写字母，把所有小写字母都转成大写字母 |
| `capitalize()` | 把第一个字符转换为大写，把其余字符转换为小写                 |
| `title()`      | 把每个单词的第一个字符转换为大写，把每个单词的剩余字符转换为小写 |

#### 内容对齐

| 方法名称   | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| `center()` | 居中对齐，第1个参数指定宽度，第2个参数指定填充符,第2个参数是可选的，默认是空格,如果设置宽度小于实际宽度则则返回原字符串 |
| `ljust()`  | 左对齐，第1个参数指定宽度，第2个参数指定填充符,第2个参数是可选的，默认是空格如果设置宽度小于实际宽度则则返回原字符串 |
| `rjust()`  | 右对齐,第1个参数指定宽度，第2个参数指定填充符,第2个参数是可选的，默认是空格如果设置宽度小于实际宽度则则返回原字符串 |
| `zfill()`  | 右对齐，左边用0填充,该方法只接收一个参数，用于指定字符串的宽度，如果指定的宽度小于等于字符串的长度，返回字符串本身 |

#### 分割操作

| 方法名称   | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| `split()`  | 1）从字符串的左边开始劈分，默认的劈分字符是空格字符患,返回的值都是一个列表<br />2）以通过参数sep指定劈分字符串是的劈分符<br />3）通过参数maxsplit指定劈分字符串时的最大劈分次数，在经过最大次劈分之后，剩余的子串会单独做为一部分 |
| `rsplit()` | 1）从字符串的右边开始劈分，默认的劈分字符是空格字符串,返回的值都是一个列表<br />2）以通过参数sep指定劈分字符串是的劈分符<br />3）通过参数maxsplit指定劈分字符串时的最大劈分次数，在经过最大次劈分之后，剩余的子串会单独做为一部分 |

```python
s1 = 'hello|world|python'
print(s1.split(sep='|')) # ['hello', 'world', 'python']
print(s1.split(sep='|', maxsplit=2))  # ['hello', 'world|python']
```

#### 判断操作

| 方法名称         | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| `isidentifier()` | 判断指定的字符串是不是合法的标识符                           |
| `isspace()`      | 判断指定的字符串是否全部由空白字符组成(回车、换行，水平制表符) |
| `isalpha()`      | 判断指定的字符串是否全部由字母组成                           |
| `isdecimal()`    | 判断指定字符串是否全部由十进制的数字组成                     |
| `isnumeric()`    | 判断指定的字符串是否全部由数字组成                           |
| `isalnum()`      | 判断指定字符串是否全部由字母和数字组成                       |

#### 其他操作

| 方法名称    | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| `replace()` | 第1个参数指定被替换的子串，第2个参数指定替换子串的字符串,该方法返回替换后得到的字符串，替换前的字符串不发生变化,调用该方法时可以通过第3个参数指定最大替换次数 |
| `join()`    | 将列表或元组中的字符串合并成一个字符串                       |

```python
lst = ['hello', 'world', 'python']
'|'.join(lst) # 'hello|world|python'
```

#### 比较操作

+ 运算符：>, >=, <, <=, ==， ！=
+ 比较规则：首先比较两个字符串中的第一个字符，如果相等则继续比较下一个字符，依次比较下去，直到两个字符串中的字符不相等时，其比较结果就是两个字符串的比较结果，两个字符串中的所有后续字符将不再被比较
+ 比较原理：比较原理:两上字符进行比较时，比较的是其ordinal value(原始值)调用内置函数ord可以得到指定字符的ordinal value。与内置函数ord对应的是内置函数chr调用内置函数chr时指定ordinalvalue可以得到其对应的字符

#### 格式化字符串

**方式一**：

`%`做占位符

```python
name = 'lisi'
age = 21
s = '我的名字叫%s, 今年%d岁了' % (name, age)
print(s) # '我的名字叫lisi, 今年21岁了'
```

**方式二**：

`{}`做占位符

```python
name = 'lisi'
age = 21
s = '我的名字叫{0}, 今年{1}岁了,我真的叫{0}'.format(name, age)
print(s) # '我的名字叫lisi, 今年21岁了,我真的叫lisi'
```

**方法三**：

```python
name = 'lisi'
age = 21
s = f'我的名字叫{name}, 今年{age}岁了'
print(s) # '我的名字叫lisi, 今年21岁了'
```

注意字符串前的`f`

**格式**：

下面两个含义一样，基本上和c语言差不多

```python
print('{0:.3}'.format(3.1415926)) # 表示共3位数
print('{0:.3f}'.format(3.1415926)) # 表示共3位小数
print('{0:10.3f}'.format(3.1415926)) # 表示宽度10，共3位小数
print('%.3f' % 3.1415926)
```

#### 编码解码

```python
s = '我靠无情'
# 编码
print(s.encode(encoding='GBK'))
print(s.encode(encoding='UTF-8'))

# 解码
byte = s.encode(encoding='GBK')
print(byte.decode(encoding='UTF-8'))
```

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
    
  + 创建空列表

    ```python
    lst = []
    lst = list()
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
    ```
    
  + 使用内置函数 `dict()`

    ```python
    user = dict(name='jack', age=20)
    print(user) # {'name': 'jack', 'age': 20}
    ```
    
  + 创建空字典

    ```python
    d = {}
    d = dict()
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


### Tuple 元素

Python内置的数据结构之一，是一个不可变序列

> + 不变可变序：字符串、元组
>   + 不变可变序列：没有增、删，改的操作
> + 可变序列：列表、字典
>   + 可变序列：可以对序列执行增、删、改操作，对象地址不发生更改
> + 为什么要将元组设计成不可变序列
>   + 在多任务环境下，同时操作对象时不需要加锁
>   + 因此，在程序中尽量使用不可变序列
> + 注意事项：元组中存储的是对象的引用
>   + 如果元组中对象本身不可对象，则不能再引用其它对象
>   + 如果元组中的对象是可变对象，则可变对象的引用不允许改变，但数据可以改变

+ 创建方式

  + 直接用小括号，`t = ('python', 'hello', 99)`

  + 使用内置函数`tuple([arg])`，参数可以是个可迭代对象

    ```python
    t = tuple([1, 2]) # (1, 2)
    print(tuple(range(1, 10))) # (1, 2, 3, 4, 5, 6, 7, 8, 9)
    ```

  + 只包含一个元组的元素需要使用逗号和小括号，`t = (10,)`

  + 创建空元组

    ```python
    t = ()
    t = tuple()
    ```

+ 遍历
  + 使用索引
  + 使用`for-in`
+ 没有元组生成式

### set 集合

集合元素不允许重复，集合元素无序

+ 创建方式

  + 直接用话花括号，`t = {}`

  + 使用内置函数`set()`，`t = set(range(1, 5))`

  + 创建空集合

    ```python
    t = set() # 只有这一个方法
    ```

+ 判断操作，`in` 或者 `not in`

+ 新增操作

  + `add()`，依次添加一个元素
  + `update()`，至少添加一个元素

+ 删除操作

  + `remove(dat)`，一次删除一个指定元素，如果指定元素不存在则抛出`KeyError`
  + `discard(dat)`，一次删除一个指定元素，如果指定的元素不存在**不抛出异常**
  + `pop()`，一次删除任意一个元素，不能添加参数
  + `clear()`，清空集合

+ 集合操作

  + 可以使用`==`或者`!=`判断是否相等

  + 可以调用`issubset()`判断一个集合是否是另一个集合的子集

  + 可以调用`isssuperset()`判断一个集合是否是另一个集合的超集

  + 可以调用`isdisjoint()`判断一个集合和另一个集合是否有交集

  + 获取交集

    ```python
    s1.intersection(s2)
    s1 & s2
    ```

  + 获取并集

    ```python
    s1.union(s2)
    s1 | s2
    ```

  + 获取差集

    ```python
    s1.difference(s2)
    s1 - s2
    ```

  + 获取对称差集，就是两个集合的并集去掉交集

    ```python
    s1.symmetric_difference(s2)
    s1 ^ s2
    ```

+ 集合生成式

  ```python
  {i for i in range(1, 10)}
  ```

## 函数

函数就是执行特定任和以完成特定功能的一段代码

函数的作用：

+ 复用代码
+ 隐藏实现细节
+ 提高可维护性
+ 提高可读性便干调试

### 函数的创建

```python
def 函数名([输入参数]):
    函数体
    [return xxx]
```

### 参数传递

+ 位置实参：根据形参对应的位置进行实参传递

  ```python
  def calc(a, b):
      pass
  calc(10, 20) # a = 10, b = 20
  ```

+ 关键字实参：根据形参名称进行实参传递

  ```python
  def calc(a, b):
      pass
  calc(b = 10, a = 20) # a = 20, b = 10
  ```

+ ==默认参数==：函数定义时，给形参设置默认值，只有与默认值不符的时候才需要传递实参

  ```python
  def fun(a, b = 10):
      print(a, b)
      
  # 函数的调用
  fun(100)    # a = 100, b = 10
  fun(20, 30) # a = 20, b = 30
  ```

+ ==个数可变的位置参数==

  + 定义函数时，可能无法事先确定传递的位置实参的个数时，使用可变的位置参数
  + 使用`*`定义个数可变的位置形参
  + 结果为一个**元组**
  + **只能定义一个**，不然会报错

  ```python
  def fun(*args):
      print(args)
      
  fun(10)	# (10, )
  fun(10, 20, 30) # (10, 20, 30)
  
  def fun2(*args, *args1): # 会报错，个数可变的位置参数只能定义一个
      pass
  ```

+ ==个数可变的关键字形参==

  + 定义函数时，无法事先确定传递的关键字实参的个数时，使用可变的关键字形参
  + 使用`**`定义个数可变的关键字形参
  + 结果为一个**字典**
  + **只能定义一个**，不然会报错

  ```python
  def fun(**args):
      print(args)
      
  fun(a = 10)	# {'a': 10}
  fun(a = 10, b = 20, c = 30) # {'a': 10, 'b': 20, 'c': 30}
  ```

  在一个函数的定义过程中，**既有个数可变的关键字形参，也有个数可变的位置形参**，要求，==个数可变的位置形参，放在个数可变的关键字形参之前==

### 返回值

函数返回多个值时，结果为元组

```python
def fun(num):
    odd = []
    even = []
    for i in num:
        if i % 2:
            odd.append(i)
        else:
            even.append(i)
    return odd, even
print(fun([10, 29, 34, 23, 44, 53, 55])) # ([29, 23, 53, 55], [10, 34, 44])
```

### 变量作用域

程序代码能访问该变量的区域

根据变量的有效范围可分为

+ 局部变量

  + 在函数内定义并使用的变量，只在函数内部有效，局部变量使用`global`声明，这个变量
    就会就成全局变量

    ```python
    def fun():
        global age
        age = 20
        print(age)
    
    fun()
    print(age) # 20
    ```

    

+ 全局变量

  + 函数体外定义的变量，可作用予函数内外

## Bug

### Bug常见类型

+ `SyntaxError`：语法错误
+ `IndexError`：索引越界错误
+ `ZeroDivisionError`：被0除异常
+ `KeyError`：映射中没有这个键
+ `NameError`：未申明/初始化的对象（没有属性）
+ `ValueError`：传入无效的参数
+ `BaseException`：异常基类

### 异常处理

捕获异常的顺序按照先子类后父亲类的顺序,为了避免遗漏可能出现的异常，可以在最后增加`BaseException`

```python
try:
    n1 = int(input('请输入一个整数'))
    n2 = int(input('请输入另一个整数'))
    result = n1 / n2
    print('结果为', result)
except ZeroDivisionError:
    print('除数不能为0')
except ValueError:
    print('不能将字符串转化为数字')
except BaseException as e:
    print(e)
```

还有一种异常处理方式`try...catch...else`，如果try块中没有抛出异常，则执行else块，如果try中抛出异常，则执行except块

```python
try:
    n1 = int(input('请输入一个整数'))
    n2 = int(input('请输入另一个整数'))
    result = n1 / n2
except BaseException as e:
    print(e)
else:
    print('结果为', result)
```

还有最后一种异常处理方式`try...catch...finally`，finally块无论是否发生异常都会被执行，能常用来释放try块中申请的资源

```python
try:
    n1 = int(input('请输入一个整数'))
    n2 = int(input('请输入另一个整数'))
    result = n1 / n2
except BaseException as e:
    print(e)
else:
    print('结果为', result)
finally:
    print('无论是否产生异常，总会被执行的代码')
```

### 打印异常

通过`traceback`可以打印异常

```python
import traceback
try:
	print(1/0)
except:
    traceback.print_exc()
```

## 面向对象

### 类的创建

基本语法：

```python
class 类名:
    pass
```

类的组成：

+ 类属性
+ 实例方法
+ 静态方法
+ 类方法

```python
class Student:
	navtive_place = 'beijing' # 类属性
    
    def __init__(self, name, age):
        self.name = name # name, age 为实例属性
        self.age = age
	# 实例方法
    def info(self):
        print('我的名字叫：', self.name, '年龄是:', self.age)
    # 类方法
    @classmethod
    def cm(cls):
        print('类方法')
	# 静态方法
	@staticmethod
    def sm():
        print('静态方法')
```

**调用实例方法的两种方式**：

```python
# 第一种方式
stu = Student()
stu.eat()

# 第二种方式
stu = Student()
Student.eat(stu)
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







