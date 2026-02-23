---
title: wordpress部署
copyright: true
tags:
  - Devops
  - linux
  - 网站
categories: 'Devops, linux'
abbrlink: 137599dc
date: 2019-06-11 17:20:46
---
2019-06-11-WordPress
<!--more-->

## redhat系统
## 软件版本:
```bash
$ wget https://wordpress.org/latest.tar.gz 
$ wget https://cdn.mysql.com//Downloads/MySQL-5.7/ mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar # rpm
$ wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.27-el7-x86_64.tar.gz # 二进制包 
$ wget http://mirrors.tuna.tsinghua.edu.cn/apache//httpd/httpd-2.4.39.tar.gz`  
$ wget https://www.php.net/distributions/php-7.3.7.tar.bz2  
```

## 安装依赖:
```
$ yum install -y apr* \ 
  autoconf automake bison bzip2 bzip2* \
  compat* cpp curl curl-devel fontconfig fontconfig-devel \
  freetype freetype* freetype-devel gcc gcc-c++ gd \ 
  gettext gettext-devel glibc kernel kernel-headers \
  keyutils keyutils-libs-devel krb5-devel libcom_err-devel \ 
  libpng libpng-devel libjpeg* libsepol-devel libselinux-devel \ 
  libstdc++-devel libtool* libgomp libxml2 libxml2-devel libXpm* \ 
  libtiff libtiff* make mpfr ncurses* ntp openssl openssl-devel \
  patch pcre-devel perl php-common php-gd policycoreutils telnet \
  t1lib t1lib* nasm nasm* wget zlib-devel
  
$ tar -xzvf cmake-3.15.1.tar.gz   
$ ./configure
$ make && make install  
```

### 1. 部署软件:
+ **[参考文档1](https://www.cnblogs.com/ftl1012/p/9302206.html)**  
+ **[参考文档2](https://www.cnblogs.com/apollo1616/p/10205216.html)**  
+ **[参考文档3](https://www.cnblogs.com/wujuntian/p/8183952.html)**  
+ **[参考文档4](https://www.cnblogs.com/inana/p/7545103.html)**  

#### 1.1 RPM安装Mysql:
> 在安装之前, 首先要查看的是, 系统中有没有已经安装过的情况, 键入 `rpm -qa|grep mysql` , 如果无任何显示，则表示没有安装过相关组件, 如果有, 则根据显示出来的名字, 键入`rpm -e --nodeps XXXX`(星号为你要删除的文件名字), 接着键入`rpm -qa|grep mariadb`, rpm 删除时如果有依赖关系, 可以用`yum remove XXXX`; 同样的步骤, 把出现的文件删除; 两步都完成后, 可以开始安装所需要的包了  

```
$ tar -xvf mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar   
$ yum remove mariadb-libs-5.5.35-3.el7.x86_64
```

***依次安装, 各包之间有依赖关系***    
```   bash
$ rpm -ivh mysql-community-common-5.7.27-1.el7.x86_64.rpm 
$ rpm -ivh mysql-community-libs-5.7.27-1.el7.x86_64.rpm  
$ rpm -ivh mysql-community-client-5.7.27-1.el7.x86_64.rpm 
$ rpm -ivh mysql-community-server-5.7.27-1.el7.x86_64.rpm 
$ find / -name mysql -print  
$ rpm -ql XXXX.rpm	#查看安装路路径
```

+ 数据库目录: 
  ```
  /var/lib/mysql/
  ```
+ 配置文件: 
  ```bash
  /usr/share/mysql #(mysql.server命令及配置文件)
  ```
+ 相关命令: 
  ```bash
  /usr/bin # (mysqladmin mysqldump等命令)
  ```
+ 启动脚本: 
  ```bash
  /etc/rc.d/init.d/   # (启动脚本文件mysql的目录)  
  $ vim /etc/my.cnf  
  # For advice on how to change settings please see
  # http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
  [mysqld]
  ...
  skip-grant-tables	# 添加
  ```

#### 1.2 二进制安装Mysql:
```bash
$ tar -xzvf mysql-5.7.27-el7-x86_64.tar.gz
$ groupadd mysql
$ useradd -r -g mysql -s /sbin/nologin mysql      # -r 创建系统账户  
$ mv ./mysql-5.7.27-el7-x86_64 /usr/local/mysql
$ mkdir /usr/local/mysql/data
$ chown -Rf mysql:mysql /usr/local/mysql/

$ cd /usr/local/mysql/bin
$ ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/myslq/data
[Note] A temporary password is generated for root@localhost: C#ePz?uG+9a.	  # 初始密码

