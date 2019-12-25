# pathlib库

> 引入库`from pathlib import Path`

## 基本用法

在过去，文件的路径是纯字符串，现在它会是一个`pathlib.Path`对象

```
In : p = Path('/home/ubuntu')

In : p
Out: PosixPath('/home/ubuntu')

In : str(p)
Out: '/home/ubuntu'
```

使用str函数可以把一个Path对象转化成字符串。

>**在Python 3.6之前**，Path对象是不能作为os模块下的参数的，需要手动转化成字符串:
>
>```python
>import os
>
>p = Path('/')
>os.chdir(p)      # 错误
>os.chdir(str(p)) # 正确
>```
>
>从Python 3.6开始，这些接受路径作为参数的函数内部会先通过`os.fspath`调用Path对象的`__fspath__`方法获得字符串类型的路径再去执行下面的逻辑。所以要注意: **如果你想全面使用pathlib模块，应该使用Python3.6或者更高版本！**

## 拼接路径

1、使用`joinpath`

```python
In : Path('/').joinpath('home', 'dongwm/code')
Out: PosixPath('/home/dongwm/code')
```

2、更方便的使用`/`进行拼接

```python
In : Path('/') / 'home' / 'dongwm/code'
Out: PosixPath('/home/dongwm/code')

In : Path('/') / Path('home') / 'dongwm/code'
Out: PosixPath('/home/dongwm/code')

In : '/' / Path('home') / 'dongwm/code'
Out: PosixPath('/home/dongwm/code')
```

## 属性

+ `Path('xx').name` ：文件名
+ `Path('xx').stem` / `Path('xx').suffix` ：后缀

## 方法 ★

+ `Path('xx').is_file()`：判断是否存在，并且是文件
+ `Path('xx').is_dir()`：判断是否存在，并且是目录
+ `Path('xx').rename(newpath)`：移动到指定路径，注意是路径，需要包括文件名
+ `Path(x).resolve()`：获取文件的绝对路径
+ `Path(x).unlink()`：获取文件的绝对路径
+ `Path(x).with_suffix('.txt')`：修改文件到指定后缀，可以结合`rename`一起使用

## 参考

[你应该使用pathlib替代os.path - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/87940289)