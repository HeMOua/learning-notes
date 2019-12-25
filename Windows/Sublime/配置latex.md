# Sublime配置Latex

## 1、先安装 `LatexTools` 和 `Latex-cwl`

## 2、配置latextools

找到latexttools的user配置文件

2.1 配置latex路径

```json
	"windows": {
		// Path used when invoking tex & friends; "" is fine for MiKTeX
		// For TeXlive 2011 (or other years) use
		// "texpath" : "C:\\texlive\\2011\\bin\\win32;$PATH",
		"texpath" : "E:\\TeXLive\\2021\\bin\\win32;$PATH",
		// TeX distro: "miktex" or "texlive"
		"distro" : "texlive",
		// Command to invoke Sumatra. If blank, "SumatraPDF.exe" is used (it has to be on your PATH)
		"sumatra": "E:\\Artifact\\DocumentRelated\\SumatraPDF\\SumatraPDF.exe",
		// Command to invoke Sublime Text. Used if the keep_focus toggle is true.
		// If blank, "subl.exe" or "sublime_text.exe" will be used.
		"sublime_executable": "",
		// how long (in seconds) to wait after the jump_to_pdf command completes
		// before switching focus back to Sublime Text. This may need to be
		// adjusted depending on your machine and configuration.
		"keep_focus_delay": 0.5
	},
```

大概在209行，主要修改三个地方

+ texpath：tex的路径
+ distro：改成texlive
+ sumatra：sumatra的安装地址

2.2 修改构建模式

380行左右，没有被注释的那块

```
"builder": "simple",
```

然后保存，至此 latextools 插件配置好了

## 3、配置sumatra的环境变量

将这个pdf阅读器位置放在环境变量中

## 4、配置sumatra的反向定位功能

sumatraPDF选项中的“设置反向搜索命令行”，如下配置

`"D:\Program Files\Sublime Text 3\sublime_text.exe" "%f:%l"`

## 5、配置sublime设置

```json
"added_words": [
    "reactance"
],
"auto_complete": true,
"auto_complete_delay": 50,
"auto_complete_selector": "source, text",
"auto_complete_triggers":
[
    {"selector": "text.html, text.xml", "characters": "<"},
    {"selector": "text.tex.latex", "characters": "\\"},
],
"spell_check": false,
```

## 6、配置快捷键

```json
{ "keys": ["ctrl+t"], "command": "title_case" },
{ "keys": ["ctrl+l"], "command": "lower_case" },
{   
    "keys": ["ctrl+1"], 
    "context":  [{
        "key": "selector", "operator": "equal", "operand": "text.tex.latex"
    }],
    "command": "jump_to_pdf", "args": {"from_keybinding": true}
},
{
    "keys": ["ctrl+2"],
    "args": {"only_current_file": true},
    "context":  [{
        "key": "selector", "operator": "equal", "operand": "text.tex"
    }],
    "command": "latex_toc_quickpanel"
},
```



## 问题

### 弹出新窗口

编译后自动弹出一个新窗口，这里有两种解决方法：

+ 方法一（不推荐）

找到 latex 配置文件，搜索 `focus` 

```json
"keep_focus": false
```



+ 方法二（推荐）

找到 latex 配置文件，搜索 `window` ，然后设置好 `sublime_executable` 配置，如下：

```json
"windows": {
		// Path used when invoking tex & friends; "" is fine for MiKTeX
		// For TeXlive 2011 (or other years) use
		// "texpath" : "C:\\texlive\\2011\\bin\\win32;$PATH",
		"texpath" : "E:\\TeXLive\\2021\\bin\\win32;$PATH",
		// TeX distro: "miktex" or "texlive"
		"distro" : "miktex",
		// Command to invoke Sumatra. If blank, "SumatraPDF.exe" is used (it has to be on your PATH)
		"sumatra": "E:\\Artifact\\DocumentRelated\\SumatraPDF\\SumatraPDF.exe",
		// Command to invoke Sublime Text. Used if the keep_focus toggle is true.
		// If blank, "subl.exe" or "sublime_text.exe" will be used.
		"sublime_executable": "D:\\Program Files\\Sublime Text\\subl.exe",
		// how long (in seconds) to wait after the jump_to_pdf command completes
		// before switching focus back to Sublime Text. This may need to be
		// adjusted depending on your machine and configuration.
		"keep_focus_delay": 0.5
	}
```

`sublime_executable` 对应的值是 sublime 安装目录下的 `subl.exe` 文件

### 补全反斜杠消失

中文版本，选择 首选项→浏览插件目录→打开LaTexTools文件夹→找到latex_cwl_completions.py

将

```python
(c[0], c[1][1:]) if c[1].startswith("\") else c
```

 替换成

```python
(c[0], c[1]) if c[1].startswith("\") else c
```

