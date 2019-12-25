~~杀死所有apt进程~~，没卵用

```
sudo killall apt apt-get
```





运行下面的命令来生成所有含有 apt 的进程列表，你可以使用ps和grep命令并用管道组合来得到含有apt或者apt-get的进程，然后 kill 干掉他

```
$ ps -A | grep apt
$ sudo kill -9 进程ID
```

