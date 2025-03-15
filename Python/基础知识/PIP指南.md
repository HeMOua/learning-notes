# PIP指南

## 常用命令

### 安装

#### 在线安装

```shell
pip install <包名>
pip install -r requirements.txt
# 安装1.9版本的django
pip install django==1.9
# 安装版本号大于1.9的django，注意有引号
pip install "django>1.9"
```

#### 离线安装(本地包)

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

### 卸载

```shell
pip uninstall <包名>
pip uninstall -r requirements.txt
```

### 升级

```shell
pip install -U <包名>
```

### 下载

```
pip install <包名> -d <目录>
pip install -d <目录> -r requirements.txt
```

### 导出

1、列出已安装的包

```shell
pip freeze
pip list
```

2、导出 requirements.txt

```shell
pip freeze > <目录>/requirements.txt
```

### 清除缓存

```shell
pip cache purge
```

### 其他

+ 显示包所在路径，不常用

  ```shell
  pip show -f <包名>
  ```

+ 搜索包

  ```
  pip search <搜索关键字>
  ```

+ 查询可升级包

  ```
  pip list -o
  ```

+ 打包

  ```
  pip wheel <包名>
  ```

## 换源

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

+ 在 unix 和 macos，配置文件为：`$HOME/.pip/pip.conf` 

+ 在 windows 上，配置文件为：`%HOME%\pip\pip.ini`

  ```ini
  [global]
  timeout = 6000
  index-url = https://pypi.tuna.tsinghua.edu.cn/simple
  [install]
  trusted-host = pypi.tuna.tsinghua.edu.cn
  ```

### 便捷换源包

```
pip install pqi

pqi ls

pqi use <name>
```

## pip配置文件位置 

（1）**Windows:** (可能在以下三个位置之一，应该是取决于安装方式)

- `C:\ProgramData\pip\pip.ini`（全局）

- `C:\Users\YOURUSERNAME\pip\pip.ini`（单个用户配置）

- `C:\Users\YOURUSERNAME\AppData\Roaming\pip\pip.ini`（单个用户配置）

我是仅为当前用户安装的（无管理员权限），pip.ini在 

```
C:\Users\YOURUSERNAME\AppData\Roaming\pip\pip.ini
```

（2）**Linux:**

```shell
~/.pip/pip.conf
```

若没有，可以用以下命令创建

```shell
mkdir -p ~/.pip && touch ~/.pip/pip.conf
```

## 修复pip

使用 `get-pip.py` 脚本重新安装 `pip`：

- 下载 `get-pip.py`

  ```shell
  curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
  ```

- 执行脚本重新安装：

  ```shell
  python get-pip.py
  ```

或者直接通过 `ensurepip` 重新安装：

```shell
python -m ensurepip --upgrade
```
