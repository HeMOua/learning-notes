# tqdm库

> tqdm 是一个快速，可扩展的Python进度条，可以在 Python 长循环中添加一个进度提示信息，用户只需要封装任意的可迭代对象(iterable)

## 简单使用

1、基本使用

```python
from tqdm import tqdm

for i in tqdm(range(1000)):  
     #do something
     pass  
```

2、描述信息

这样会用进度条显示，并不能知道这个进度条是干什么的，我们可以使用`desc`属性来添加一些描述信息

```python
for i in tqdm(range(1000), desc='training...'):  
     #do something
     pass  
```

而通过`set_description`方法可以**动态**的给进度条设置左侧的描述信息

```python
from tqdm import tqdm, trange
import time

bar = trange(10)
for i in bar:  
    bar.set_description(f'Processing {i}')
    time.sleep(0.1)
    pass  
```

通过`set_postfix`可以给进度条设置右侧的描述信息

```python
from tqdm import trange
from random import random,randint
import time

with trange(100, desc="training") as t:
    for i in t:
        t.set_postfix(loss=random(),gen=randint(1,999),str="h",lst=[1,2])
        time.sleep(0.1)
```

效果如下

```
training: 100%|█████████████████████| 100/100 [00:10<00:00,  9.17it/s, gen=341, loss=0.332, lst=[1, 2], str=h]
```

3、进度条颜色

如果希望进度条骚一点，可以给他加个颜色

```python
for i in tqdm(range(10), colour='green'):  
     #do something
     pass  
```



## 参考

[tqdm的常见用法以及参数详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/377685333)