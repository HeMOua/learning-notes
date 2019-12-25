# argparse基本用法

## 简单使用

编写`1.py`文件，内容如下

```python
import argparse

def main():
    parser = argparse.ArgumentParser(description="Demo of argparse")
    parser.add_argument('-n','--name', default='Lisi')
    parser.add_argument('-y','--year', default='20')
    args = parser.parse_args()
    print(args)
    name = args.name
    year = args.year
    print('Hello {}  {}'.format(name,year))

if __name__ == '__main__':
    main()
```

然后在cmd中运行`python 1.py -n HeMOu -y 20`，可得到下面的结果

```
# python 1.py -n HeMOu -y 20
Namespace(name='HeMOu', year='20')
Hello HeMOu  20
```



