# Linux学习

## 目录结构

+ `/`：这就是根目录，想必地球人都知道了。对于你的电脑来说，有且只有一个根目录，所有的文件都是从这里开始的。举个例子：当你在终端里输入“/home”，你其实是在告诉电脑，先从/（根目录）开始，再进入到home目录。

+ `/root`：系统管理员（root user）的目录。至于系统管理员的权限都多大我这里就不在废话了。因此，请小心使用root帐号。

+ `/boot`：系统启动文件，所有与系统启动有关的文件都保存在这里 。

+ ==/bin==：全称binary，含义是二进制。该目录中存储的都是一些二进制文件，文件都是可以被运行的。这里是存放系统的程序。

+ `/etc`：主要存放了系统配置方面的文件。

+ `/dev`：这里主要存放与设备（包括外设）有关的文件（unix和linux系统均把设备当成文件）。想连线打印机吗？系统就是从这个目录开始工作的。另外还有一些包括磁盘驱动、USB驱动等都放在这个目录。

+ ==/home==：这里主要存放你的个人数据。具体每个用户的设置文件，用户的桌面文件夹，还有用户的数据都放在这里。每个用户都有自己的用户目录，位置为：/home/用户名。当然，root用户除外。

+ `/tmp`：这是临时目录。对于某些程序来说，有些文件被用了一次两次之后，就不会再被用到，像这样的文件就放在这里。因此，千万不要把重要的数据放在这里哦。

+ `/usr`：存放的是用户自己安装的软件。类似于windows下的program files | 在这个目录下，你可以找到那些不适合放在/bin或/etc目录下的额外的工具。比如像游戏阿，一些打印工具等等。/usr目录包含了许多子目录：/usr /bin目录用于存放程序；/usr/share用于存放一些共享的数据，比如音乐文件或者图标等等；/usr/lib目录用于存放那些不能直接运行的， 但却是许多程序运行所必需的一些函数库文件。你的软件包管理器会自动帮你管理好/usr目录的。系统级的目录，可以理解为C:/Windows/。

+ `/opt`：opt目录用来安装附加软件包，是用户级的程序目录，可以理解为D:/Software。安装到/opt目录下的程序，它所有的数据、库文件等等都是放在同个目录下面。opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。

+ `/usr/local`：用户级的程序目录，可以理解为C:/Progrem Files/，这里主要存放那些手动安装的软件，即不是通过“新立得”或apt-get安装的软件。它和/usr目录具有相类似的目录结构。让软件包管理器来管理/usr目录，而把自定义的脚本（scripts）放到/usr/local目录下面。

+ `/media`：这个目录是用来挂载那些usb接口的移动硬盘（包括U盘）、CD/DVD驱动器等等。
  其实安装软件程序并不是非要在指定的目录下完成，安装java、tomcat等也可以安装在opt目录下，但是安装程序的扩展性和管理性来说，方便使用才是最好的。总结opt目录和usr/local目录就是/usr/local下一般是你安装软件的目录，这个目录就相当于在windows下的programefiles这个目录。/opt这个目录是一些大型软件的安装目录，或者是一些服务程序的安装目录 。

## 指令与选项

+ **pwd - Print current working driectory**

+ **ls - List directory conte**

+ **cd - change directory**

+ **mkdir - make directories**

+ **touch - change file timestamps**

+ **rm - remove files or directories**

  + **作用**：删除文件/目录
  + **用法1**：rm [选项] 需要移除的文件路径

  ```shell
  # 删除当前路径下的myfile文件
  rm myfile 
  # 删除/usr路径下的myfile文件
  rm /usr/myfile
  ```

  + **用法2**：rm [选项] 需要移除的目录

  ```shell
  # 删除当前路径下的abc文件
  rm -rf myfolder
  # 删除/usr路径下的abc文件
  rm -rf /usr/myfolder
  ```

+ **cp - copy files and directories**

  + **作用**：复制文件/文件夹到指定的位置

  + **用法1**：cp [被复制的文件路径]  [文件被复制到的路径]

  ```shell
  # cp命令来复制一个文件
  cp /home/bing/myfile ./
  ```

  + **用法2**：cp -r  [被复制的文件夹路径]  [文件夹被复制到的路径]

  + **含义**：-r 表示递归复制，复制文件夹的时候需要加 -r

  ```shell
  # 复制/home/bing/myfolder文件夹到根目录/下
  cp -r /home/bing/myfolder /
  ```

+ **mv** - move (rename) files

+ **man** - an interface to the system reference manuals

+ **reboot** -  reboot the machine
  + **作用**：重启linux系统
  + **用法**：reboot

+ **shutdown** - power-off the machine
  
  + **作用**：关机
  + **用法**：shut -h [时间]
  
  ```shell
  # 立即关机
  shutdown -h now
  ```
  

## 网络设置

**修改主机名**

```cmd
hostnamectl # 查看主机信息
hostnamectl [set-hostname 主机名] # 设置主机名
```

**固定ip**

```shell
nmcli connection show  # 查看所有网卡信息
nmcli connection show [name|uuid] # 查看指定网卡信息
nmcli connection modify [uuid]  \  # 修改网络配置
  ipv4.method manual \
  ipv4.address 192.168.56.89 \
  ipv4.gateway 192.168.56.1 \
  ipv4.dns 192.168.56.1
```

## 解压缩

```markdown
* tar -xvf 解压文件   
	解压后的文件放在当前目录中
* tar -zxvf 压缩文件 –C指定目录
	解压后的文件放入指定目录中，注意-C是大写字母C，后面不用接空格符
	
* unzip
	+ unzip test.zip 				->将test.zip解压到当前文件下
	+ unzip -n test.zip -d /tmp 	->将test.zip解压到/tmp目录下，并且不要覆盖已有文件
	+ unzip -v test.zip				->查看test.zip内容，但不解压
	+ unzip -o test.zip -d tmp/	->将test.zip解压到/tmp目录下，并且覆盖已有文件
```

