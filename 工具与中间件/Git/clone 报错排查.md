## 1、`fatal: unable to access xxx: Failed to connect to xxx port 443 after xxx ms: Couldn't connect to server`

```
fatal: unable to access 'https://github.com/facebookresearch/iopath/': Failed to connect to github.com port 443 after 21169 ms: Couldn't connect to server
```

==可能是没配置代理，重点是配置 http ，配置 https 可能会报错（https不配置也行）==

在`~/.gitconfig` 中进行配置

```
[https "https://github.com"]
	proxy = socks5://127.0.0.1:7890

# 必须配置 http
[http "https://github.com"]           
	proxy = socks5://127.0.0.1:7890
```

## 2、`fatal: unable to access xxx: OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to xxx:443`

```
fatal: unable to access 'https://huggingface.co/GanymedeNil/text2vec-large-chinese/': OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to huggingface.co:443
```

==可能与SSL连接无法建立有关，这时可以先尝试使用 ssh 进行克隆，不要使用http协议了==

## 3、`fatal: Could not read from remote repository. Please make sure you have the correct access rights and the repository exists.`

```
git clone git@hf.co:GanymedeNil/text2vec-large-chinese
Cloning into 'text2vec-large-chinese'...
The authenticity of host 'hf.co (34.202.8.246)' can't be established.
ED25519 key fingerprint is SHA256:dVjzGIdV7d6cwKIeZiCoRMa2gMvSKfGZAvHf4gMiMao.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

==首先排查 ssh 连接的地址是否正确，有时 http 切换成 ssh 可能会修改域名。其次检查 ssh 秘钥的问题==

对于 ssh 的问题，首先要在远端配置 ssh 秘钥，如果还发生相同的报错，那可能就是没有定位到 ssh 秘钥，这时就需要在 `~/.ssh/config` 配置当前 ssh 连接所对应的秘钥

```
Host hf.co
  HostName hf.co
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
  TCPKeepAlive yes
```

4、`fatal: unable to access xxx: SSL certificate problem: unable to get local issuer certificate`

```
git clone https://git.devcloud.pcitc.com/chao.chen1/PPT_OCR.git
Cloning into 'PPT_OCR'...
fatal: unable to access 'https://git.devcloud.pcitc.com/chao.chen1/PPT_OCR.git/': SSL certificate problem: unable to get local issuer certificate
```

已确定此问题的两个潜在原因。

1. 无法验证自签名证书。
2. 默认 GIT 加密后端（Windows 客户端）



==解决方案1 - 自签名证书==

*（1）解决方案 - 临时*

告诉 git 不要使用 global 选项执行证书验证：

```none
git config --global http.sslVerify false
```

请注意，全局禁用 SSL 验证*可能会被视为安全风险*，应仅暂时实施

*（2）解决方案2 - 客户端*

1. 确保将根证书添加到 git.exe 的证书存储中。此文件的位置将取决于 GIT 的安装方式/位置。例如，Git Bash 的受信任证书存储目录为 *C：\Program Files\Git\mingw64\ssl\certs。* [此 Microsoft 博客](https://docs.microsoft.com/en-us/archive/blogs/phkelley/adding-a-corporate-or-self-signed-certificate-authority-to-git-exes-store)上也对此进行了讨论。

2. 告诉 Git 在何处可以找到 CA 捆绑包，方法是运行：

   ```none
   git config --system http.sslCAPath /absolute/path/to/git/certificates
   ```

   

   其中 `/absolute/path/to/git/certificates` 是放置包含 CA 证书的文件的路径。

   或者将 CA 捆绑包复制到 `/bin` 目录，并将以下内容添加到 `gitconfig` 文件中：

   ```none
   sslCAinfo = /bin/curl-ca-bundle.crt
   ```

3. 重新安装 Git。

4. 确保 CA 捆绑包文件中存在完整的证书链，包括根证书。

*（3）解决方案 - 服务器端*

使用 SSL 终止连接器而不是代理保护 Bitbucket Server 的配置也可能发生此问题

1. 确保 Java KeyStore 具有完整的证书链（中间 CA 和根 CA）
   - 使用 [KeyStore Explorer](https://keystore-explorer.org/) 等工具查看 KeyStore 中的证书链详细信息，以检查

==解决方案2 - 默认 GIT 加密后端==（不行）

使用 Windows 时，问题在于 git 默认使用 “Linux” 加密后端，因此 GIT 操作可能偶尔无法完成。从 Git for Windows 2.14 开始，您可以将 Git 配置为使用 SChannel（内置 Windows 网络层）作为加密后端。为此，只需在 GIT 客户端中运行以下命令：

```none
git config --global http.sslbackend schannel
```

这意味着它将使用 Windows 证书存储机制，并且您 不需要显式配置 curl CA 存储 （`http.sslCAInfo`） 机制。
