# 进程池multiprocessing.Pool

在利用Python进行系统管理的时候，特别是同时操作多个文件目录，或者远程控制多台主机，并行操作可以节约大量的时间。

当被操作对象数目不大时，可以直接利用multiprocessing中的Process动态成生多个进程，十几个还好，但如果是上百个，上千个目标，手动的去限制进程数量却又太过繁琐，此时可以发挥进程池的功效。

Pool可以**提供指定数量的进程**供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程执行结束，然后使用刚刚空闲的进程来执行新的任务。

## 简单使用（非阻塞）

```python
#coding: utf-8
import multiprocessing
import time

def func(msg):
    print("msg:", msg)
    time.sleep(0.3)
    print("end")

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes = 3)
    for i in range(4):
        msg = "hello %d" %(i)
        
        #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去
        pool.apply_async(func, (msg, ))   

    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
    pool.close()
    
    # 调用join之前，先调用close函数，否则会出错。
    # 执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
    pool.join()   
    print("Sub-process(es) done.")
```

运行结果

```
Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~
msg: hello 0
end
msg: hello 2
end
msg: hello 1
end
msg: hello 3
end
Sub-process(es) done.
```

**函数解释**：

- `apply_async(func[, args[, kwds[, callback]]])`：它是**非阻塞**
- `apply(func[, args[, kwds]])`是**阻塞**的
- `close()`关闭pool，使其不在接受新的任务。
- `terminate()`结束工作进程，不在处理未完成的任务。
- `join()`主进程阻塞，等待子进程的退出， join方法要在close或terminate之后使用。

**执行说明**：

+ 创建一个进程池pool，并设定进程的数量为3，range(4)会相继产生四个对象[0, 1, 2, 3]，四个对象被提交到pool中
+ 因pool指定进程数为3，所以0、1、2会直接送到进程中执行，当其中一个执行完事后才空出一个进程处理对象3，所以输出“msg: hello 3”出现在最后。
+ 因为为非阻塞，主函数会自己执行自个的，不搭理进程的执行，所以运行完for循环后直接输出“mMsg: hark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~”，主程序在pool.join（）处等待各个进程的结束。

## 简单使用（阻塞）

```python
#coding: utf-8
import multiprocessing
import time

def func(msg):
    print("msg:", msg)
    time.sleep(0.3)
    print("end")

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes = 3)
    for i in range(4):
        msg = "hello %d" %(i)
        
        #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去
        pool.apply_async(func, (msg, ))   

    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
    pool.close()
    
    # 调用join之前，先调用close函数，否则会出错。
    # 执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
    pool.join()   
    print("Sub-process(es) done.")
```

**注意**：按理来说应该是先输出子线程的内容，msg + end，最后输出 Mark~ Mark~ 但是测试的效果不是这样，待以后研究

## imap和imap_unordered

这两者都用于对大量数据遍历多进程计算，返回一个迭代器(multiprocessing.pool.IMapIterator)。

**imap**返回结果顺序和输入相同，**imap_unordered**则为不保证顺序。

经过测试，发现Python多进程和imap()的一些特性:

1、`iter = pool.imap(fn, data)`一旦生成，无论使不使用iter，多进程计算都会开始。计算结果会缓存在内存中，所以要注意内存用尽的问题。

2、fn，即执行函数，不可以是局部对象（不能嵌套在其他函数里），否则会报错：

```python
def fn_outer():
    def fn(a,b):
        return a+b
    pool = Pool()
    pool.imap(fn, [(1,2)])
    pool.close()
# AttributeError: Can't pickle local object 'fn_outer.<locals>.fn'
```

3、使用进程池map数据时，如果每次的运算量很小，最后的效率还不如单进程。这时多进程切换造成的开销已大于多进程计算提升的效率。

这时，可以将输入数据集分段，每次map，计算一段。具体分段多大时获得最佳效率，需要实际测试。

4、Pool使用完毕后必须关闭，否则进程不会退出。

写法一

```python
from multiprocess import Pool
 
pool = Pool(args.n_cpus)
 
# eg: f_list = [a, b, c]
 
for a in pool.imap_unordered(f, f_list):
    # Some unrelated code
 
pool.close()
pool.join()
 
def f(a):
    # Some unrelated code 
    return a
```

写法二（推荐）

```python
from multiprocess import Pool
 
# eg: f_list = [a, b, c]
 
with Pool() as pool:
    for a in pool.imap_unordered(f, f_list):
        # Some unrelated code
 
def f(a):
    # Some unrelated code 
    return a
```



## 参考

[python进程池：multiprocessing.pool](https://www.cnblogs.com/kaituorensheng/p/4465768.html)

[进程池Pool的imap方法](https://zhuanlan.zhihu.com/p/315627656)