# Linux学习

## 文件描述及权限

`-rw------- 1 root root` 是 Unix/Linux 系统中文件权限的一部分，通常通过命令如 `ls -l` 查看文件列表时显示。下面详细分析每个部分的含义：

（1）`-rw-------`

这是文件权限部分，分为四部分：

1. **`-`**：第一个字符表示文件类型。
   - `-` 表示是一个普通文件。
   - `d` 表示目录。
   - `l` 表示符号链接。
2. **`rw-------`**：这是文件的权限部分，共9个字符，分为3组，每组三个字符，分别表示**文件所有者（owner）**、**同组用户（group）**、和**其他用户（others）**的权限：
   - **`rw-`**：文件所有者（owner）的权限。`r` 表示读权限，`w` 表示写权限，`-` 表示没有执行权限。
   - **`---`**：同组用户（group）的权限。这里所有字符都是 `-`，表示没有读、写、执行的权限。
   - **`---`**：其他用户（others）的权限。与 group 类似，也没有任何权限。

（2）`1`

这是文件的**硬链接数**，即链接到该文件的硬链接数量。通常文件会有一个硬链接（自身）。

（3）`root root`

- 第一个 `root`：表示文件的**所有者**。在这个例子中，`root` 是系统的超级用户。
- 第二个 `root`：表示文件所属的**组**。这里也是 `root` 组，表示与该文件相关的用户组也是 root。

## 目录结构

**系统启动必须：**

- **/boot：**存放的启动Linux 时使用的内核文件，包括连接文件以及镜像文件。

- **/etc：**存放**所有**的系统需要的**配置文件**和**子目录列表，**更改目录下的文件可能会导致系统不能启动。

- **/lib**：存放基本代码库（比如c++库），其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。

- **/sys**： 这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。该文件系统是内核设备树的一个直观反映。当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中

**指令集合：**

- **/bin：**存放着最常用的程序和指令

- **/sbin：**只有系统管理员能使用的程序和指令。

**外部文件管理：**

- **/dev ：**Device(设备)的缩写, 存放的是Linux的外部设备。**注意：**在Linux中访问设备和访问文件的方式是相同的。

- **/media**：类windows的**其他设备，**例如U盘、光驱等等，识别后linux会把设备放到这个目录下。

- **/mnt**：临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。

**临时文件：**

- **/run**：是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。

- **/lost+found**：一般情况下为空的，系统非法关机后，这里就存放一些文件。

- **/tmp**：这个目录是用来存放一些临时文件的。

**账户：**

- **/root**：系统管理员的用户主目录。

- **/home**：用户的主目录，以用户的账号命名的。

- **/usr**：用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。

- **/usr/bin：**系统用户使用的应用程序与指令。

- **/usr/sbin：**超级用户使用的比较高级的管理程序和系统守护程序。

- **/usr/src：**内核源代码默认的放置目录。

**运行过程中要用：**

- **/var**：存放经常修改的数据，比如程序运行的日志文件（/var/log 目录下）。

- **/proc**：管理**内存空间！**虚拟的目录，是系统内存的映射，我们可以直接访问这个目录来，获取系统信息。这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件来做修改。

**扩展用的：**

- **/opt**：默认是空的，我们安装额外软件可以放在这个里面。

- **/srv**：存放服务启动后需要提取的数据**（不用服务器就是空）**

> `/usr/local` 目录通常用于存放系统本地安装的软件。这个目录下的结构类似于整个文件系统，有着 `bin`、`lib`、`share` 等子目录。下面是 `/usr/local` 目录结构的主要子目录及其作用：
>
> 1. **`/usr/local/bin`：** 存放本地系统安装的用户可执行程序（二进制文件），通常由系统管理员手动安装的软件会被放置在这里。
> 2. **`/usr/local/lib`：** 用于存放本地安装的库文件，类似于系统的 `/lib` 目录，但是这里存放的是管理员手动安装的软件所需的库。
> 3. **`/usr/local/include`：** 存放本地安装的软件所需的头文件，这些头文件通常在编译软件时使用。
> 4. **`/usr/local/share`：** 存放本地安装的软件所需的共享数据，如文档、图标等。
> 5. **`/usr/local/etc`：** 存放本地安装的软件的配置文件。
>
> 总体而言，`/usr/local` 是一个用于存放系统管理员手动安装的软件的常见目录，有助于与系统默认的软件包管理系统（例如 `apt`, `yum`, `pacman`等）分离。这样可以避免与系统自带软件包发生冲突，同时也方便管理本地安装的软件。

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

