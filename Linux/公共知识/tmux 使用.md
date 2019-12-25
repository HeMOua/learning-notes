# tmux 使用

## 安装

```shell
# Ubuntu 
$ sudo apt update
$ sudo apt install tmux

# Mac
$ brew install tmux
```

## 会话

### 创建会话

```shell
$ tmux
```

可以新建一个session，其中包含一个window，该window中包含一个pane，pane里打开了一个shell对话框。

**tmux创建的session的名字默认都是按数字排序**，所以可以在进入tmux时对session自定义名字。

```shell
$ tmux new -s [session_name]
```

### 退出会话

如果需要重新退回terminal，可输入`Ctrl + b, d` 。其中 d 表示**detaching**，运行后并不会真正关闭session，而是将session挂起，在tmux session里面的程序还是会在后台继续运行。

### 连接会话

如果想要重新连接刚才退出的tmux session，可以输入下面命令：

```shell
$ tmux attach
$ tmux a
```

如果想连接到特定的session：

```shell
$ tmux attach -t [session_name]

#也可以将attach简写成a
$ tmux a -t [session-name]
```

### 重命名会话

在session里时，`Ctrl + b` + `$` 对当前session进行重命名。

在terminal里，可直接对某个特定session重命名：

```shell
tmux rename-session -t 0 [session_new_name]
```

### 显示会话

`Ctrl + b` + `s` 可视化选择任意session，同时，按下**左右**键可以对当前session的windows进行展开和收起，以及对某个windows的panes进行展开并选择某一pane。

如果不知道当前存在哪些session，可以输入下方命令查看：

```shell
$ tmux ls
```

### 关闭会话

如果要关闭某个会话：

```shell
# 使用会话编号或具体名字
$ tmux kill-session -t 0
$ tmux kill-session -t <session-name>
```

在`tmux ls`中可以看到，第一列显示的是session名字，默认创建时名字是从1开始的整数，然后表示当前session里有几个windows。创建一个session的时候，默认只有一个窗口。

## 会话内命令

session、window、panel 结构如下

```
|--session1
	|--window1
		|--panel1
		|--panel2
		|--...
	|--window2
	|--...
|--session1
|--session1
|--...
```

### 窗口

+ 创建window：`Ctrl + b` `c`
+ 切换指定window：`Ctrl+b` `[window-name]` ，如`Ctrl+b` `0` 表示切换到 0 号 window
+ 切换到上一个 window：`Ctrl + b` `p`
+ 切换到下一个 window：`Ctrl + b` `n`

+ 对当前window进行重命名：`Ctrl+b` `,` 

+ 从window列表里选择window：`Ctrl+b` `w` ，该显示结果与`Ctrl + b` + `s` 一样。

### 面板

在新建的一个window里，默认只有一个pane，但是可以对其进行切分：

+ 将当前pane**分成左右**两个panes：`Ctrl+b` `%` 
+ 将当前pane**分成上下**两个panes：`Ctrl+b` `"` 
+ 通过**上下左右箭头来切换**panes：`Ctrl + b` `<arrow key>`
  + `Ctrl+b` `o`可以移动到下一个pane里（不推荐）
  + `Ctrl+b` `;`可以切换到上一个pane里（不推荐）
+ **关闭**当前pane：`Ctrl + d` ，如果当前window的所有pane均已关闭，则自动关闭当前window；直至所有window均已关闭，则自动关闭当前session
  + `Ctrl+b` `x`关闭当前所在pane，这种关闭，会在关闭前进行确认（不推荐）
  + 直接输入`exit`（不推荐）
+ **放大/缩小**当前pane：`Ctrl + b` `z` 可以将当前的pane进行放大/缩小。