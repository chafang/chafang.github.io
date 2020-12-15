---
title: gitlab 安装与初始化配置
date: 2020-12-10 09:31:32
tags:
---

企业代码仓库放在第三方肯定是不太安全的，所以 gitlab 是非常好的本地化部署解决方案。

<!-- more -->

## 新增 Gitlab 仓库并安装

``` bash
vim /etc/yum.repos.d/Gitlab-ce.repo

# 添加以下内容
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1

# 更新 yum 缓存
yum makecache

# 安装 gitlab 社区版
yum install gitlab-ce
```

当然也可以下载官方的 rpm 包，地址[点击这里](https://packages.gitlab.com/gitlab/gitlab-ce)

## Gitlab 配置

``` bash
vim /etc/gitlab/gitlab.rb

# 修改 Gitlab 配置
external_url 'http://192.168.0.101' # 当前的IP或者绑定的域名

gitlab_rails['time_zone'] = 'Asia/shanghai' # 时区设置

gitlab_rails['gitlab_email_enabled'] = true # 这里相当于管理员的邮箱，设置自己的即可
gitlab_rails['gitlab_email_from'] = 'example@example.com'
gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'

# 这里填写自己的配置
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.server"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "smtp user"
gitlab_rails['smtp_password'] = "smtp password"
gitlab_rails['smtp_domain'] = "smtp.server"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
```

## 启动 Gitlab
``` bash
# Gitlab 初始化配置，第一次会比较漫长
gitlab-ctl reconfigure

# 确认已关闭防火墙和路由配置
systemctl stop firewalld
systemctl disable firewalld

iptables -P INPUT ACCEPT
iptables -F

# Gitlab 启动
gitlab-ctl start

# Gitlab 关闭
gitlab-ctl stop
```

## 内网穿透

下载 FRP 穿透工具，可以 [点击这里](https://github.com/fatedier/frp/releases) 进行版本的选择，这里就用 linux 版本进行演示。

具体可查看 [Frp中文文档](https://gofrp.org/) 进行配置。
