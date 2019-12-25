## apt命令

> APT全称Advanced Packaging Tool，可以自动下载，配置，安装二进制或者源代码格式的软件包，因此简化了Linux系统上管理软件的过程。现在Debian和其衍生发行版（如 Ubuntu）中都包含了APT。

**APT命令（package 为软件包名称）**：

apt-cache search package 搜索包

apt-cache show package 获取包的相关信息，如说明、大小、版本等

sudo apt-get install package 安装包

sudo apt-get install package –reinstall 重新安装包

sudo apt-get -f install 强制安装

==sudo apt-get purge package 先清除原先的再安装==

sudo apt-get remove package 删除包

sudo apt-get remove package –-purge 删除包，包括删除配置文件等

sudo apt-get autoremove 自动删除不需要的包

sudo apt-get update 更新源

sudo apt-get upgrade 更新已安装的包

sudo apt-get dist-upgrade 升级系统

sudo apt-get dselect-upgrade 使用 dselect 升级

apt-cache depends package 了解使用依赖

apt-cache rdepends package 了解某个具体的依赖

sudo apt-get build-dep package 安装相关的编译环境

apt-get source package 下载该包的源代码

sudo apt-get clean && sudo apt-get autoclean 清理下载文件的存档

sudo apt-get check 检查是否有损坏的依赖



除了要了解基本的apt指令外，还需要了解下一些与APT相关的文件，具体每个文件作用如下

/etc/apt/sources.list 设置软件包的获取来源

/etc/apt/apt.conf apt配置文件

/etc/apt/apt.conf.d apt的零碎配置文件

/etc/apt/preferences 版本参数

/var/cache/apt/archives/partial 存放正在下载的软件包

/var/cache/apt/archives 存放已经下载的软件包

/var/lib/apt/lists 存放已经下载的软件包详细信息

/var/lib/apt/lists/partial 存放正在下载的软件包详细信息

## DPKG命令

> dpkg是Debian软件包管理器的基础，被用于安装、卸载和供给和.deb软件包相关的信息。dpkg本身是一个底层的工具，本身并不能从远程包仓库下载包以及处理包的依赖的关系，需要将包从远程下载后再安装。

**DPKG常用命令**：

dpkg -i package.deb 安装包

dpkg -r package 删除包

dpkg -P package 删除包（包括配置文件）

dpkg -L package 列出与该包关联的文件

dpkg -l package 显示该包的版本

dpkg –unpack package.deb 解开 deb 包的内容

dpkg -S keyword 搜索所属的包内容

dpkg -l 列出当前已安装的包

dpkg -c package.deb 列出 deb 包的内容

dpkg –configure package 配置包

注意：更多选项可通过 dpkg -h 查询，有些指令需要超级用户权限才能执行。





