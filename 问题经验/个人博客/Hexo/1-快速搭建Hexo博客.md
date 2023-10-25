## 快速部署

```shell
$ npm install hexo-cli -g
$ hexo init <folder>
$ cd <folder>
$ npm install
$ hexo server
```

新建完成后，指定文件夹的目录如下：

```
.
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
├── themes
├── _config.yml
├── package.json
```

其中：

+ `_config.yml`：网站的 [配置](https://hexo.io/zh-cn/docs/configuration) 信息，您可以在此配置大部分的参数。
+ `scaffolds`：[模版](https://hexo.io/zh-cn/docs/writing#模版（Scaffold）) 文件夹。当您新建文章时，Hexo 会根据 scaffold 来创建文件。
  + Hexo 的模板是指在新建的文章文件中默认填充的内容。
  + 例如，如果您修改 `scaffold/post.md` 中的 Front-matter 内容，那么每次新建一篇文章时都会包含这个修改。
+ `source`：资源文件夹是存放用户资源的地方。
  + 除 `_posts` 文件夹之外，开头命名为 `_`（下划线）的文件 / 文件夹和隐藏的文件将会被忽略。
  + Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。
+ `themes`：[主题](https://hexo.io/zh-cn/docs/themes) 文件夹。Hexo 会根据主题来生成静态页面。

## 常用命令

1. `init`：初始化文件夹。如果没有设置 `folder` ，`Hexo` 默认在目前的文件夹建立网站。

```shell
$ hexo init [folder]
//例如：
$ hexo init blog
```

2. `new`：新建一篇文章，如果标题包含空格的话，请使用引号括起来。

```shell
$ hexo new "title"
```

3. `generate`：生成静态文件。

```shell
$ hexo generate # 简写为 hexo g
```

4. `server`：启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。

```shell
$ hexo server # 简写为 hexo s
```

| 选项             | 描述                           |
| :--------------- | :----------------------------- |
| `-p`, `--port`   | 重设端口                       |
| `-s`, `--static` | 只使用静态文件                 |
| `-l`, `--log`    | 启动日记记录，使用覆盖记录格式 |


5. `deploy`：部署网站

```shell
$ hexo deploy # 简写 hexo d
```

| 参数               | 描述                     |
| :----------------- | :----------------------- |
| `-g`, `--generate` | 部署之前预先生成静态文件 |

6. `clean`：清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

```shell
$ hexo clean
```

7. `version`：显示 `Hexo` 版本。

```shell
$ hexo version
```



顺序：新建文章 -> generate 生成静态文件 -> deploy 部署网站

## 关联 Github Page

1、创建仓库，命名规则 `用户名.github.io`，必须是用户名加 github.io 后缀

2、复制好仓库地址，比如 `git@github.com:HeMOua/hemoua.github.io.git`

3、在博客目录安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)

```shell
$ npm install hexo-deployer-git --save
```

4、配置 `_config.yml`

```yaml
deploy:
  type: git
  repo:
    github: git@github.com:HeMOua/hemoua.github.io.git
  branch: branch_name
```

5、生成静态文件 `hexo generate`

6、部署 `hexo deploy`

这样可以在 github page 看到更改了