$ mkdir /var/lib/mysql 
$ ln -s /usr/local/mysql/lib/ /usr/lib/mysql
$ ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock
$ ln -s /usr/local/mysql/include/mysql /usr/include/mysql

# 配置文件
$ vim /etc/cnf	# 配置文件
  [mysqld]
  #skip-grant-tables
  secure_file_priv="/usr/local/mysql"
  port = 3306
  basedir = /usr/local/mysql
  datadir = /usr/local/mysql/data
  socket=/tmp/mysql.sock
  pid-file=/usr/local/mysql/data/mysql.pid
  log-error=/usr/local/mysql/data/error.log
  character_set_server=utf8
  user=mysql
  max_connections=1500
  symbolic-links=0
  !includedir /etc/my.cnf.d            
$ mkdir /etc/my.cnf.d      
$ cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
  46 basedir=/usr/local/mysql  
  47 datadir=/usr/local/mysql/data  
$ service mysqld start    
$ chkconfig mysqld on
```

#### 1.3 源码编译安装Mysql:  
**[源码安装参考](https://www.cnblogs.com/galengao/p/5755788.html)**

+ 安装boost(若只是安装mysql, 不用安装, 编译引用解包路径即可)  
```bash
$ wget https://nchc.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz    
$ tar xzf boost_1_59_0.tar.gz 
$ cd boost_1_59_0  
$ ./bootstrap.sh  
$ ./b2 install
$ useradd -r -s /sbin/nologin mysql
$ mkdie /usr/local/mysql/data
$ chown -Rf mysql:mysql /usr/local/mysql  
$ mkdir build && cd build
$ cmake .. \
  -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
  -DMYSQL_DATADIR=/usr/local/mysql/data \
  -DMYSQL_USER=mysql \
  -DSYSCONFDIR=/etc \
  -DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
  -DWITH_DEBUG=0 \
  -DEXTRA_CHARSETS=all \
  -DDEFAULT_CHARSET=utf8 \ 
  -DDEFAULT_COLLATION=utf8_general_ci \
  -DWITH_BOOST=boost
$ make -j 4 && make install  
$ cp /etc/my.cnf /etc/my.cnf.bak 
$ cd /usr/local/mysql/bin
$ ./mysqld --initialize-insecure \
  --user=mysql \
  --basedir=/usr/local/mysql \
  --datadir=/usr/local/myslq/data 
  [Note] A temporary password is generated for root@localhost: Plip9_BU81ZA
$ ./mysql_ssl_rsa_setup \
  --user=mysql \
  --basedir=/usr/local/mysql \
  --datadir=/usr/local/mysql/data
```

**接下来配置如二进制安装方法**
**关于Mysql: ERROR 1819 (HY000): Your password does not satisfy the current policy requirements解决方法:**  

```SQL
mysql> show variables like 'validate_password%';	
	+--------------------------------------+--------+
	| Variable_name                        | Value  |
	+--------------------------------------+--------+
	| validate_password_check_user_name    | OFF    |
	| validate_password_dictionary_file    |        |
	| validate_password_length             | 8      |
	| validate_password_mixed_case_count   | 1      |
	| validate_password_number_count       | 1      |
	| validate_password_policy             | MEDIUM |
	| validate_password_special_char_count | 1      |
	+--------------------------------------+--------+
	7 rows in set (0.01 sec)

mysql> set global validate_password_policy=LOW;  # 设置校验等级
  Query OK, 0 rows affected (0.00 sec) 
mysql> set global validate_password_length=6;    # 设置密码长度
  Query OK, 0 rows affected (0.01 sec)
mysql> alter user 'root'@'localhost' IDENTIFIED BY '123456';
  Query OK, 0 rows affected (0.00 sec)
mysql> grant all privileges on *.* to root@'%' with grant option; 
  Query OK, 0 rows affected (0.00 sec)
```

#### 1.4 安装Apache:
+ 安装依赖：
+ pcre
  ```bash
  tar -xzvf pcre-8.43.tar.gz
  ./configure --prefix=/usr/local/pcre
  `make && make install
  ```

+ apr
  ```bash
  tar -xzvf apr-1.7.0.tar.gz
  ./configure --prefix=/usr/local/apr
  make && make install
  ```
