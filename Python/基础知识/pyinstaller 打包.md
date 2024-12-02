# pyinstaller 打包 exe

## 安装

```
pip install pyinstaller
```

## 路径

打包后的 ROOT 路径

```python
if getattr(sys, 'frozen', False):
    # 如果是在打包后的环境中运行，获取 .exe 文件的路径
    ROOT = Path(sys.executable).parent
else:
    # 如果是在开发环境中运行，获取当前文件的路径
    ROOT = Path(__file__).parent.resolve()
```

获取资源路径

```python
if getattr(sys, 'frozen', None):
    basedir = sys._MEIPASS
else:
    basedir = os.path.dirname(__file__)
```

设置图标

```python
def setup_icon(root):
    if getattr(sys, 'frozen', None):
        basedir = sys._MEIPASS
    else:
        basedir = os.path.dirname(__file__)
    root.iconbitmap(os.path.join(basedir, "favor.ico"))
```

## 打包

### 普通打包

```shell
pyinstaller -F -w -i xxx.ico xxx.py
```

### 打包 tkinterdnd2

```shell
pyinstaller -F -w -i symlink.ico src\main.py --add-data="symlink.ico;." --additional-hooks-dir=.\src
```

hook 文件在 src 下，名叫 `hook-tkinterdnd2.py`

```python
from PyInstaller.utils.hooks import collect_data_files
datas = collect_data_files('tkinterdnd2')
```

## 参数

| **参数名**                                                | **描述**                                                     |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| -n NAME, -name=NAME                                       | 可选的项目(产生的spec的)名字。如果省略，第一个脚本的主文件名将作为spec的名字 |
| -D                                                        | 文件夹模式。在打包完成后生成一个文件夹，其中包含一个exe文件和一个包含若干依赖文件的文件夹（详见上文）。（默认） |
| -F                                                        | 单文件模式。在打包完成后只会生成一个单独的exe文件（详见上文）。 |
| --add-data <SRC;DEST or SRC:DEST>                         | 指定一个文件夹或文件（非二进制），将其嵌入到exe中。          |
| --add-binary <SRC;DEST or SRC:DEST>                       | 和--add-data类似，不过指定的文件夹或文件是二进制的           |
| -p DIR--paths DIR                                         | 提供一个路径进行搜索并且导入里面的模块（不同的路径使用路径分隔符os.pathsep分隔开，或者多次使用这个参数）。这可以解决有时候第三方模块找不到的问题。 |
| --hidden-import MODULENAME<br />--hiddenimport MODULENAME | 需要进行额外导入的模块。当pyinstaller在程序中找不到一些模块时，需要你额外指定。这个参数可以多次使用，可以解决一些模块找不到的问题。 |
| --splash IMAGE_FILE                                       | 添加一个启动画面（图片文件）路径，在程序运行前显示指定的启动图片，起到加载提示的效果。 |
| -c, --console, --nowindowed                               | 打包程序运行后出现一个黑色的控制台窗口（默认）               |
| -w, --windowed, --noconsole                               | 打包程序运行后隐藏控制台窗口                                 |
| -i <FILE.ico><br />--icon <FILE.ico>                      | 设置打包后exe程序的图标（只能在Windows和macOS上使用）        |
| --disable-windowed-traceback                              | 禁用异常提示（只能在Windows和macOS上使用）                   |
| --help, -h                                                | 打印pyinstaller的帮助信息并退出                              |
| --workpath <your_custom_build_path>                       | 自定义的 `build` 文件夹路径                                  |
| --distpath <your_custom_dist_path>                        | 自定义的 `dist` 文件夹路径                                   |

## 示例

```
pyinstaller -F -w -i favor.ico app.py -n 图片归类器 --add-data "favor.ico;." --workpath E:/Cache/Temp/build
```

