# notepad++配置c/c++环境

## 安装MinGW

[MinGW-w64安装教程——著名C/C++编译器GCC的Windows版本 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/76613134)

## 配置Notepad++

1、打开notepad++，按F5，或者点击菜单栏“运行->运行”，弹出如下对话框，输入：

```shell
cmd /k cd /d "$(CURRENT_DIRECTORY)" & g++ "$(FILE_NAME)" -o "$(NAME_PART)" & "$(NAME_PART).exe"
```

然后点击"保存"

2、点击“保存”后，弹出如下对话框，设置此命令的名称与快捷键，例如：name输入“C/C++ compiler”,shortcut （快捷键）设置为自己习惯的即可。点击“OK”，大功告成

