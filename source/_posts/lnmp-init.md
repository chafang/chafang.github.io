---
title: LNMP 安装和初始化配置
date: 2020-12-09 17:29:12
tags:
---

Linux 下的 Nginx、PHP、MariaDB 安装相对来说比较简单，以下是操作详细步骤。

<!-- more -->

## 安装 Nginx

新增用户组，新增用户不可登陆

``` bash
# 准备用户和组
groupadd nginx

# 禁止用户登录
useradd nginx -g nginx -s /sbin/nologin -M
```

官网下载 [Download](http://nginx.org/en/download.html "nginx")

``` bash
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

``` bash
htpasswd -c /usr/local/share/nginx/conf/ptList [username]

# 在 server 或 location 配置中添加，htpasswd 只支持目录认证
auth_basic "Please input password";
auth_basic_user_file /usr/local/share/nginx/conf/ptList;
```

## 安装 PHP

安装 PHP 需要用到的包

``` bash
yum -y install bzip2 bzip2-devel libzip libzip-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel libicu libicu-devel sqlite sqlite-devel oniguruma oniguruma-devel libevent libevent-devel libpng-devel libwebp-devel libjpeg-devel icu libicu-devel libxslt-devel
```

官网下载 [Download](https://www.php.net/downloads "php")

``` bash
tar -zxf php-latest.tar.gz
cd php-latest

# 可使用 ./configure --help 查看帮助和相关插件
./configure --prefix=/usr/local/share/php --with-fpm-user=nginx --with-fpm-group=nginx --enable-fpm --with-curl --with-gettext --enable-intl --with-mysqli --with-openssl --with-pdo-mysql --with-pdo-sqlite --with-pear --with-xsl --with-zlib --with-bz2 --with-mhash --enable-bcmath --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-sysvshm --enable-xml --enable-gd --with-webp --with-jpeg

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
ps aux|grep -E "php-fpm:\ master\ process"|awk '{print $2}'|xargs kill -TERM

# 平滑重启
ps aux|grep -E "php-fpm:\ master\ process"|awk '{print $2}'|xargs kill -USR1
```

## 安装 Composer

``` bash
mkdir /usr/local/share/composer

php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/share/composer --filename=composer
```

## 安装 Mariadb

打开官网资源 [Repo](https://downloads.mariadb.org/mariadb/repositories/ "mariadb")

按照自身实际情况选择适合的版本，同时也可将官方的镜像源替换为国内镜像，中科大的镜像源相对完整，可[点击这里](http://mirrors.ustc.edu.cn/help/mariadb.html)

``` bash
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

# 找到 mariaDB 配置文件中的 [mysqld] 部分，添加一行，是跳过验证
# 通常在 /etc/my.cnf.d/server.cnf 中
[mysqld]
skip-grant-tables

# 进行 sql 初始化设置
mysql_secure_installation

# 重启 SQL 服务
systemctl restart mariadb
```

## 常用 MySQL 命令

``` sql
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
