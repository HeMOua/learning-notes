# SSH与Deployment

## SSH Configurations

1、首先打开settings -> Tools -> SSH Configurations

2、新建一个配置，输入Host和Username，端口默认都是22，这样这个配置的名称就默认由`Host@Username:port`构成了

3、最后就是选择授权方式了，总共有3中方式

+ Password：通过输入密码
+ Key pair：通过SSH密钥来进行授权，如果密钥设置了密码还需要输入passphrase
+ OpenSSH config and authentication agent：这个我只知道OpenSSH config的方式，就是使用`~/.ssh/config`文件中的配置来进行连接

## Deployment

1、选项卡栏Tools -> Deployment -> Configurations

2、然后可以自己添加分组以及主机

3、这时可以通过选择上一步配置的SSH Configuration来进行代码Deployment



最后最重要的一步就是选择Python编译器，首先在settings中找到编译器一栏，先新建一个SSH 的远程编译器，然后点击Show ALL，在其中的页面进行修改