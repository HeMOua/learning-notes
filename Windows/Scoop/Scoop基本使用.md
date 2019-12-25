Scoop基本使用

## 安装

> 打开 [PowerShell](https://so.csdn.net/so/search?q=PowerShell&spm=1001.2101.3001.7020)远程权限
>
> ```
> Set-ExecutionPolicy RemoteSigned -scope CurrentUser
> ```
>
> 若出现提示是否要更改执行策略?，输入 Y 回车

可以参考文档 [ScoopInstaller/Install: 📥 Next-generation Scoop (un)installer (github.com)](https://github.com/ScoopInstaller/Install)

首先下载 `isntall.ps1` 文件，打开 power shell，输入以下命令：

```cmd
irm get.scoop.sh -outfile 'install.ps1'
```

并输入以下命令自定义安装目录

```cmd
.\install.ps1 -ScoopDir 'D:\ProgramData\Scoop' -ScoopGlobalDir 'E:\ServerTools\GlobalScoopApps' -NoProxy
```

## 应用仓库源(bucket)

以下是已知的 Scoop 仓库：

+ [main](https://github.com/ScoopInstaller/Main) - Default bucket for the most common (mostly CLI) apps
+ [extras](https://github.com/ScoopInstaller/Extras) - Apps that don't fit the main bucket's [criteria](https://github.com/ScoopInstaller/Scoop/wiki/Criteria-for-including-apps-in-the-main-bucket)
+ [games](https://github.com/Calinou/scoop-games) - Open source/freeware games and game-related tools
+ [nerd-fonts](https://github.com/matthewjberger/scoop-nerd-fonts) - Nerd Fonts
+ [nirsoft](https://github.com/kodybrown/scoop-nirsoft) - Almost all of the [250+](https://rasa.github.io/scoop-directory/by-apps#kodybrown_scoop-nirsoft) apps from [Nirsoft](https://nirsoft.net/)
+ [sysinternals](https://github.com/niheaven/scoop-sysinternals) - Sysinternals Suite and all individual application from [Microsoft](https://learn.microsoft.com/sysinternals/)
+ [java](https://github.com/ScoopInstaller/Java) - A collection of Java development kits (JDKs), Java runtime engines (JREs), Java's virtual machine debugging tools and Java based runtime engines.
+ [nonportable](https://github.com/ScoopInstaller/Nonportable) - Non-portable apps (may require UAC)
+ [php](https://github.com/ScoopInstaller/PHP) - Installers for most versions of PHP
+ [versions](https://github.com/ScoopInstaller/Versions) - Alternative versions of apps found in other buckets

main 仓库已被默认安装。要添加任何其他存储段，请键入以下内容：

```cmd
scoop bucket add bucketname
```

比如想要安装 extras 仓库，可以键入

```
scoop bucket add extras
```

如果出现问题，或者不想用这个 bucket 了，那么可以用下面这条语句来删除：

```
scoop bucket rm extras
```

## 基本指令

+ 帮助语法：scoop help

+ 搜索软件：scoop search python

+ 安装操作：scoop install 软件名

+ 安装指定 Bucket 中的应用：scoop install extras/sumatrapdf
+ 安装指定版本：scoop install python@3.7.9
+ 查看软件信息：scoop info python
+ **查看当前已安装软件**：scoop list

+ 版本切换
  scoop reset python
  scoop reset python27

+ 更新指定应用：scoop update python

+ 禁止更新指定应用：scoop hold python
+ 解除禁止更新指定应用：scoop unhold python
+ 更新所有已安装应用：scoop update *
+ 更新 bucket 库：scoop update
+ 清理所有旧版本：scoop cleanup *
+ 卸载操作：scoop uninstall 软件名
+ 全局卸载（包括 persist）：scoop uninstall 软件名 -p
+ 删除旧版本：scoop cleanup

