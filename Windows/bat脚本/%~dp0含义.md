# %~dp0 的含义

％0 －％9表示命令行参数，%0表示bat文件名本身，％1－％9表示其后的参数，如在cmd输入命令

## %~dp0的含义分析

```
~           是扩展的意思，变量扩充(下面详细介绍)，相当于把一个相对路径转换绝对路径
%0          代指批处理文件自身（绝对路径 "D:\test\bat\test.bat"，注意有双引号）
%~d0        是指批处理所在的盘符，其中d代表drive。(扩充到盘符 D: )
%~p0        是指批处理所在的目录，其中p代表path。(扩充到路径 \test\bat\ ）
%~dp0       是批处理所在的盘符加路径。（扩充到盘符和路径 ：D:\test\bat\）
```

`cd %~dp0` 就是进入批处理所在目录了



## %cd%和%~dp0获取当前目录区别

%cd%和%~dp0都能用来表示当前目录，但是他们在不同的使用场景下，功能却不相同：

%cd% 代表的是执行文件的当前工作目录（强调bat是在哪里启动的，可变的）；
%~dp0 代表的是当前批处理文件所在完整目录（强调bat的文件位置，固定的）。