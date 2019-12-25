# nohup使用

## 启动

```shell
nohup python -u test.py > test.log 2>&1 &
```

+ 最后一个“&”表示后台运行程序
+ \> test.log：表示将打印信息重定向到test.log文件中
+ 2>&1：表示将标准错误输出转变化标准输出，可以将错误信息也输出到日志文件中

## 停止

1、`jobs -l`

jobs命令只看当前终端生效的，关闭终端后，在另一个终端jobs已经无法看到后台跑得程序了，此时利用ps

2、ps grep，然后 `kill -9 pid`jobs
