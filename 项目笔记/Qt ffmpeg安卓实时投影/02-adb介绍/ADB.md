# Android 调试桥
>https://developer.android.com/studio/command-line/adb

>https://github.com/mzlogin/awesome-adb

Android 调试桥 (adb) 是一个通用命令行工具，其允许您与模拟器实例或连接的 Android 设备进行通信。它可为各种设备操作提供便利，如安装和调试应用，并提供对 Unix shell（可用来在模拟器或连接的设备上运行各种命令）的访问。该工具作为一个客户端-服务器程序，包括三个组件：

- adb客户端，该组件发送命令。adb客户端在**开发计算机**上运行。我们可以通过从命令行运行adb客户端来发送命令。
- adb服务器，该组件管理客户端和后台程序之间的通信。adb服务器在**开发计算机**上作为后台进程运行。
- adbd服务器，该组件在设备上运行命令。adbd服务器在每个**模拟器或设备实例**上作为后台进程运行。

以下是 ADB 和 Fastboot 的谷歌官方下载链接：

ADB和Fastboot for Windows

https://dl.google.com/android/repository/platform-tools-latest-windows.zip

ADB和Fastboot for Mac

https://dl.google.com/android/repository/platform-tools-latest-darwin.zip

ADB和Fastboot for Linux

https://dl.google.com/android/repository/platform-tools-latest-linux.zip

由于这些是直接的 Google 链接，用户可以确保下载不仅是官方的，而且将始终能够获得最新版本的 ADB 和 Fastboot 。

adb 的工作方式
---
启动一个 adb 客户端时，此客户端首先检查是否有已运行的 adb 服务器进程。如果没有，它将启动服务器进程。当服务器启动时，它与本地 TCP 端口 5037 绑定，并侦听从 adb 客户端发送的命令，**所有 adb 客户端均使用端口 5037 与 adb 服务器通信**。

