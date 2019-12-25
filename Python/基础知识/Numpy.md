# Numpy

## 介绍

> Numpy（Numerical Python 的缩写）
>
> + 一个开源的Python科学计算库
> + 使用Numpy可以方便的使用数组、矩阵进行计算
> + 包含线性代数、傅里叶变换、随机数生成等大量函数

> 对于同样的数值计算任务，使用Numpy比直接编写Python代码实现，优点如下：
>
> + 代码更简洁：Numpy直接以数组、矩阵为粒度计算并且支持大量的数学函数，而Python需要用for循环
>   从底层实现
> + 性能更高效: Numpy的数组存储效率和输入输出计算性能，比Python使用List或者嵌套List好很多;
>
> 注：Numpy的数据存储和Python原生的List是不一样的
>
> 注：Numpy的大部分代码都是C语言实现的，这是Numpy比纯Python代码高效的原因;

> Numpy是Python各种数据科学类库的基础库
>
> + 比如 SciPy、Scikit-Learn、Tensorflow、PaddlePaddle等
> + 如果不会Numpy，这些库的深入理解都会遇到障碍

## Array

**array对象的背景**：

+ Numpy的核心数据结构，就叫做array就是数组,array对象可以是一维数组，也可以是多维数组;
+ Python的List也可以实现相同的功能，但是array比List的优点在于性能好、包含数组元数据信息、大量便捷函数
+ Numpy成为事实上的Scipy、Pandas、Scikit-Learn、Tensorflow、PaddlePaddle等框架的”通用底层语言”
+ Numpy的array和Python的List的一个区别，是它元素必须都是**同一种数据类型**，比如都是数字int类型，这也Numpy高性能的一个原因;

### 创建

创建数组的方法主要分为以下三类

+ 从Python的列表List和嵌套列表创建array
+ 使用预定函数arange、ones/ones_like、zeros/zeros_like、empty/empty_like、full/full_like、eye等函数创建。
+ 生成随机数的np.random模块构建

```python
# 方法一：使用 python 中的 list 来创建
import numpy as np
x1 = np.array([1, 2, 3, 4, 5, 6, 7, 8]) # 一维
x2 = np.array([							# 二维
    [1, 2, 3, 4],
    [5, 6, 7, 8]
])

# 方法二：使用 arange 创建数字序列
# 语法：arange([start,] stop [, step], dtype=None)
np.arange(10) # array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
np.arange(2, 10, 2) # array([2, 4, 6, 8])

# 方法三：使用 ones 创建全是 1 的数组
np.ones(shape, dtype=None, order='C')
# shape: 数组 或 一个元组
np.ones(10) # array([1., 1., 1., 1., 1., 1., 1., 1., 1., 1.])
np.ones((2, 3)) 
# array([[1., 1., 1.],
#       [1., 1., 1.]])

# 使用 ones_like 创建形状类似的数组
np.ones_like(list1, dtype=float, order='C')

# 方法四：创建的数组内容全为0
np.zeros(shape, dtype=Node, order='C')
np.zeros_like(list, dtype=float, order='C')

# 方法五：数据未初始化，里面的值可能是随机值
np.empty(10)
np.empty((2, 4))
np.empty_like(list1)

# 方法六：用指定值填充数组
np.full(shape, fill_value, dtype=None, order='C')
np.full(10, 666)
np.full((2, 4), 666)

np.full_like(10, 666)
np.full_like((2, 4), 666)

# 方法七：使用random模块生成随机数组
np.random.randn() # 生成一个随机数
np.random.randn(3) # 生成长度为3的nparray数组
np.random.randn(3, 2) # 生成 3行2列 的nparray数组
np.random.randn(3, 2, 4) # 生成 3块 每块2行4列的数组

np.random.randint(start, end, count)
np.random.randint(1, 100, 10) # 从1到100中随机选取10个数字
```

### 属性

