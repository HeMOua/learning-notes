[TOC]

# AutoHotKey

## 一、热键&热字符串

### 1、热键

`::`左边的就叫热键，如下

```AutoHotKey
^j::
	Send, my first script
```

在这里`^j`就是热键，它在`::`的左边

### 2、热字符串

位于`::`左边的叫热键，热键以`::`开头的就是热字符串，当输入热字符串完全时，并按住<kbd>enter</kbd>键就会将其替换为`热键`之后和`return`之前的字符串或`热键`之后的字符串。如下

```AutoHotKey
;第一种
::ftw::Free the whales

;第二种
::ftw::
	Send, Free the whales
return
```

若将热字符串开头的`::`替换成`:*:`则不需要输入<kbd>enter</kbd>也可以直接替换。

### 3、热键修饰符

| 符号 |                             描述                             |
| :--: | :----------------------------------------------------------: |
|  #   |                <kbd>Win</kbd>(windows徽标键)                 |
|  ！  |                        <kbd>Alt</kbd>                        |
|  ^   |                       <kbd>ctrl</kbd>                        |
|  +   |                       <kbd>shift</kbd>                       |
|  &   |              用于连接**两个**按键（含鼠标按键）              |
|  ~   | 修饰键，不会覆盖掉原快捷键的作用，假如设置一个快捷键`^c`那么这按<kbd>ctrl</kbd>+<kbd>c</kbd>本身的复制作用就是失效，而`~`的作用就是不屏蔽之前的快捷键 |

最需要注意的是不要把**按键**和**热键修饰符**弄混了，热键修饰符是用来修饰按键的，单独的热键修饰符不能作为热键。

例如，如下代码

```ahk
^::
MsgBox showTip
return
```

本意是想按下<kbd>ctrl</kbd>键时就弹出一个提示框，但是实际上它并没啥作用，正确的写法是

```ahk
ctrl::
MsgBox showTip
return
```

点击[这里](https://wyagd001.github.io/zh-cn/docs/KeyList.htm)可查看所有按键列表

### 4、指定窗口热键

```ahk
#IfWinActive
#IfWinExist
```

### 5、goto与gosub的区别

`goto`跳转过后不会返回到原来的位置

`gosub`跳转之后会回到原来的位置

### 6、对象

想创建一个对象属性必须先创建对象，如下

```ahk
global config.keys := "mykey"
msgbox % config.keys
```

本意是想给`config`创建一个名为`keys`的属性，但是`msgbox`总是为空，后来才发现需要首先创建`config`对象，如下

```ahk
global config := {}
config.keys := "mykey"
msgbox % config.keys
```

### 7、#include

通过指令包含的文件，如果想要其按照自己想的顺序去执行，最好要加上`gosub`与标签，不然会发生意想不到的结果，比如`lib_init.ahk`里面的代码执行完后，最好添加`gosub`跳转到指定标签。

