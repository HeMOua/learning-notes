# 如何拉取 HuggingFace 模型

## 配置 Git 代理

先在 `~/.gitconfig` 临时配置一下 Git 的代理

```ini
[http]
	proxy = socks5://127.0.0.1:7890
[https]
	proxy = socks5://127.0.0.1:7890
```

拉取项目结束后，将上面的配置删除

## clone 时不下载 LFS

1、cmd 输入命令，设置临时 git clone 不会下载 lfs 文件

```cmd
set GIT_LFS_SKIP_SMUDGE=1
```

2、配置好以上命令

+ 使用 `git clone <repository-url>` 克隆项目
+ 使用 `git clone --depth 1 <repository-url>`，这种方式只克隆最新的提交历史，省略整个历史记录

## 下载 LFS

```shell
git lfs pull
```

这样可以显示下载进度

