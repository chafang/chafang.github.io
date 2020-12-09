---
title: 博客切换至 Github Pages
---
经过两天研究，自己的博客成功切换到 Github 上了。目前博客的主程序用了比较流行的 hexo 工具进行搭建，今天就简单说说基本步骤和部分问题的方法吧。

<!-- more -->

## 安装 npm

去官网下载，地址可以[点击这里](https://nodejs.org/en/download/current/)，根据自己当前系统的版本选择就可以了。

Windows 系统可以直接下载 msi 文件，正常安装就行了。Linux 可以下载二进制包，解压运行即可。

``` bash
wget https://nodejs.org/dist/v15.3.0/node-v15.3.0-linux-x64.tar.xz
xz -d node-v15.3.0-linux-x64.tar.xz
tar -xf node-v15.3.0-linux-x64.tar
mv node-v15.3.0-linux-x64 /usr/local/share/node-15.3.0
ln -s /usr/local/share/node-15.3.0/bin/node /usr/local/sbin/node
ln -s /usr/local/share/node-15.3.0/bin/npm /usr/local/sbin/npm
ln -s /usr/local/share/node-15.3.0/bin/npx /usr/local/sbin/npx

npm --version
```

如果出现了版本号，则表示已经正常安装了，否则就需要修改 `PATH` 配置

可以在 `/etc/bashrc` 的文件结尾添加 `PATH=$PATH:/usr/local/sbin` 后，重载文件

```bash
source /etc/bashrc
```

接下来，就可以安装 hexo 了。官网打开比较卡，[点击这里](https://hexo.bootcss.com/)可以看国内的版本。

## 安装 hexo

执行以下命令进行安装，参数 `--registry=https://registry.npm.taobao.org` 是使用阿里的 npm 镜像。

``` bash
npm install hexo-cli -g --registry=https://registry.npm.taobao.org
```

安装完毕后，会在 `/usr/local/share/node-15.3.0/bin` 目录下生成 `hexo` 可执行文件，我们可以将其添加到之前的 `bin` 文件夹中。

``` bash
ln -s /usr/local/share/node-15.3.0/bin/hexo /usr/local/sbin/hexo
```

接下来就可以正常使用 hexo 了。

## 使用 hexo

这里确保电脑上已经有 `git` 管理工具，如果没有的话可能会到时 hexo 初始化失败。

``` bash
mkdir blog && cd blog
hexo init
```

因为 hexo 的文件比较多，而且服务器在国外，导致初始化可能会异常缓慢。

大家泡杯茶，打打游戏，煲个电话粥啥的都行，安装成功后就跟着官方文档使用就行了，没啥大问题。

初始化后，运行 `npm install` 更新一下，别忘了用阿里的镜像源。

``` bash
npm install --registry=https://registry.npm.taobao.org
```

## 安装 NexT

不得不说，这主题确实还行，凑合用吧，我是懒得写了。

新主题的 Github 地址从 [这里](https://github.com/iissnan/hexo-theme-nex) 换到了 [这里](https://github.com/theme-next/hexo-theme-next)

所以，我们执行以下命令安装 NexT 主题，参数 `--depth=1` 的意思是只获取最新一次。

``` bash
git clone https://github.com/theme-next/hexo-theme-next themes/next --depth=1
```

接下来修改  `_config.yml` 文件的 `theme` 配置为 `next`，`language` 配置为 `zh-CN`，并重启本地服务器。

如果有报 `Cannot find module 'js-yaml'` 的错误，可安装一下该软件包。

``` bash
npm install js-yaml --registry=https://registry.npm.taobao.org
```

## Github 配置

首先新建一个仓库，

> 名称为：`username.github.io`
> 名称为：`username.github.io`
> 名称为：`username.github.io`

重要的事情说三遍。这也是和多年之前不同，导致耽误时间的原因。

其中 `username` 需要和你的 github 账号同名，具体官方信息可查看[这里](https://pages.github.com/)

接下来可使用 git 上传代码至仓库 `git@github.com:[username]/[username].github.io.git` 注意 `[username]` 的替换。

如果以上操作没有任何问题，代码会默认 `master` 分支，点击 Github 的当前项目代码设置，其中有个叫 `GitHub Pages` 的设置。

里面有一个 `source` 就能看到里面有 `master` 分支。

接下来我们回到本地代码，将 `_config.yml` 文件中的 `deploy` 设置为下面这样，注意 `username` 的替换。

``` yml
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: blog
```

其中 `branch` 的 `blog` 可以根据需要自己随意设置，它稍后将出现在 Github 仓库设置下的 `source` 中。我们在本地执行

``` bash
hexo g -d
```

hexo 程序将会自动将编译过的代码推送到 `blog` 分支。

我们刷新 Github 仓库的设置，找到 `GitHub Pages` 项，点开 `source`，就能看到刚刚推送的 blog 分支了。

选中，然后保存。然后看看能不能通过 `username.github.io` 进行访问。如果可以，就说明一切顺利。

## 发布说明和域名配置

hexo 系统的自动发布流程挺不错的，配置好后，本地一行命令就可以完成。发布代码和源代码互不干扰，属于两个分支，可以任意指定。

譬如我可以使用 master 分支进行文章的撰写，可以用 blog 分支进行发布，只需要在 `_config.yml` 文件配置即可。

打开 `Github` 的仓库设置，找到 `Custom domain` 修改为自己解析的域名，同时在域名解析里配置好 `cname` 到 `username.github.io` 。

点击保存，勾选 `Enforce HTTPS`，过几分钟可以使用 `ping` 命令检查是否解析过去了，接着就能进行访问了。

## 自定义域名消失

使用过程中，碰到一个棘手的问题就是 `Custom domain` 总是在执行发布后自动消失，需要重新绑定。

这时需要在当前项目中的 `source` 下新建一个名为 `CNAME` 的文件，将自己绑定的域名写进去，发布一下即可。

## 结语

hexo 还是很有意思的，有些东西需要一点点研究，但总之我不需要自己搭建数据库和配置了，hexo 自己维护自己的数据库。

最重要的是，博客空间不需要付费；博客空间不需要付费；博客空间不需要付费。

代码随时更新；随时发布；随处修改。
