# 搭建OpenVPN

## 1、安装软件

本教程的第一步是安装 OpenVPN 和 Easy-RSA。Easy-RSA 是一种公钥基础设施 (PKI) 管理工具，您将在 OpenVPN 服务器上使用它来生成证书请求，然后您将在 CA 服务器上进行验证和签名。

首先，更新您的 OpenVPN 服务器的软件包索引并安装 OpenVPN 和 Easy-RSA。这两个软件包都在 Ubuntu 的默认存储库中可用，因此您可以`apt`用于安装：

```bash
sudo apt update
sudo apt install openvpn easy-rsa
```

接下来，您需要在 OpenVPN 服务器上创建一个新目录，作为您的非 root 用户调用`~/easy-rsa`：

```bash
mkdir ~/easy-rsa
```

现在您需要从`easyrsa`包安装到`~/easy-rsa`您刚刚创建的目录中的脚本创建一个符号链接：

```bash
ln -s /usr/share/easy-rsa/* ~/easy-rsa/
```

> **注意：**虽然其他指南可能会指导您将`easy-rsa`包文件复制到 PKI 目录中，但本教程采用符号链接方法。因此，对`easy-rsa`包的任何更新都将自动反映在您的 PKI 脚本中。

最后，确保目录的所有者是您的非 root sudo 用户，并使用`chmod`以下命令限制对该用户的访问：

```bash
sudo chown sammy ~/easy-rsa
chmod 700 ~/easy-rsa
```

一旦这些程序被安装并移动到您系统上的正确位置，下一步是在 OpenVPN 服务器上创建一个公钥基础设施 (PKI)，以便您可以为客户端和其他服务器请求和管理 TLS 证书连接到您的 VPN。

> **复制的方法**：与软链接的方法不同，这里使用复制的方法
>
> 首先切换到root用户
>
> ```shell
> sudo su
> ```
>
> 然后创建文件夹并开始复制
>
> ```shell
> cp -r  /usr/share/easy-rsa/* ~/easy-rsa/
> ```

## 2、创建PKI

在创建 OpenVPN 服务器的私钥和证书之前，您需要在 OpenVPN 服务器上创建一个本地公钥基础设施目录。您将使用此目录来管理服务器和客户端的证书请求，而不是直接在您的 CA 服务器上进行。

要在您的 OpenVPN 服务器上构建 PKI 目录，您需要填充一个`vars`使用一些默认值调用的文件。首先您将`cd`进入该`easy-rsa`目录，然后您将`vars`使用 nano 或您喜欢的文本编辑器创建和编辑文件。

```bash
cd ~/easy-rsa
nano vars
```

打开文件后，粘贴以下两行：

~/easy-rsa/vars

```bash
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
```

这是`vars`OpenVPN 服务器上此文件中唯一需要的两行，因为它不会用作证书颁发机构。他们将确保您的私钥和证书请求配置为使用现代椭圆曲线加密 (ECC) 为您的客户端和 OpenVPN 服务器生成密钥和安全签名。

将 OpenVPN 和 CA 服务器配置为使用 ECC 意味着当客户端和服务器尝试建立共享对称密钥时，它们可以使用椭圆曲线算法进行交换。使用 ECC 进行密钥交换比使用带有经典 RSA 算法的普通 Diffie-Hellman 快得多，因为数字小得多，计算速度更快。

