---
title: 博客成功切换至 github pages
---
经过两天研究，自己的博客成功切换到 github 上了。目前博客的主程序用了比较流行的 hexo 工具进行搭建，之前没有怎么详细研究过，今天就简单说说步骤和遇到的坑吧。 

## 基本要求

### 安装 npm

去官网下，可以使用二进制包，方便快捷。这里就不多说了。地址可以[点击这里](https://nodejs.org/en/download/current/)，根据自己当前系统的版本选择就可以了。

我这里介绍一下 linux 环境下的安装

``` bash
$ hexo new "My New Post"
```

安装好后，就可以跟着教程进行 hexo 的安装了。官网打开比较卡，[点击这里](https://hexo.bootcss.com/)可以看国内的版本。

### 安装 hexo

``` bash
$ npm install hexo-cli
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
