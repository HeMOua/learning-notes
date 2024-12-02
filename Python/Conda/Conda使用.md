# Conda使用

## 查看环境

```
conda env list
```

## 创建环境

`conda create -n pytorch python=3.6`

+ `-n`：后面接环境的名称
+ `python=3.6`：指定环境所安装的python版本

## 切换环境

激活环境

```cmd
conda activate [envName]
```

默认环境

```cmd
conda deactivate
```

## 删除环境

```
conda remove -n [envName] --all
```

## 重命名环境

conda 其实没有重命名指令，实现重命名是通过 clone 完成的，分两步：

- 先 clone 一份 new name 的环境
- 删除 old name 的环境

比如，想把环境 envName 重命名成 tf

```
# 第一步
conda create -n tf --clone [envName]
# 第二步
conda remove -n [envName] --all
```

## 换源

编辑 `~/.condarc`

```
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

## 修改 envs、pkgs 路径

编辑 `~/.condarc`

```
envs_dirs:
  - D:\ProgramData\miniconda3\envs
pkgs_dirs:
  - D:\ProgramData\miniconda3\pkgs
```

## 清理缓存

**（1）删除未使用的包**

```shell
conda clean --packages
```

**（2）删除缓存的压缩包文件**

清理缓存的压缩包文件。这些文件是在安装包时下载的，但在安装完成后就不再需要了。输入以下命令：

```shell
conda clean --tarballs
```

**（3）删除不再使用的包和缓存**

```bash
conda clean --all
```
