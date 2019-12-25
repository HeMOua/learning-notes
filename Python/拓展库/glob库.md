# glob库

glob模块是最简单的模块之一，内容非常少。用它可以查找符合特定规则的文件路径名。跟使用windows下的文件搜索差不多。

查找文件只用到三个匹配符：`*`、`?`、`[]`

+ `*`：匹配0个或多个字符
+ `?`：匹配单个字符
+ `[]`：匹配指定范围内的字符，如：`[0-9]`匹配数字

## glob.glob

返回所有匹配的文件路径列表。它只有一个参数`pathname`，定义了文件路径匹配规则，这里可以是绝对路径，也可以是相对路径。下面是使用glob.glob的例子：

```python
import glob

#获取指定目录下的所有图片
print (glob.glob(r"/home/qiaoyunhao/*/*.png"),"\n")#加上r让字符串不转义

#获取上级目录的所有.py文件
print (glob.glob(r'../*.py')) #相对路径
```

## glob.iglob

获取一个可编历对象，使用它可以逐个获取匹配的文件路径名。与glob.glob()的区别是：glob.glob同时获取所有的匹配路径，而glob.iglob获取一个路径的可迭代对象

```python
import glob
#父目录中的.py文件
f = glob.iglob(r'./*.py')
print (f) #<generator object iglob at 0x00B9FF80>
for py in f:
    print (py)
```