+ apr-util
  ```shell
  $ tar -xzvf apr-util-1.6.1.tar.gz
  $ ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
  make && make install

  $ useradd www -s /sbin/nologin
  $ ./configure --prefix=/usr/local/apache24 \
    --with-mysql=/usr/local/mysql \
    --with-pcre=/usr/local/pcre \
    --with-apr=/usr/local/apr \
    --with-apr-util=/usr/local/apr-util \
    --with-openssl=/usr/local/openssl \
    --with-zlib=/usr/local/zlib \
    --with-mpm=prefork \
    --enable-modules=most \
    --enable-rewrite \
    --enable-shared=max \
    --enable-so \
    --enable-mpms-shared=all \
    --enable-ssl \
    --enable-cgi \
    --enable-mpms-shared=all 
  ```

+ 自启动脚本配置
  ```bash
  /usr/local/apache/bin	
  vim httpd			# 编辑自启动脚本
    #!/bin/bash
    # chkconfig:2345 80 90
    # description:Apache httpd
    function start_http()
    {
      /usr/local/apache24/bin/apachectl  start
    }
    function stop_http()
    {
      /usr/local/apache24/bin/apachectl  stop
    }
    case "$1" in
    start)
      start_http
    ;;  
    stop)
      stop_http
    ;;  
    restart)
      stop_http
      start_http
    ;;
    *)
    echo "Usage : start | stop | restart"
    ;;
    esac
      
  $ chmod a+x httpd
  $ cp -frp ./httpd /etc/init.d	# -f强制合并, 不询问	-r递归合并	-p保持文件属性不变
  $ systemctl daemon-reload
  $ systemctl start httpd
  $ chkconfig --add httpd	  # 设置开机自启
  $ vim ./apache24/conf/httpd.conf
    164 User deamon -->User www
    165 Group deamon -->Group www
    196 #ServerName www.example.com:80 -->ServerName localhost:80
  $ firewall-cmd --permanent --add-service=http
  $ firewall-cmd --permanent --add-service=http
  $ firewall-cmd --reload 
  ```

+ 测试: 192.168.10.130 -->It Works!
+ 问题1:   
  - **服务不支持chkconfig的解决?**  
  - **解决方法:**
    ```bash
    httpd 脚本的开头要这样加：  
	  #!/bin/bash
	  #chkconfig:345 61 61 	# 此行的345参数表示, 在哪些运行级别启动
	  #description:Apache httpd # 此行必写,描述服务.
    ```
+ level<等级代号> 　指定读系统服务要在哪一个执行等级中开启或关毕。
  ```bash
  #等级0表示：表示关机
  #等级1表示：单用户模式
  #等级2表示：无网络连接的多用户命令行模式
  #等级3表示：有网络连接的多用户命令行模式
  #等级4表示：不可用
  #等级5表示：带图形界面的多用户模式
  #等级6表示：重新启动
  ```