+ `shape`：返回一个元组，表示array的维度
+ `ndim`：一个数字，表示array的维度的数目
+ `size`：一个数字，表示array中所有数据元素的数目
+ `dtype`：array中元素的数据类型

使用上面方法一创建的例子

```python
x1.shape # (8,)
x2.shape # (2, 4)

x1.ndim  # 1
x2.ndim  # 2

x1.size  # 8
x2.size  # 8

x1.dtype # dtype('int32')
x2.dtype # dtype('int32')
```

### 操作及函数

+ 直接逐元素的加减乘除等算数操作 
+ 更好用的面向多维的数组索引
+ 求sum/mean等聚合函数
+ 线性代数函数，比如求解逆矩阵、求解方程组
+ `astype`：创建指定类型的同型矩阵

```python
A = np.arange(10).reshape(2, 5) # 可以将长度为10的一位数组转换为 2行5列 的数组
A.shape # (2, 5)
A + 1 # 数组的每个元素+1，下面同理
A * 3
np.sin(A)
np.exp(A) 

B = np.random.randn(2, 5)
A + B # 同型矩阵相加

array1 = np.array([1., 2., 3.])
array2 = array1.astype('')
```

## 索引查询

```python
import numpy as np

# 一维向量
x1 = np.arange(10)

# 二维向量
x2 = np.arange(20).reshape(4, 5)
# array([[ 0,  1,  2,  3,  4],
#        [ 5,  6,  7,  8,  9],
#        [10, 11, 12, 13, 14],
#        [15, 16, 17, 18, 19]])
```

### 基础索引

对于**一维数组**和python的list操作一样

```python
print(x1[2], x1[5], x1[-1]) # 2 5 9
print(x1[2:4]) # 2 3
print(x1[2:-1]) # 2 3 4 5 6 7 8
```

对于**二维数组**

```python
# 筛选值
x2[0, 0] # 0
x2[-1, 2] # 17

# 筛选具体行
x2[2] # array([10, 11, 12, 13, 14])
x2[-1] # array([15, 16, 17, 18, 19])

# 筛选多行
x2[:-1]
# array([[ 0,  1,  2,  3,  4],
#        [ 5,  6,  7,  8,  9],
#        [10, 11, 12, 13, 14]])

# 筛选多行，然后筛选列
x2[:2, 2:4]
# array([[2, 3],
#        [7, 8]])

# 筛选列
x2[:, 2] # array([ 2,  7, 12, 17])
```

==注意：对切片的修改会影响原来的数组==

所以对数组进行操作时，可以先进行复制操作

```python
arr2 = arr.copy()
```

### 神奇索引

用整数数组进行索引，也就是神奇索引

**一维数组**

```python
x = np.arange(10)

# 通过索引数组获取数组的值
x[[3, 4, 7]] # array([3, 4, 7])

# 设置二维数组，获取元素组的值，返回值也是个二维数组
indexs = np.array([[0, 2], [1, 3]])
x[indexs]
# array([[0, 2],
#        [1, 3]])

# 实例：
# 1. 获取数组中最大的前n个数字
arr = np.random.randint(1, 100, 5) # array([61, 71, 24,  9, 19])

# 2. 获取最大值对应的3个下标
arr.argsort()[-3:] # array([2, 0, 1], dtype=int64)

# 3. 通过下标获取对应值
arr[arr.argsort()[-3:]] # array([24, 61, 71])
```

**二维数组**

```python
x = np.arange(20).reshape(4, 5)

# 刷选多行，列可以忽略
x[[0, 2]]

# 筛选多列，行不可忽略
x[:, [0, 2, 3]]

# 指定行列坐标，返回一维数组
x[[0, 2, 3], [1, 2, 3]]
```

### 布尔索引

**一维数组**

```python
x = np.arange(10)x>5 # array([False, False, False, False, False, False,  True,  True,  True, True])X[x>5] # array([6, 7, 8, 9])
```

**二维数组**