> **背景：**当客户端连接到 OpenVPN 时，他们使用非对称加密（也称为公钥/私钥）来执行[TLS 握手](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)。但是，在传输加密的 VPN 流量时，服务器和客户端使用对称加密，也称为共享密钥加密。
>
> 与非对称加密相比，对称加密的计算开销要小得多：使用的数字要小得多，而且现代 CPU[集成了指令来执行优化的对称加密操作](https://en.wikipedia.org/wiki/AES_instruction_set)。为了从非对称加密转换为对称加密，OpenVPN 服务器和客户端将使用[椭圆曲线 Diffie-Hellman (ECDH) 算法](https://en.wikipedia.org/wiki/Elliptic_curve_Diffie-Hellman)尽快就共享密钥达成一致。

填充`vars`文件后，您可以继续创建 PKI 目录。为此，请`easyrsa`使用该`init-pki`选项运行脚本。尽管您已经作为先决条件的一部分在 CA 服务器上运行了此命令，但有必要在此处运行它，因为您的 OpenVPN 服务器和 CA 服务器具有单独的 PKI 目录：

```bash
./easyrsa init-pki
```

请注意，在您的 OpenVPN 服务器上，无需创建证书颁发机构。您的 CA 服务器全权负责验证和签署证书。VPN 服务器上的 PKI 仅用作存储证书请求和公共证书的方便且集中的地方。

在 OpenVPN 服务器上初始化 PKI 后，您就可以继续下一步，即创建 OpenVPN 服务器证书请求和私钥。

## 3、创建 OpenVPN 服务器证书请求和私钥

现在您的 OpenVPN 服务器已安装所有先决条件，下一步是在您的 OpenVPN 服务器上生成私钥和证书签名请求 (CSR)。之后，您将请求转移到您的 CA 进行签名，创建所需的证书。获得签名证书后，您将把它传输回 OpenVPN 服务器并安装它以供服务器使用。

首先，以`~/easy-rsa`非 root 用户身份导航到OpenVPN 服务器上的目录：

```bash
cd ~/easy-rsa
```

现在，您将`easyrsa`使用该`gen-req`选项后跟机器的通用名称 (CN) 来调用。CN 可以是您喜欢的任何内容，但使其具有描述性会很有帮助。在本教程中，OpenVPN 服务器的 CN 将是`server`. 一定要包括该`nopass`选项。如果不这样做，将对请求文件进行密码保护，这可能会导致以后出现权限问题。

**注意：**如果您选择`server`此处以外的名称，则必须调整下面的一些说明。例如，将生成的文件复制到`/etc/openvpn`目录时，您必须替换正确的名称。您还必须`/etc/openvpn/server.conf`稍后修改该文件以指向正确的`.crt`和`.key`文件。

```bash
./easyrsa gen-req server nopass
```

```bash
OutputCommon Name (eg: your user, host, or server name) [server]:

Keypair and certificate request completed. Your files are:
req: /home/sammy/easy-rsa/pki/reqs/server.req
key: /home/sammy/easy-rsa/pki/private/server.key
```

这将为服务器创建一个私钥和一个名为`server.req`. 将服务器密钥复制到`/etc/openvpn/server`目录：

```bash
sudo cp /home/sammy/easy-rsa/pki/private/server.key /etc/openvpn/server/
```

完成这些步骤后，您已成功为 OpenVPN 服务器创建了私钥。您还为 OpenVPN 服务器生成了证书签名请求。CSR 现在已准备好由您的 CA 签名。在本教程的下一部分中，您将学习如何使用 CA 服务器的私钥签署 CSR。

## 4、签署 OpenVPN 服务器的证书请求

在上一步中，您为 OpenVPN 服务器创建了证书签名请求 (CSR) 和私钥。现在 CA 服务器需要了解`server`证书并对其进行验证。一旦 CA 验证证书并将其转发回 OpenVPN 服务器，信任您的 CA 的客户端也将能够信任 OpenVPN 服务器。

在 OpenVPN 服务器上，作为您的非 root 用户，使用 SCP 或其他传输方法将`server.req`证书请求复制到 CA 服务器进行签名：

```bash
scp /home/sammy/easy-rsa/pki/reqs/server.req sammy@your_ca_server_ip:/tmp
```

如果您遵循了本教程的先决条件 ，下一步是以您为管理 CA 而创建的非 root 用户身份登录到**CA 服务器。**您将`cd`进入`~/easy-rsa`创建 PK的目录，然后使用`easyrsa`脚本导入证书请求：

```bash
cd ~/easy-rsa
./easyrsa import-req /tmp/server.req server
```
```
Output. . .
The request has been successfully imported with a short name of: server
You may now use this name to perform signing operations on this request.
```

接下来，通过运行`easyrsa`带有`sign-req`选项的脚本来签署请求，后跟请求类型和通用名称。请求类型可以是`client`或`server`。由于我们正在处理 OpenVPN 服务器的证书请求，请务必使用`server`请求类型：

```bash
./easyrsa sign-req server server
```

> 如果提示错误，可以先构建 ca
>
> ```shell
> ./easyrsa build-ca nopass
> ```

在输出中，系统会提示您验证请求是否来自受信任的来源。输入`yes`然后按`ENTER`确认：

```
OutputYou are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 3650 days:

subject=
commonName = server


Type the word 'yes' to continue, or any other input to abort.
Confirm request details: yes
. . .
Certificate created at: /home/sammy/easy-rsa/pki/issued/server.crt
```

请注意，如果您加密了 CA 私钥，此时系统会提示您输入密码。

完成这些步骤后，您已经使用 CA 服务器的私钥签署了 OpenVPN 服务器的证书请求。生成的`server.crt`文件包含 OpenVPN 服务器的公共加密密钥，以及来自 CA 服务器的签名。签名的目的是告诉任何信任 CA 服务器的人，当他们连接到 OpenVPN 服务器时，他们也可以信任它。

（以下步骤可忽略）

要完成配置证书，请将`server.crt`和`ca.crt`文件从 CA 服务器复制到 OpenVPN 服务器：

```bash
scp pki/issued/server.crt sammy@your_vpn_server_ip:/tmp
scp pki/ca.crt sammy@your_vpn_server_ip:/tmp
```

现在回到您的 OpenVPN 服务器，将文件复制`/tmp`到`/etc/openvpn/server`：

```bash
sudo cp /tmp/{server.crt,ca.crt} /etc/openvpn/server
```

现在您的 OpenVPN 服务器几乎可以接受连接了。在下一步中，您将执行一些额外的步骤来提高服务器的安全性。

## 5、 配置 OpenVPN 加密材料

为了增加一层安全性，我们将添加一个额外的共享密钥，服务器和所有客户端将使用[OpenVPN 的`tls-crypt`指令](https://build.openvpn.net/doxygen/group__tls__crypt.html#details)。此选项用于混淆服务器和客户端最初相互连接时使用的 TLS 证书。OpenVPN 服务器也使用它来对传入的数据包执行快速检查：如果使用预共享密钥对数据包进行签名，则服务器会对其进行处理；如果它没有签名，那么服务器知道它来自不受信任的来源并且可以丢弃它而不必执行额外的解密工作。

此选项将有助于确保您的 OpenVPN 服务器能够应对未经身份验证的流量、端口扫描和拒绝服务攻击，这些攻击会占用服务器资源。这也使得识别 OpenVPN 网络流量变得更加困难。

要生成`tls-crypt`预共享密钥，请在 OpenVPN 服务器上的`~/easy-rsa`目录中运行以下命令：

```bash
cd ~/easy-rsa
openvpn --genkey --secret ta.key
```

结果将是一个名为`ta.key`. 复制到`/etc/openvpn/server/`目录：

```bash
sudo cp ta.key /etc/openvpn/server
```

将这些文件放在 OpenVPN 服务器上后，您就可以为您的用户创建客户端证书和密钥文件，您将使用它们连接到 VPN。

