---
title: CentOS 8 初始化设置
date: 2020-12-07 19:52:09
---

CentOS 的初始化工作比较琐碎，整理一些常用的脚本以后使用，一些设置如果不用可以略过，长期更新。

<!-- more -->

## 配置网络

``` bash
# 查看系统网卡名称
ip addr

# 编辑网络配置，我的是 ifcfg-ens192，每台电脑的可能不同
vi /etc/sysconfig/network-scripts/ifcfg-ens192
```

其中需检查以下参数是否设置正确

``` bash
BOOTPROTO=none # dhcp 或 static，对应着自动获取 或 静态IP
ONBOOT=yes # 开机自启动
IPADDR=192.168.0.101 # 当前绑定的 IP 是多少
NETMASK=255.255.255.0 # 子网掩码，CentOS8 改成了 PREFIX="24"
GATEWAY=192.168.0.1 # 当前网关
DNS1=192.168.0.1 # DNS1 地址
DNS2=114.114.114.114
```

保存退出，重启网卡
``` bash
ifdown ens192
ifup ens192
```

如果可以正常 `ping` 通外网，说明配置成功。

## 远程登陆

CentOS 8 已默认安装了 ssh 服务，如果没有安装，可使用 `yum install openssh-server` 进行安装。

``` bash
vim /etc/ssh/sshd_config

# 更改配置
PPermitRootLogin no # 是否运行 root 登陆
TCPKeepAlive yes # 是否运行长链接
ClientAliveInterval 60 # 客户端心跳时间
ClientAliveCountMax 10 # 客户端最大次数

# 启动服务
service sshd start

# 重载服务
service sshd reload
```

现在可使用 ssh 登陆了。

## 更换镜像源

更换 CentOS 的镜像源，推荐使用[中科大](http://mirrors.ustc.edu.cn/help/centos.html)的，更新的比较及时。

``` bash
# CentOS8
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-Base.repo \
         /etc/yum.repos.d/CentOS-Extras.repo \
         /etc/yum.repos.d/CentOS-AppStream.repo

# CentOS7
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/centos|baseurl=https://mirrors.ustc.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-Base.repo

# 更新缓存
yum makecache
```

## 更新和安装常用包

```
yum -y update

# 安装其他的源和包
yum -y install epel-release
yum -y install psmisc git net-snmp httpd-tools curl-devel openssl openssl-devel libxml2 libxml2-devel pcre pcre-devel libffi-devel ncurses-devel htop net-tools vim openssh-server openssh-clients net-tools

# 安装开发工具，可运行 yum grouplist 查看
yum -y groupinstall "Development Tools"
```

## 系统基本设置

修改主机名称，需重启生效

``` bash
hostnamectl set-hostname [hostname]
```

修改 shell 提示符样式可在 `/etc/profile` 新增一行 `export PS1='\u@\h:\w \$ '`

``` bash
source /etc/profile
```

关闭防火墙、清空路由规则

``` bash
systemctl stop firewalld
systemctl disable firewalld

iptables -P INPUT ACCEPT
iptables -F
```

关闭 `SELinux` 模式，需修改 `/etc/selinux/config` 中的 `SELINUX` 的值为 `disabled` 即可，需重启生效

## 用户相关

新增用户并设置密码

``` bash
adduser [username]
passwd [username]
```

设置用户运行 sudo 跳过密码，重新登录生效

``` bash
visudo

# 找到相关行，添加以下代码
[username] ALL=(ALL) NOPASSWD: ALL
```

## 配置 git

设置 git 基本信息

``` bash
git config --global user.name "name"
git config --global user.email "email"

git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch

# 设置 ssh-key
ssh-keygen -t rsa -C "email"
```

## 免密码登录

以下代码需在本地计算机运行

``` bash
ssh-copy-id username@ip

# 本地新建文件
vim ~/.ssh/config

# 设置名称
Host name
HostName ip
User username
IdentitiesOnly yes

# 名称用空行隔开
```
