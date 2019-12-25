# Jupyter

## 安装

```
conda install jupyter notebook
```



## 起始目录

因为jupyter的配置默认不存在，所有首先将其产生

```shell
jupyter notebook --generate-config
```

然后就可以在用户目录下的`.jupyter`文件夹中找到它，下面是具体操作

1. 用记事本打开配置文件`jupyter_notebook_config.py`；
2. Crtl + F组合键找到`c.NotebookApp.notebook_dir`元素，删掉前面的注释`#`；
3. 在后面的单引号里输入要设置的目录路径（注意双斜杠），保存关闭； 例如：`c.NotebookApp.notebook_dir = "F:/CodeMgr/Jupyter"`
4. 修改快捷键，在win开始菜单中找到`jupyter notebook`快捷图标，右击选择属性，删除目标值最后的 “%USERPROFILE%/”，点击确定退出。

[Jupyter Notebook最常用的五大配置技巧](https://zhuanlan.zhihu.com/p/166165379)

## 快捷键

Jupyter Notebook 有两种键盘输入模式：

1、命令模式，键盘输入运行程序命令；这时的单元框线是蓝色。

2、编辑模式，允许你往单元中键入代码或文本；这时的单元框线是绿色的。

在编辑模式下按键 Esc键进入命令模式， 接着再按一次 Enter 键启动编辑模式

### 命令模式

**命令模式下的快键键：**

Enter : 转入编辑模式                                                

Shift-Enter : 运行本单元，选中下个单元

Ctrl-Enter : 运行本单元

Alt-Enter : 运行本单元，在其下插入新单元

- Y : 单元转入代码状态

- M :单元转入markdown状态

- R : 单元转入raw状态

- A : 在上方插入新单元

- B : 在下方插入新单元

- D,D : 删除选中的单元


- H : 显示快捷键帮助


### 编辑模式

编辑模式时边框为绿色，此时允许用户编辑文档，此模式下的快捷键有：

- Esc : 进入命令模式
- Shift-Enter : 运行本单元，选中下一单元

- Ctrl-Enter : 运行本单元

- Alt-Enter : 运行本单元，在下面插入一单元

- Tab : 代码补全或缩进


[JupyterNotebook的快捷键](http://blog.sina.com.cn/s/blog_a36a563e0102yunz.html)

## 局域网访问

```
# 找到 c.ConnectionFileMixin.ip 并修改如下：
c.ConnectionFileMixin.ip = '0.0.0.0'

# 找到 c.NotebookApp.ip 并修改如下：
c.NotebookApp.ip = '0.0.0.0'

c.NotebookApp.ip = '*'  # 允许访问此服务器的 IP，星号表示任意 IP
#c.NotebookApp.password = u'sha1:xxx:xxx' # 之前生成的密码 hash 字串
c.NotebookApp.open_browser = False # 运行时不打开本机浏览器
c.NotebookApp.port = 8888 # 使用的端口
c.NotebookApp.enable_mathjax = True # 启用 MathJax
c.NotebookApp.token = ''
```

## 开机自启

1、vim /etc/systemd/system/jupyter.service

```
[Unit]
Description=Jupyter Notebook
After=network.target
[Service]
Type=simple
User=chenchao
ExecStart=/home/chenchao/miniconda3/bin/jupyter-notebook
ExecReload=/home/chenchao/miniconda3/bin/jupyter-notebook
WorkingDirectory=/home/chenchao/CodeMgr/
#文件路径名
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

2、设置

```
systemctl enable jupyter #设置开机自启
systemctl start jupyter #启动
```

