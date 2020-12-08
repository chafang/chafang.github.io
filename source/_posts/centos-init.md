---
title: CentOS 8 初始化设置
date: 2020-12-07 19:52:09
---

github支持githubPage静态界面来搭建我们的个人博客，自己配置。

<!-- more -->

更新系统
----

```
yum -y update

# 安装开发工具，可运行 yum grouplist 查看
yum -y groupinstall "Development Tools"

# 若报 “No packages in any requested group available to install or update”，尝试运行以下进行安装
yum -y group install "Development tools" --setopt=group_package_types=mandatory,default,optional

# 安装其他的源和包
yum -y install epel-release
yum -y install psmisc git net-snmp httpd-tools curl-devel openssl openssl-devel libxml2 libxml2-devel pcre pcre-devel libffi-devel ncurses-devel htop net-tools vim openssh-server openssh-clients net-tools

# 安装 PHP 可能用到的包
yum -y install bzip2 bzip2-devel libzip libzip-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel libicu libicu-devel sqlite sqlite-devel oniguruma oniguruma-devel libevent libevent-devel libpng-devel libwebp-devel libjpeg-devel icu libicu-devel libxslt-devel
```

以上可直接使用 shell 脚本直接运行，需注意的是最好更换国内镜像源。


基本配置
----

新增用户
```
adduser [username]
passwd [username]
```

设置普通用户运行 sudo 跳过密码，重新登录 shell 生效
```
visudo

# 找到相关行，添加以下代码
[username] ALL=(ALL) NOPASSWD: ALL

# 保存
```

修改 sshd 配置
```
vim /etc/ssh/sshd_config

# 更改配置
PPermitRootLogin no # 是否运行 root 登陆
TCPKeepAlive yes # 是否运行长链接
ClientAliveInterval 60 # 客户端心跳时间
ClientAliveCountMax 10 # 心跳最大次数

# 保存

# 重新加载配置
service sshd reload
```

修改主机名称（可选），重启生效
```
hostnamectl set-hostname [hostname]
vim /etc/profile
```

修改 shell 提示符样式（可选）
```
vim /etc/profile

# 新增一行
export PS1='\u@\h:\w \$ '

# 保存

source /etc/profile
```

关闭防火墙
```
systemctl stop firewalld
systemctl disable firewalld
```

配置 IpTables
```
# 接收所有转发至本机的数据包
iptables -P INPUT ACCEPT

# 清空所有 rule 规则
iptables -F
```

配置 git
```
git config --global user.name "name"
git config --global user.email "email"

git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch

# 设置 ssh-key
ssh-keygen -t rsa -C "email"

# 设置 push 方式
git config --global push.default simple
```

免密码登录 SSH
----
以下代码需在本地计算机运行
```
ssh-copy-id username@ip

# 没有则新建文件
vim ~/.ssh/config

# 设置名称
Host alias-name
HostName ip
User username
IdentitiesOnly yes

# 保存，多个 ssh 用空行隔开
```

以上配置完成


安装 Nginx 服务
----
```
# 准备用户和组
groupadd nginx

# 禁止用户登录
useradd nginx -g nginx -s /sbin/nologin -M
```