```python
x = np.arange(20).reshape(4, 5)x>5x[x>5] # 会返回一维数组# 筛选第3列数字大于5的行x[x[:, 3]>5]
```

### 条件组合

```python
x = np.arange(20)condition = (x % 2 == 0) & (x > 7)x[condition] # array([ 8, 10, 12, 14, 16, 18])
```

## 随机数函数

[Random sampling (numpy.random) — NumPy v1.14 Manual (scipy.org)](https://docs.scipy.org/doc/numpy-1.14.0/reference/routines.random.html)

1、随机整数

+ `np.random.random`
+ 

| 函数名                              | 说明                                                      |
| ----------------------------------- | --------------------------------------------------------- |
| seed( [seed])                       | 设定随机种子，这样每次生成的随机数会相同                  |
| rand(d0,d1, ..., dn)                | 返回数据在[0，1)之间，具有均匀分布                        |
| randn(d0,d1, ..., dn)               | 返回数据具有标准正态分布（均值0，方差1)                   |
| randint( low[ , high, size, dtype]) | 生成随机整数，包含1ow，不包含high                         |
| random( [size])                     | 生成[0.0, 1.0)的随机数                                    |
| choice(a[, size, replace，p])       | a是一维数组，从它里面生成随机结果                         |
| shuffle(×)                          | 把一个数组x进行随机排列，**会修改原数组**                 |
| permutation(x)                      | 把一个数组x进行随机排利，或者数字的全排列，**返回新数组** |
| normal( [loc, scale, size])         | 按照平均值1oc和方差scale生成高斯分布的数字                |
| uniform( [ low, high, size])        | 在[low, high)之间生成均匀分布的数字                       |

```python
import numpy as npnp.random.seed(666) # 置随机数种子
# 1. rand(d0, d1, ..., dn) 返回数据在[0，1)之间，具有均匀分布
np.random.rand(5)
# array([0.70043712, 0.84418664, 0.67651434, 0.72785806, 0.95145796])

np.random.rand(3, 4)
# array([[0.0127032 , 0.4135877 , 0.04881279, 0.09992856],
#        [0.50806631, 0.20024754, 0.74415417, 0.192892  ],
#        [0.70084475, 0.29322811, 0.77447945, 0.00510884]])

np.random.rand(2, 3, 4)

# 2. randn(d0,d1, ..., dn)  返回数据具有标准正态分布（均值0，方差1)
np.random.randn(5)
np.random.randn(3， 4)
np.random.randn(2， 3， 4)

# 3. choice(a[, size, replace，p])  a是一维数组，从它里面生成随机结果
# a是数字的话，则从range(a)中生成
np.random.choice(5, 3) # array([0, 3, 3])

# 4. shuffle(x) 将数组随机排序
a = np.arange(10)
np.random.shuffle(a)# array([3, 9, 7, 8, 1, 2, 0, 5, 6, 4])

a = np.arange(20).reshape(4, 5)
np.random.shuffle(a)
# 对于二维数组，shuffle只会对第一个维度进行打散
# array([[15, 16, 17, 18, 19],
#        [ 5,  6,  7,  8,  9],
#        [10, 11, 12, 13, 14],
#        [ 0,  1,  2,  3,  4]])
```

**实例：对数组加入随机噪声**

```python
import matplotlib.pyplot as plt

# 绘制sin曲线
x = np.linspace(-10, 10, 100) # 在 -10 到 10 随机生成 100 个点
y = np.sin(x)
plt.plot(x, y)
plt.show()

# 加入噪声
x = np.linspace(-10, 10, 100) # 在 -10 到 10 随机生成 100 个点
y = np.sin(x) + np.random.rand(len(x))
plt.plot(x, y)
plt.show()
```

## 统计函数

1.Numpy有哪些数学统计函数：

以上函数，都有一个参数叫做axis用于指定计算轴为行还是列，如果不指定，那么会计算所有元素的结果

