# SdkMan

## 1.安装

在终端中输入以下命令进行安装:

```shell
$ curl -s "https://get.sdkman.io" | bash
```

如果提示缺少zip或unzip,安装后再次执行上面的命令即可.

```shell
# 安装需要的组件,Ubuntu为例
$ apt install zip
$ apt install unzip
```

安装完成后,在终端中输入:

```shell
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

## 2.安装到自定义位置

SDKMAN的默认安装位置为:$HOME/.sdkman.你可以通过设置SDKMAN_DIR环境变量来修改安装位置:

```shell
$ export SDKMAN_DIR="/usr/local/sdkman" && curl -s "https://get.sdkman.io" | bash
```

## 3.Beta通道

SDKMAN的Bate版,包含一些cli的新功能,但是可能会不稳定.如果需要使用Bate版本,需要修改~/.sdkman/etc/config文件:

```shell
sdkman_beta_channel=true
```

然后打开一个终端执行:

```shell
$ sdk selfupdate force
```

如果不需要使用Bate版本了,将上面的配置修改为false,再执行一次更新即可.

## 4.卸载

SDKMAN!没有提供自动化的卸载方法,可以通过以下命令进行卸载:

```shell
tar zcvf ~/sdkman-backup_$(date +%F-%kh%M).tar.gz -C ~/ .sdkman
$ rm -rf ~/.sdkman
```

然后从.bashrc，.bash_profile和/或.profile文件中编辑和删除初始化代码片段。如果您使用ZSH，请将其从.zshrc文件中删除。要删除的代码片段如下所示：

```shell
#THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
[[ -s "/home/dudette/.sdkman/bin/sdkman-init.sh" ]] && source "/home/dudette/.sdkman/bin/sdkman-init.sh"
```

## 5.使用

### 5.0 列出支持的软件 ★

```shell
$ sdk list # 执行命令后进入vi模式进行阅读,q退出阅读
```

### 5.1 列出软件的版本 ★

```shell
$ sdk list gradle
```

### 5.2 安装gradle

```shell
$ sdk install gradle
```

### 5.3 安装指定版本软件 ★

```shell
# 后面跟上版本号即可
$ sdk install gradle 4.4.1
```

### 5.4 安装本地包

```shell
$ sdk install groovy 3.0.0-SNAPSHOT /path/to/groovy-3.0.0-SNAPSHOT
```

### 5.5 卸载包

```shell
$ sdk uninstall scala 2.11.6
```

### 5.6 选择版本

选择一个版本用于当前终端:

```shell
$ sdk use scala 2.12.1
```

### 5.7 设置默认版本

```shell
$ sdk default scala 2.11.6
```

### 5.8 查看当前使用的版本

```shell
$ sdk current java  
	Using java version 8u111
	
#查看所有本地包的当前版本
$ sdk current  
	Using:  
	groovy: 2.4.7  
	java: 8u111  
	scala: 2.12.1
```

### 5.9 sdk版本升级

```shell
$ sdk upgrade springboot  
	Upgrade:  springboot (1.2.4.RELEASE, 1.2.3.RELEASE < 1.2.5.RELEASE)

# 本地所有sdk全部升级
$ sdk upgrade 
	Upgrade:  gradle (2.3, 1.11, 2.4, 2.5 < 2.6)  
	grails (2.5.1 < 3.0.4)  
	springboot (1.2.4.RELEASE, 1.2.3.RELEASE < 1.2.5.RELEASE)
```

### 5.10 离线模式

```shell
$ sdk offline enable  Forced offline mode enabled.
$ sdk offline disable  Online mode re-enabled!
```

当电脑没有网的时候,离线模式会进行自动切换.

### 5.11 SDKMAN!版本升级

```shell
$ sdk selfupdate
# 强制重新安装$ sdk selfupdate force
```

## 6.环境变量设置原理

git-bash 首先会加载用户目录的 `.bashrc` 文件，安装了 sdkman 后会额外多出两行

```shell
export SDKMAN_DIR="E:\Config\.sdkman"
[[ -s "E:\Config\.sdkman/bin/sdkman-init.sh" ]] && source "E:\Config\.sdkman/bin/sdkman-init.sh"
```

我们找到 `${SDKMAN_DIR}/bin/sdkman-init.sh` 文件，它内部存在以下代码会进行环境变量的设置

```shell
for candidate_name in "${SDKMAN_CANDIDATES[@]}"; do
    candidate_dir="${SDKMAN_CANDIDATES_DIR}/${candidate_name}/current"
    if [[ -h "$candidate_dir" || -d "${candidate_dir}" ]]; then
        __sdkman_export_candidate_home "$candidate_name" "$candidate_dir"
        __sdkman_prepend_candidate_to_path "$candidate_dir"
    fi
done
unset candidate_name candidate_dir
export PATH
```

它涉及到两个函数，具体如下

（1）`__sdkman_export_candidate_home`

添加新的环境变量

```shell
function __sdkman_export_candidate_home() {
	local candidate_name="$1"
	local candidate_dir="$2"
	local candidate_home_var="$(echo ${candidate_name} | tr '[:lower:]' '[:upper:]')_HOME"
	export $(echo "$candidate_home_var")="$candidate_dir"
}
```

（2）`__sdkman_prepend_candidate_to_path`

将环境变量添加到 PATH

```shell
function __sdkman_prepend_candidate_to_path() {
	local candidate_dir candidate_bin_dir

	candidate_dir="$1"
	candidate_bin_dir=$(__sdkman_determine_candidate_bin_dir "$candidate_dir")
	echo "$PATH" | grep -q "$candidate_dir" || PATH="${candidate_bin_dir}:${PATH}"
	unset CANDIDATE_BIN_DIR
}
```

当然这些环境变量只会在 git-bash 中生效

## 7.软链接问题

git-bash 创建软链接的方法 `ln -s` 无效，他只是简单的将文件进行了一个复制

方法一：

（1）想要解决这个问题，首先需要安装 [MSYS2](https://www.msys2.org/) 

（2）然后在 window 用户目录下的 `.bashrc` 文件添加一行代码，如下：

```shell
MSYS=winsymlinks:nativestrict
```

（3）同时还要打开 window 的开发者选项，这样就可以在 git-bash 使用软链接命令了。

**方法二：**★★ **推荐**

在 git 的 `usr/bin` 目录创建文件 `mkl.bat`

```bat
@echo off
setlocal EnableDelayedExpansion

set "linkPath=%1"
set "targetPath=%2"

if exist "!linkPath!" (
rem    rmdir /s /q "!linkPath!" >nul 2>&1
    rmdir /s /q "!linkPath!"
)

rem mklink /D "!linkPath!" "!targetPath!" >nul 2>&1
mklink /D "!linkPath!" "!targetPath!"
```

下载的 sdkman，***删除掉 libexec***（别人 rust 重写的软链接相关操作依旧不行）

找到 sdkman 中 src 目录下的 `sdkman-path-helpers.sh`，并修改源码

```shell
function __sdkman_link_candidate_version() {
	local candidate version

	candidate="$1"
	version="$2"

	# Change the 'current' symlink for the candidate, hence affecting all shells.
	if [[ -L "${SDKMAN_CANDIDATES_DIR}/${candidate}/current" || -d "${SDKMAN_CANDIDATES_DIR}/${candidate}/current" ]]; then
		rm -rf "${SDKMAN_CANDIDATES_DIR}/${candidate}/current"
	fi

	# ln -s "${version}" "${SDKMAN_CANDIDATES_DIR}/${candidate}/current"
	mkl.bat "${SDKMAN_CANDIDATES_DIR}/${candidate}/current" "${version}"
}
```