+ [nginx参考](https://anyue967.github.io/2019/06/11/LNMP架构部署动态网站环境/)  

#### 1.5 安装PHP7(http://pecl.php.net):
+ [依赖包安装见](https://anyue967.github.io/2019/06/11/LNMP架构部署动态网站环境/)   
+ [编译参考](https://blog.csdn.net/zhou75771217/article/details/83303058)  
+ [PHP整合apache参考](https://blog.csdn.net/m0_37886429/article/details/79643078)
  ```bash
  $ wget wget https://nih.at/libzip/libzip-1.2.0.tar.gz 
  $ tar -xzvf libzip-1.2.0.tar.gz
  $ cd libzip-1.2.0/
  $ mkdir build && cd build
  $ cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local/libzip 
  $ make && make install 
  ```

+ 问题1:  
  - **off_t undefined?**  
  - **解决方法:**  
    ```bash
    $ export LD_LIBRARY_PATH=/usr/local/libgd/lib	
    $ echo '/usr/local/lib64 	# error: off_t undefined; 
      /usr/local/lib 
      /usr/lib 
      /usr/lib64'>>/etc/ld.so.conf
    $ ldconfig -v
    ```

+ 问题2:  
  - **如何让apache支持解析PHP?**  
  - **解决方法:**  
+ 编译PHP必须加 **--with-apxs2=/usr/local/apache24/bin/apxs**, 目的是在 /usr/local/apache24/modules 下生成对应的libphp7.so
  ```bash
  $ ./configure \
    --prefix=/usr/local/php73 \
    --with-config-file-path=/usr/local/php73/etc \
    --with-apxs2=/usr/local/apache24/bin/apxs \
    --enable-mysqlnd \
    --with-mysqli=mysqlnd \
    --with-mysql-sock=/tmp/mysql.sock \
    --with-pdo-mysql=mysqlnd \
    --enable-inline-optimization \
    --enable-opcache --enable-fpm \
    --with-fpm-user=www --with-fpm-group=www \
    --with-gettext --with-iconv --with-mhash \
    --with-openssl=/usr/local/openssl \
    --enable-bcmath --enable-soap \
    --with-libxml-dir --enable-libxml --enable-pcntl \
    --enable-shmop --enable-sysvsem --enable-sockets \
    --with-curl --with-zlib-dir=/usr/local/zlib \
    --enable-zip --with-gd \
    --with-png-dir=/usr/local/libpng \
    --with-jpeg-dir=/usr/local/jpeg \
    --with-freetype-dir=/usr/local/freetype \
    --with-xpm-dir=/usr \
    --with-libzip=/usr/local/libzip \
    --without-pear --with-tidy=/usr/local/tidy \
    --enable-xml --enable-mbregex \
    --enable-mbstring --enable-ftp \
    --with-xmlrpc --enable-session \
    --enable-ctype 
  ```
**php.ini-production / php.ini-devlopmentphp.ini -->php.ini**
**php-fpm.conf.default -->php-fpm.conf**
**sapi/fpm/init.d.php-fpm -->/etc/rc.d/init.d/php-fpm**

```bash
php.ini-development
php.ini-production
$ mv /etc/php.ini /etc/php.ini.bak
$ cp php.ini-production /usr/local/php73/etc/php.ini
$ ln -s /usr/local/php73/etc/php.ini /etc/php.ini
$ cp /usr/local/php73/etc/php-fpm.conf.default /usr/local/php73/etc/php-fpm.conf
$ ln -s /usr/local/php73/etc/php-fpm.conf /etc/php-fpm.conf
$ vim /usr/local/php73/etc/php-fpm.conf
  17 pid = run/php-fpm.pid

$ cp sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
$ chmod 755 /etc/rc.d/init.d/php-fpm
$ chkconfig php-fpm on
```

**注意: PHP7里的用户设置在如下文件中配置, 而不是在php-fpm.conf中**
```bash
/usr/local/php73/etc/php-fpm.d
$ cp ./www.conf.default www.conf
$ vim ./www/conf
  user=www
  group=www
```

**注意 httpd.conf 配置:**
```bash
162 LoadModule php7_module        modules/libphp7.so
173 User www
174 Group www
206 ServerName localhost:80

263 <IfModule dir_module>
264     DirectoryIndex index.html index.php
265 </IfModule>

400     AddType application/x-compress .Z
401     AddType application/x-gzip .gz .tgz
402     AddType application/x-httpd-php .php	# 增加的
```

问题1: **checking for libzip... not found configure: error: Please reinstall the libzip distribution**
**解决方法:[参考](https://www.hahack.com/codes/cmake/)** 
```bash
ln -s /usr/local/libzip/lib/libzip/include/zipconf.h /usr/local/include/zipconf.h
```

问题2: **error: ‘INT_MAX’ undeclared (first use in this function)**  
***解决方法:***  
```bash
#define INT_MAX 4096	# 声明宏定义  
#include "/usr/include/limits.h"
```

问题3: **configure: WARNING: unrecognized options: --with-mysql, --with-vpx-dir, --with-t1lib, --enable-gd-native-ttf, --with-mcrypt**  
**解决方法:**
+ 对于PHP7, --with-mysql 不适用: 更改为`--with-pdo-mysql`;    
+ 对于64位系统, 出现 --with-t1lib 执行: 
  - `ln -s /usr/lib64/libltdl.so /usr/lib/libltdl.so`
  - `cp -frp /usr/lib64/libXpm.so* /usr/lib`
+ 对于 --with-mcrypt: 从PHP7.2开始, Mcrypt no longer provided, [参考](https://webtatic.com/packages/php72/);     
+ 对于 --enable-gd-native-ttf: 同--with-mcrypt;    
+ 对于 --with-vpx-dir: [参考](https://www.php.net/manual/zh/image.installation.php), 要激活 GD 支持, 配置 PHP 时加上 --with-gd[=DIR], DIR 是 GD 的基本安装目录, 要使用推荐的绑定的 GD 库版;  

### 2. 测试:
```bash
$ cd /usr/local/apache24/htdocs/wordpress
$ cp ../wp-config-sample.php wp-config.php
$ vim wp-config.php
  22 /** WordPress数据库的名称 */
  23 define( 'DB_NAME', 'wordpress' );
  24 
  25 /** MySQL数据库用户名 */
  26 define( 'DB_USER', 'wordpress' );
  27 
  28 /** MySQL数据库密码 */
  29 define( 'DB_PASSWORD', 'password' );
```
**IP:80/wordpress**  
**IP:80/wordpress/wp-login.php?loggedout=true**  
**[A Globally Recognized Avatar](https://cn.gravatar.com)**  