| 函数名        | 说明                          |
| ------------- | ----------------------------- |
| np.sum        | 所有元素的和                  |
| np.prod       | 所有元素的乘积                |
| np.cumsum     | 元素的累积加和                |
| np.cumprod    | 元素的累积乘积                |
| np.min        | 最小值                        |
| np.max        | 最大值                        |
| np.percentile | 0-100百分位数                 |
| np.quantile   | 0-1分位数                     |
| np.median     | 中位数                        |
| np.average    | 加权平均，参数可以指定weights |
| np.mean       | 平均值                        |
| np.std        | 标准差                        |
| np.var        | 方差                          |

```python
arr = np.arange(12).reshape(3, 4) + 1 
# array([[ 1,  2,  3,  4],
#        [ 5,  6,  7,  8],
#        [ 9, 10, 11, 12]])

# sum
np.sum(arr) # 78

# 加权平均
weight = np.random.rand(*arr.shape) # *可以将arr.shape拆开放进参数
np.average(arr, weights=weight)
```

以上函数，都有一个参数叫做`axis`用于指定计算轴为行还是列，如果不指定，那么会计算所有元素的结果

+ `axis = 0`：代表行
+ `axis = 1`：代表列

```python
arr = np.arange(12).reshape(3, 4) + 1 
arr 
# array([[ 1,  2,  3,  4],
#        [ 5,  6,  7,  8],
#        [ 9, 10, 11, 12]])

arr.sum(axis=0) # array([15, 18, 21, 24])

arr.sum(axis=1) # array([10, 26, 42])

arr.cumsum(axis=0)
# array([[ 1,  2,  3,  4],
#        [ 6,  8, 10, 12],
#        [15, 18, 21, 24]], dtype=int32)
```

**实例：机器学习将数据进行标准化**

arr如果对应到现实世界的一种解释：

+ 行：每行对应一个样本数据
+ 列：每列代表样本的一个特征

数据标准化:

+ 对于机器学习、神经网络来说，不同列的量纲应该相同，训练收敛的更快;
+ 比如商品的价格是0到100元销量是1万到10万个，这俩数字没有可比性，因此需要先都做标准化;。不同列代表
+ 不同的特征，因此需要axis=O做计算
+ 标准化一般使用`A=(A - mean(A, axis=0)) / std(A, axis=0)`公式进行

```python
arr = np.random.randint(1, 100, size=(3, 4))
arr
# array([[94, 61, 28, 99],
#       [91, 25, 73, 73],
#       [90, 92, 66, 16]])

result = (arr - np.mean(arr, axis=0)) / np.std(arr, axis=0)
result
# array([[ 1.37281295,  0.0608761 , -1.39936232,  1.04817384],
#       [-0.39223227, -1.2540477 ,  0.87670892,  0.29810449],
#       [-0.98058068,  1.1931716 ,  0.5226534 , -1.34627832]])
```

求result时，可以发现arr和np.mean的维度不一样，但是减的时候仍然后将其每一行复制与arr进行相减，这就叫numpy的拓展

## 满足条件数

大于5000的数字数

1、使用numpy生成1亿个数字

```python
import numpy as np
arr = np.random.randint(1, 10000, size=int(1e8)) # 生成1亿个数字
arr[:10] # 前十个数字
arr.size # 数量
```

2、python原生语法判断

```python
pyarr = list(arr)
len([x for x in pyarr if x > 5000])
%timeit len([x for x in pyarr if x > 5000]) #jupyter 可以计算时间
```

3、numpy向量化判断

```python
arr[arr>5000].size
```

## 增加维度