官网下载 [Download](http://nginx.org/en/download.html "nginx")

```
tar -zxf nginx-latest.tar.gz
cd nginx-latest

# 可使用 ./configure --help 查看帮助和相关插件
./configure --user=nginx --group=nginx --prefix=/usr/local/share/nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_auth_request_module --with-http_secure_link_module --with-http_stub_status_module

make && make install

# nginx 可使用 ln 做软连接至 /usr/local/sbin 中
ln -s /usr/local/share/nginx/sbin/nginx /usr/local/sbin/nginx

# 检查 nginx 配置
nginx -t

# 启动 nginx 服务
nginx

# 关闭或重载 nginx 服务
nginx -s stop
nginx -s reload

```

配置 nginx 验证功能（可选）
```
htpasswd -c /usr/local/share/nginx/conf/ptList [username]

# 在 server 或 location 配置中添加，htpasswd 只支持目录认证
auth_basic "Please input password";
auth_basic_user_file /usr/local/share/nginx/conf/ptList;
```

安装 php-fpm 服务
----
官网下载 [Download](https://www.php.net/downloads "php")

```
tar -zxf php-latest.tar.gz
cd php-latest

# 可使用 ./configure --help 查看帮助和相关插件
./configure --prefix=/usr/local/share/php --with-fpm-user=nginx --with-fpm-group=nginx --enable-fpm --disable-fileinfo --with-curl --with-gettext --with-iconv-dir --enable-intl --with-mysqli --with-openssl --with-pdo-mysql --with-pdo-sqlite --with-pear --with-xmlrpc --with-xsl --with-zlib --with-bz2 --with-mhash --enable-bcmath --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-sysvshm --enable-xml --enable-gd --with-webp --with-jpeg

# 以上配置安装时可能会报各种包版本问题或找不到，可搜索相关报错信息进行修复，部分常见错误可见附加列表。

make && make test && make install

# 设置软链接
ln -s /usr/local/share/php/sbin/php-fpm /usr/local/sbin/php-fpm
ln -s /usr/local/share/php/bin/php /usr/local/sbin/php
ln -s /usr/local/share/php/bin/pecl /usr/local/sbin/pecl
ln -s /usr/local/share/php/bin/php-config /usr/local/sbin/php-config
ln -s /usr/local/share/php/bin/phpize /usr/local/sbin/phpize
ln -s /usr/local/share/php/bin/php-cgi /usr/local/sbin/php-cgi

# 复制配置文件
cp php.ini-* /usr/local/share/php/lib/
cp /usr/local/share/php/lib/php.ini-production /usr/local/share/php/lib/php.ini
cp /usr/local/share/php/etc/php-fpm.conf.default /usr/local/share/php/etc/php-fpm.conf
cp /usr/local/share/php/etc/php-fpm.d/www.conf.default /usr/local/share/php/etc/php-fpm.d/www.conf

# 验证配置
php-fpm -t

# 启动服务
php-fpm -D

# 平滑退出
ps aux|grep -E "php-fpm:\ master\ process"|awk '{print $2}'|xargs kill -QUIT

# 平滑重启
ps aux|grep -E "php-fpm:\ master\ process"|awk '{print $2}'|xargs kill -USR2
```

#### 安装 PHP 时常见错误列表

```
# 常见的报错可参考以下进行修复

# 解决 error: the HTTP XSLT module requires the libxml2/libxslt 错误
yum -y install libxml2 libxml2-dev libxslt-devel

# 解决 error: the HTTP image filter module requires the GD library. 错误
yum -y install gd-devel

# 解决 error: the GeoIP module requires the GeoIP library. 错误
yum -y install GeoIP GeoIP-devel GeoIP-data

# 解决 error: the Google perftools module requires the Google perftools 错误
yum -y install gperftools

# 解决 error: libatomic_ops library was not found. 错误
yum -y install libuuid-devel libblkid-devel libudev-devel fuse-devel libedit-devel libatomic_ops-devel

# 解决 error trying to exec 'cc1plus': execvp: No such file or directory 错误
yum -y install gcc-c++

# 解决configure: error: mbed TLS libraries not found. 错误。需要安装mbedtls，教程：https://www.24kplus.com/linux/281.html

# 解决 error: Cannot find OpenSSL's <evp.h> 错误
yum -y install openssl openssl-devel
ln -s /usr/lib64/libssl.so /usr/lib/

# 解决 error: Libtool library used but 'LIBTOOL' is undefined 错误
yum -y install libtool

# 解决 exec: g++: not found 错误
yum -y update gcc
yum -y install gcc+ gcc-c++

# 解决 configure: error: tss lib not found: libtspi.so 错误
yum -y install trousers-devel

# 解决 Can't exec "autopoint": No such file or directory 错误
yum -y install gettext gettext-devel gettext-common-devel

# 解决 configure: error: libcrypto not found. 错误
yum -y remove openssl-devel
yum -y install openssl-devel

# 解决 configure: error: Package requirements (libffi >= 3.0.0) were not met: No package 'libffi' found 错误
yum -y install libffi-devel

# 解决 fatal error: uuid.h: No such file or directory 错误
yum -y install e2fsprogs-devel uuid-devel libuuid-devel

# 解决 configure: error: openssl lib not found: libcrypto.so 错误
yum -y install openssl-devel

# 解决 tar (child): lbzip2: Cannot exec: No such file or directory 错误
yum -y install bzip2

# 解决 configure: error: C++ preprocessor "/lib/cpp" fails sanity check 错误
yum -y install gcc-c++

# 解决 configure: error: Please reinstall the BZip2 distribution 错误
yum -y install bzip2 bzip2-devel

# 解决 configure: error: cURL version 7.15.5 or later is required to compile php with cURL support 错误
yum -y install curl-devel

# 解决 configure: error: not found. Please provide a path to MagickWand-config or Wand-config program 错误
yum -y install ImageMagick-devel

# 解决 configure: error: no acceptable C compiler found in $PATH 错误
yum -y install gcc

# 解决 configure: error: Package requirements (icu-uc >= 50.1 icu-io icu-i18n) were not met: 错误
yum -y install libicu-devel

# 解决 configure: error: Package requirements (sqlite3 > 3.7.4) were not met: No package 'sqlite3' found 错误
yum -y install sqlite-devel

# 解决 configure: error: Package requirements (oniguruma) were not met: No package 'oniguruma' found 错误
yum -y install oniguruma oniguruma-devel

# 以上常见错误解决，其他未列出的需自己尝试处理，无非就是一些包的安装

```

安装 Composer 工具
----
```
mkdir /usr/local/share/composer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/share/composer --filename=composer
```

安装 Mariadb 服务
----
打开官网资源 [Repo](https://downloads.mariadb.org/mariadb/repositories/ "mariadb")

```
# 选择对应的系统，将生成的数据保存至 /etc/yum.repos.d/MariaDB.repo

# MariaDB 10.3 CentOS repository list - created 2019-03-26 08:55 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

# 安装 MariaDB
yum -y install MariaDB-server MariaDB-client

# 自启动并运行 SQL 服务
systemctl enable mariadb
systemctl start mariadb

# 找到 mariaDB 配置文件中的 [mysqld] 部分，添加一行
# 通常在 /etc/my.cnf.d/server.cnf 中
[mysqld]
skip-grant-tables

# 进行 sql 初始化设置
mysql_secure_installation

# 重启 SQL 服务
systemctl restart mariadb
```

以下是常见 MySQL 命令
```
# 创建数据库
CREATE DATABASE database_name;

# 创建新用户:
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';

# 赋予操作权限:
GRANT ALL PRIVILEGES ON database_name.table_name TO 'username'@'localhost';
GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';

# 删除用户:
DROP USER 'username'@'localhost';

# 显示某个用户权限:
SHOW GRANTS FOR 'root'@'localhost';

# 刷新权限:
FLUSH PRIVILEGES;
```

安装 Vsftpd
---

vsftpd 是 *very secure FTP daemon* 的缩写

```
# 安装 ftp 服务
yum -y install vsftpd

# 编辑配置文件
vim /etc/vsftpd/vsftpd.conf
```

匿名用户访问
```
anonymous_enable=YES
no_anon_password=YES
write_enable=YES
anon_root=/var/ftp # 配置匿名用户根目录，可选
anon_upload_enable=YES # 允许匿名用户上传文件
anon_mkdir_write_enable=YES # 匿名用户可以建目录
anon_other_write_enable=YES # 匿名用户修改删除
chown_username=username # 匿名上传文件所属用户名

# 修改匿名用户根目录权限
chmod 777 /var/ftp

# 重启 ftp 服务
systemctl start vsftpd
```

本地用户访问
```
# 创建 ftp 用户名为 username 且不可登录，也可使用 -d [指定文件夹] 作为上传根目录
useradd username -s /sbin/nologin [-d /home/username]

# 设置 ftp 用户密码
passwd username

# 配置参数
```
vim /etc/vsftpd/vsftpd.conf

ftpd_banner=Hello, world! # ftp 欢迎信息
local_root=/var/ftp # ftp 根路径文件夹
anon_root=/var/ftp # ftp 匿名用户根路径文件夹
use_localtime=YES # 设置 ftp 服务使用本地时间

anonymous_enable=NO # 禁止匿名访问
chroot_list_enable=YES # 启用用户列表
chroot_list_file=/etc/vsftpd/chroot_list # 用户列表清单

pasv_min_port=61001 # 传输文件最小端口
pasv_max_port=62000 # 传输文件最大端口

systemctl start vsftpd
```
# 配置 selinux
vim /etc/selinux/config

# 设置 SELINUX 为 disabled
SELINUX=disabled

# 重读配置文件
setenforce 0

systemctl start vsftpd
```
