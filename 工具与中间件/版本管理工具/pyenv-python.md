# pyenv

## Windows环境

### 安装

去 github 下载源码，只需要将 pyenv-win 放在合适的地方即可，需要添加以下环境变量

+ `xxx/pyenv-win/bin`：pyenv 的执行目录
+ `xxx/pyenv-win/shims`：python 路径

### 命令

+ `pyenv install [versoin]`：安装python
+ `pyenv local [versoin]`：设置当前文件夹python版本
+ `pyenv global [versoin]`：设置全局python版本
+ `pyenv version`：查看当前设置的 python 版本
+ `pyenv versions`：查看所有安装的 python 版本

### 换源

添加环境变量 `PYTHON_BUILD_MIRROR_URL`：`https://mirrors.huaweicloud.com/python/`