![adb原理](http://aliyunzixunbucket.oss-cn-beijing.aliyuncs.com/jpg/30c68fad7f038acf3948f98799105cbb.jpg?x-oss-process=image/resize,p_100/auto-orient,1/quality,q_90/format,jpg/watermark,image_eXVuY2VzaGk=,t_100)
![adb原理详细](http://aliyunzixunbucket.oss-cn-beijing.aliyuncs.com/jpg/11b093e21f4ec9c27b9a3a8783c8140b.jpg?x-oss-process=image/resize,p_100/auto-orient,1/quality,q_90/format,jpg/watermark,image_eXVuY2VzaGk=,t_100)

在您的设备上启用 adb 调试
---
要在通过 USB 连接的设备上使用 adb，您必须在设备系统设置中启用 USB debugging（位于 Developer options 下）。

在运行 Android 4.2 及更高版本的设备上，Developer options 屏幕默认情况下处于隐藏状态。如需将其显示出来，请转到 Settings > About phone 并点按 Build number 七次。返回上一屏幕，在底部可以找到 Developer options。

在某些设备上，Developer options 屏幕所在的位置或命名方式可能有所不同。

现在，您可以将设备与 USB 连接。可以从 android_sdk/platform-tools/ 目录执行 adb devices 来验证设备是否连接。如果已连接，您将看到设备名称以“设备”形式列示。

> 注：当您连接运行 Android 4.2.2 或更高版本的设备时，系统将显示一个对话框，询问您是否接受允许在这台计算机上调试的 RSA 密钥。这种安全机制可以保护用户设备，因为它可以确保只有在您能够解锁设备并确认对话框的情况下才能执行 USB 调试和其他 ADB 命令。

基本用法
---
### 命令语法
adb 命令的基本语法如下：

`adb [-d|-e|-s <serialNumber>] <command>`

如果只有一个设备/模拟器连接时，可以省略掉 [-d|-e|-s <serialNumber>] 这一部分，直接使用 adb <command>。
### 为命令指定目标设备
如果有多个设备/模拟器连接，则需要为命令指定目标设备。
参数|含义
--|:--
-d|指定当前唯一通过 USB 连接的 Android 设备为命令目标
-e|指定当前唯一运行的模拟器为命令目标
-s <serialNumber>|指定相应 serialNumber 号的设备/模拟器为命令目标
在多个设备/模拟器连接的情况下较常用的是 -s <serialNumber> 参数，serialNumber 可以通过 **adb devices** 命令获取。如：
```
$ adb devices

List of devices attached
cf264b8f	device
emulator-5554	device
10.129.164.6:5555	device
```
输出里的 cf264b8f、emulator-5554 和 10.129.164.6:5555 即为 serialNumber。

比如这时想指定 cf264b8f 这个设备来运行 adb 命令获取屏幕分辨率：

`adb -s cf264b8f shell wm size`

adb常用命令
---
- 查看已连接的模拟器/设备的列表
  
    `adb devices`
    
- 查看所有建立的反向代理

    `adb reverse --list`

- 将命令发送至特定设备

    `adb -s serial_number command `

    如果在多个设备可用时您未指定目标模拟器/设备实例就发出命令，那么 adb 将生成一个错误。

- 安装应用

    `adb install path_to_apk`
    
- 设置端口转发

    您可以使用 reserve 命令设置任意端口转发 — 将对模拟器/设备实例上特定端口的请求转发到主机的其他端口。下面向您介绍如何设置模拟器/设备端口 6100 到主机端口 7100 的转发：
    
    `adb reserve tcp:6100 tcp:7100`
    
    您也可以使用 adb 设置传输到指定的UNIX域套接字的转发，如下所示：
    
    `adb reserve localabstract:logd tcp:7100`
    
    [unix域套接字介绍](https://www.cnblogs.com/pengdonglin137/p/3849012.html)
    
- 将文件复制到设备/从设备复制文件

    您可以使用 adb 命令 pull 和 push 将文件复制到模拟器/设备实例或从其中复制文件。与 install 命令不同（其仅将 APK 文件复制到特定位置），pull 和 push 命令允许您将任意目录和文件复制到模拟器/设备实例中的任意位置。

    要从模拟器或设备复制文件或目录（及其子目录），请使用
    
    `adb pull remote local`
    
    要将文件文件或目录（及其子目录）复制到模拟器或设备，请使用
    
    `adb push local remote`
    
    在上述命令中，local 和 remote 指的是开发计算机（本地）和模拟器/设备实例（远程）上目标文件/目录的路径。例如：

    `adb push foo.txt /sdcard/foo.txt`
    
- 停止 adb 服务器

    在某些情况下，您可能需要终止 adb 服务器进程，然后重启它以解决问题（例如，如果 adb 不响应命令）。
    要停止 adb 服务器，请使用 `adb kill-server` 命令。然后，您可以通过发出任意其他 adb 命令重启服务器。
    
- 发出 shell 命令

    您可以使用 shell 命令通过 adb 发出设备命令，可以进入或不进入模拟器/设备实例上的 adb 远程 shell。要在不进入远程 shell 的情况下发出一个命令，请使用如下 shell 命令：

    `adb [-d|-e|-s serial_number] shell shell_command`
    
    或者，使用如下命令进入模拟器/设备实例上的远程 shell：

    `adb [-d|-e|-s serial_number] shell`
    
    当您准备退出远程 shell 时，按 Control + D 或输入 exit。
    shell 命令二进制文件存储在模拟器或设备的文件系统中，其路径为 /system/bin/。
通过 WLAN 连接到设备
---
一般情况下，通过 USB 使用 adb。不过，也可以按照下面的说明通过 WLAN 使用它。

- 将 Android 设备和 adb 主计算机连接到这两者都可以访问的常用 WLAN 网络。请注意，并非所有访问点均适用；您可能需要使用已正确配置防火墙的访问点以支持 adb 的访问点。

- 使用 USB 电缆将设备连接到主计算机。
- 设置目标设备以侦听端口 5555 上的 TCP/IP 连接。

    `$ adb tcpip 5555`

- 从目标设备断开 USB 电缆连接。
- 查找 Android 设备的 IP 地址。例如，在 Nexus 设备上，您可以通过访问 Settings > About tablet（或 About phone) > Status > IP address 查找 IP 地址。或者，在 Android Wear 设备上，您可以通过访问 Settings > Wi-Fi Settings > Advanced > IP address 查找 IP 地址。
- 连接至设备，通过 IP 地址识别此设备。

    `$ adb connect device_ip_address`
    
- 请确认您的主计算机已连接至目标设备：

    `$ adb devices`
    List of devices attached
    device_ip_address:5555 device

现在，您可以开始操作了！

如果 adb 连接丢失：

请确保您的主机仍与您的 Android 设备连接到同一个 WLAN 网络。
通过再次执行 adb connect 步骤重新连接。
如果无法连接，则重置 adb 主机：
adb kill-server
然后，从头开始操作。