## 修改缓存路径 

使用环境变量 `UV_CACHE_DIR`

设置 `UV_CACHE_DIR` 变量，让 `uv` 使用自定义的缓存目录。例如：

```shell
export UV_CACHE_DIR=/path/to/custom/cache  # Linux/macOS
set UV_CACHE_DIR=C:\path\to\custom\cache   # Windows (CMD)
$env:UV_CACHE_DIR="C:\path\to\custom\cache" # Windows (PowerShell)
```

然后运行 `uv add` 之类的命令，`uv` 就会使用新的缓存目录。

## uv 换源

**1、全局源**

UV 提供了 `UV_DEFAULT_INDEX` 环境变量来设置默认的包索引源。

镜像源例如 `https://pypi.tuna.tsinghua.edu.cn/simple`。

**2、用户级别配置文件**

文件路径：%APPDATA%\uv\uv.toml

在该文件中添加以下内容：

```toml
[[index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

**3、项目级别配置文件**

文件路径：项目目录下的 pyproject.toml 或 uv.toml。

在 pyproject.toml 中添加：

```toml
[[tool.uv.index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

或在 uv.toml 中添加：

```toml
[[index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

**4、通过命令行临时指定**

在命令行中运行 UV 命令时，可以直接指定镜像源：

```toml
uv add --default-index https://pypi.tuna.tsinghua.edu.cn/simple requests
```

**5、注意事项**

如果同时设置了环境变量和配置文件，环境变量的优先级更高。

通过以上方法，你可以在 Windows 系统上灵活地修改 UV 的下载源。

## 定义检索

**1、通过配置文件**

```toml
[[tool.uv.index]]
# Optional name for the index.
name = "pytorch"
# Required URL for the index.
url = "https://download.pytorch.org/whl/cpu"
```

**2、通过命令行**

```
uv lock --index pytorch=https://download.pytorch.org/whl/cpu
```

