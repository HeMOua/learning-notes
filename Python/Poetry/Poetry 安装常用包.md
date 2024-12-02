## 安装 torch

首先，我们添加一个 source

```shell
poetry source add --priority=supplemental pytorch_cu121 https://download.pytorch.org/whl/cu121
```


通过指定 source 安装

```shell
poetry add --source pytorch_cu121 torch torchvision torchaudio
```

## 安装 PIL

```
poetry add Pillow
```

## 安装 pyinstaller

```shell
poetry add --group dev pyinstaller
```

可能会出现以下报错

```shell
poetry add --group dev pyinstaller
Using version ^6.10.0 for pyinstaller

Updating dependencies
Resolving dependencies... (0.0s)

The current project's supported Python range (>=3.12,<4.0) is not compatible with some of the required packages Python requirement:
  - pyinstaller requires Python <3.14,>=3.8, so it will not be satisfied for Python >=3.14,<4.0

Because no versions of pyinstaller match >6.10.0,<7.0.0
 and pyinstaller (6.10.0) requires Python <3.14,>=3.8, pyinstaller is forbidden.
So, because 图片归类器 depends on pyinstaller (^6.10.0), version solving failed.

  • Check your dependencies Python requirement: The Python requirement can be specified via the `python` or `markers` properties

    For pyinstaller, a possible solution would be to set the `python` property to ">=3.12,<3.14"

    https://python-poetry.org/docs/dependency-specification/#python-restricted-dependencies,
    https://python-poetry.org/docs/dependency-specification/#using-environment-markers
```

可以看见以下关键信息

```
pyinstaller requires Python <3.14,>=3.8, so it will not be satisfied for Python >=3.14,<4.0
```

而 poetry 中默认的配置如下：

```toml
[tool.poetry.dependencies]
python = "^3.12"
```

需要将其修改为如下形式

```toml
[tool.poetry.dependencies]
python = ">=3.12,<3.14"
```

这样就能正确安装依赖了
