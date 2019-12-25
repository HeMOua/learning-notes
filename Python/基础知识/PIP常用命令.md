# PIP常用命令

## 列出已安装的包

```shell
pip freeze
pip list
```

### 导出 requirements.txt

```shell
pip freeze > <目录>/requirements.txt
```

## 安装包

### 在线安装

```shell
pip install <包名>` 或 `pip install -r requirements.txt
# 安装1.9版本的django
pip install django==1.9
# 安装版本号大于1.9的django，注意有引号
pip install "django>1.9"
pip install "django>=1.9"
pip install "django<1.9"
pip install "django<=1.9"
pip install "django><1.9"
```

requirements.txt 内容格式为：

```
APScheduler==2.1.2
Django==1.5.4
MySQL-Connector-Python==2.0.1
MySQL-python==1.2.3
PIL==1.1.7
South==1.0.2
django-grappelli==2.6.3
django-pagination==1.0.7
```

### 安装本地安装包

```shell
pip install <目录>/<文件名>
```

```shell
pip install --use-wheel --no-index --find-links=wheelhouse/ <包名>
```

*<包名> 前有空格*

可简写为

```shell
pip install --no-index -f=<目录>/ <包名>
```

### 卸载包

```shell
pip uninstall <包名>` 或 `pip uninstall -r requirements.txt
```

### 升级包

```shell
pip install -U <包名>
```

### 升级 pip

```shell
pip install -U pip
```

## 显示包所在的目录

```shell
pip show -f <包名>
```

## 搜索包

```shell
pip search <搜索关键字>
```

## 查询可升级的包

```shell
pip list -o
```

## 下载包而不安装

```shell
pip install <包名> -d <目录>` 或 `pip install -d <目录> -r requirements.txt
```

## 打包

```shell
pip wheel <包名>
```

## 更换国内 pypi 镜像

### 国内 pypi 镜像

- 豆瓣：https://pypi.douban.com/simple
- 中国科学技术大学：https://mirrors.ustc.edu.cn/pypi/web/simple/
- 清华大学 TUNA：https://pypi.tuna.tsinghua.edu.cn/simple
  https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple/

### 指定单次安装源

```shell
pip install <包名> -i http://pypi.v2ex.com/simple
```

### 指定全局安装源

在 unix 和 macos，配置文件为：$HOME/.pip/pip.conf （可用）
在 windows 上，配置文件为：% HOME%\pip\pip.ini

```shell
[global]
timeout = 6000
index-url = https://pypi.mirrors.ustc.edu.cn/simple/ 
```

### 便捷换源包

```
pip install pqi

pqi ls

pqi use <name>
```

