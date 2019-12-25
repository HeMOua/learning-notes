# 安装Tensorflow

```text
conda create -n tf[x] python=3.6
```

## 安装tf1

## 判断

```python
tf.test.is_gpu_available()
```

如果提示如下等信息

```
Could not load dynamic library 'libcudart.so.10.0'; dlerror: libcudart.so.10.0: cannot open shared object file: No such file or directory

Could not load dynamic library 'libcublas.so.10.0'; dlerror: libcublas.so.10.0: cannot open shared object file: No such file or directory
```

则说明没有安装`cudatoolkit`，根据提示安装相应版本即可，如下

```shell
conda install cudatoolkit=10.0
```