> **背景**
>
> 很多数据计算都是二维或三维的，对于一维的数据输入为了形状匹配，经常需升维变成二维
>
> **需要**
>
> 在不改变数据的情况下，添加数组维度;(注意观察这个例子，维度变了，但数据不变)
>
> 原始数组:一维数组arr=[1,2,3,4]，其shape是(4,)，取值分别为arr[0],arr[1],arr[2],arr[3]
>
> 变形数组:二维数组arr[1,2,3.4]，其shape实(1.4),取值分别为a[0,0],a[0,1],a[0.2],a[0,3]
>
> **实操**
>
> 经常需要在纸上手绘数组的形状，来查看不同数组是否形状匹配，是否需要升维降维

### np.newaxis

关键字，使用索引的语法给数组添加维度

注意：`np.newaxis`其实就是None的别名

```python
np.newaxis is None # True
np.newaxis == None 3 True
```

即以下所有的np.newaxis的位置，都可以用None替代

1、添加行维度

```python
>>> arr = np.arange(5)
>>> arr
array([0, 1, 2, 3, 4])
>>> arr[np.newaxis, :]
array([[0, 1, 2, 3, 4]])
```

2、增加列维度

```python
>>> arr[:, np.newaxis]
array([[0],
       [1],
       [2],
       [3],
       [4]])
```

### np.expand_dims(arr, axis)

方法，和np.newaxis实现一样的功能，给arr在axis位置添加维度

```python
# 添加行维度
>>> np.expand_dims(arr, axis=0)
array([[0, 1, 2, 3, 4]])
>>> np.expand_dims(arr, axis=0).shape
(1, 5)

# 添加列维度
>>> np.expand_dims(arr, axis=1)
array([[0],
       [1],
       [2],
       [3],
       [4]])
>>> np.expand_dims(arr, axis=1).shape
(5, 1)
```

### np.reshape(a, newshape)

方法，给一个维度设置为1完成升维

```python
# 添加行维度
np.reshape(arr, (1, 5)) 
np.reshape(arr, (1, -1)) # 指定列数太复杂，可以直接指定-1，那么就可以对1所在位置，也就是行进行升维
np.reshape(arr, (1, -1)).shape

# 添加列维度
np.reshape(arr, (-1, 1))
np.reshape(arr, (-1, 1)).shape
```

## 数据合并

> **背景**
>
> 在给机器学习准备数据的过程中，经常需要进行不同来源的数据合并的操作。
>
> **两类场景**
>
> 1. 给已有的数据添加多行，比如**增添—些样本**数据进去尽
> 2. 给已有的数据添加多列，比如**增添一些特征**进去;以下操作均可以实现数组合并:
>
> **方法**
>
> + `np.concatenate(array_list, axis=0/1)`：沿着指定axis进行数组的合并，默认axis=0(行)
>
> + `np.vstack`或者`np.row_stack(array_list)`：垂直vertically、按行row wise进行数据合并
>
> + `np.hstack`或者`np.column_stack(array_list)`：水平horizontally、按列column wise进行数据合并

示例代码

```python
# 行拓展
>>> a = np.arange(6).reshape(2, 3)
>>> a
array([[0, 1, 2],
       [3, 4, 5]])
>>> b = np.random.randint(10, 20, size=(4, 3))
>>> b
array([[18, 17, 15],
       [17, 17, 17],
       [12, 14, 11],
       [19, 19, 14]])

>>> np.concatenate([a, b])
>>> np.vstack([a, b])
>>> np.row_stack([a, b])
array([[ 0,  1,  2],
       [ 3,  4,  5],
       [18, 17, 15],
       [17, 17, 17],
       [12, 14, 11],
       [19, 19, 14]])

# 列拓展
>>> a = np.arange(6).reshape(2, 3)
>>> a
array([[0, 1, 2],
       [3, 4, 5]])
>>> b = np.random.randint(10, 20, size=(2, 3))
>>> b
array([[17, 17, 15],
       [15, 10, 18]])

>>> np.concatenate([a, b], axis=1)
>>> np.hstack([a, b])
>>> np.column_stack([a, b])
array([[ 0,  1,  2, 17, 17, 15],
       [ 3,  4,  5, 15, 10, 18]])
```

