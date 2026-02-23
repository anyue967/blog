---
title: review
tags:
  - Devops
  - linux
categories: 'Devops, linux'
abbrlink: 794381c6
date: 2019-07-26 10:49:34
---

2019-07-26-Review
<!-- more -->

## Linux
### **linux启动过程**？
+ [文档参考链接](https://www.runoob.com/linux/linux-system-boot.html)
+ **POST--> BIOS(Boot Sequence)--> MBR(bootloader,446)--> Kernel--> initrd(initramfs)--> [ROOTFS]--> /sbin/init(/etc/inittab)**  
  
* initrd：Linux初始RAM磁盘（initrd）是在系统引导过程中挂载的一个临时根文件系统；
  
+ init进程是系统所有进程的起点
    |  类型   |  名称   |                         位置                          |
    | :------ | :------ | :---------------------------------------------------- |
    | SysV    | init    | CentOS5之前: /etc/inittab                             |
    | Upstart | init    | CentOS6: /etc/inittab, /etc/init/*.conf               |
    | Systemd | systemd | CentOS7: /usr/lib/systemd/system, /etc/systemd/system |

+ Linux允许为不同的场合, 分配不同的开机启动程序, 这就叫做"运行级别"（runlevel）, 也就是说，启动时根据"运行级别", 确定要运行哪些程序
+ 系统初始化, 在init的配置文件中有这么一行: `si::sysinit:/etc/rc.d/rc.sysinit`, 完成一些系统初始化的工作(**rc启动脚本实际上都是放在/etc/rc.d/init.d/目录下**)　

### **Apache**有几种**工作模式**, 各有什么优缺点?
+ [文档参考链接1](https://www.cnblogs.com/zhichaoma/articles/7784688.html)
+ [文档参考链接2](https://www.cnblogs.com/qiujun/p/6861773.html)
  
+ 三种工作模式:`MPM: (Multi-Processing Module, 多进程处理模块), 它们分别是 prefork, worker, event`
  * **prefork**: 是一个**多进程, 非线程型**的, 预派生的MPM, 使用多个进程, 每个进程在某个确定的时间只单独处理一个连接, *效率高*, 但*内存使用比较大*
  * **worker**: 使用了**多进程和多线程**的混合模式, *占用内存少*, 但*由于多个子进程共享父进程内存地址, 易发生阻塞*
  * **event**: 类似于worker模式, **多进程, 多线程, epoll**, 解决了keep-live场景下, 长期占用线程的资源浪费问题(某些线程因为被keep-alive，挂在那里等待，中间几乎没有请求过来，一直等到超时)

### Apache设置虚拟目录的字段?
+ `vim /etc/httpd/conf/httpd.conf`
  
  ```bash
  基于IP地址:
  <VirtualHost IP>
  DocumentRoot /home/wwwroot/10
  ServerName www.anyue.com
  <Directory /home/wwwroot/10 >
  AllowOverride None
  Require all granted
</VirtualHOst>
  
  基于主机域名:
  <VirtualHost IP>
  DocumentRoot /home/wwwroot/www
  ServerName www.anyue.com
  <Directory /home/wwwroot/www >
  AllowOverride None
  Require all granted
</VirtualHOst>
  
  基于端口号:
  <VirtualHost IP:8080>
  DocumentRoot /home/wwwroot/8080
  ServerName www.anyue.com
  <Directory /home/wwwroot/8080 >
  AllowOverride None
  Require all granted
  </VirtualHOst>
  ```

### 列出Linux打包工具并写出相应解压参数, 至少3种
```bash
tar -czvf     # -z gzip(.tar.gz)  -j bzip(.bz2) 
zip test.txt.zip test.zip  #压缩文件
unzip test.zip -d /tmp   # -l 查看
```

### 利用sed命令将test.txt中所有的回车换成空格?
```bash
sed -i s/\r/ /g test.txt
seq 10 |tr "\n" " "
seq 10 |awk -v ORS=" " '{print $0}'
```
### 在每周六的凌晨3:15执行/home/shell/collect.pl, 并将标准输出和标准错误输出到/dev/null设备, 写出crontab中的语句?
```bash
15 3 * * 6 /home/shell/collect.pl > /dev/null 2>&1
```

### 计划每星期天早上8点服务器定时重启，如何实现？
```bash
00 8 * * 7 /sbin/init 6
```

### 当用户在浏览器输入网址，计算机对DNS解释经过哪些流程？（注：本机跟本地DNS还没有缓存）
+ a.用户输入网址到浏览器；
+ b.浏览器发出DNS请求信息；
+ c.计算机`首先查询本机HOST文件`，看是否存在，存在直接返回结果，不存在，继续下一步；
+ d.计算机按照`本地DNS`的顺序，向合法dns服务器查询IP结果；
+ e.合法dns返回dns结果给本地dns，本地dns并缓存本结果，直到TTL过期，才再次查询此结果；
+ f.返回IP结果给浏览器；
+ g.浏览器根据IP信息，获取页面；

### iptables是否支持time时间控制用户行为，请写出步骤
支持，需要增加相关支持的内核补丁，并且要重新编译内核。或者使用crontab配合iptables，首先：`vi /deny.bat` 输入`/sbin/iptables -A OUTPUT -p tcp -s 192.168.1.0/24 --dport 80 -j DROP`保存退出，打开crontab-e `00 21＊　＊　＊ /bin/sh /deny.bat`  

### 源码编译apache, 要求安装目录为/usr/local/apache, 需有压缩模块, rewrtie, worker模式, 并说明apache的worker MPM中,为什么ServerLimit要放到配置段最前面?
```bash
tar -xzvf apr-1.6.3.tar.gz 
cd apache-1.6.3
./configure --prefix=/usr/local/apache --enable-so --with-rewrite --with-mpm-worker  # 不放在最前面, client会忽略掉的
make
make install
```

### 匹配文本中的key, 并打印出该行及下面5行？
```bash
grep -A5 key filename
```

### dmseg命令中看到ip_conntrack:table full, dropping packet.如何解决？
- [参考链接](http://www.ttlsa.com/linux/ip_conntrack-table-full-dropping-packet-solution/)
- 更改ip_conntrack大小
- 不加载ip_conntrack模块

### 简述21, 22, 23, 25, 110, 143, 873, 3306端口运行的服务?
- 23--> telnet 服务使用
- 110--> POP3  服务
- 143--> IMAP v2
- 873--> rsync 文件传输服务
- 3306--> MySQL 数据库服务

### 如何在Linux上重建初始化内存盘镜像文件？
- CentOS 5.X / RHEL 5.X: `mkinitrd -f -v /boot/initrd-$(uname -r).img $(uname -r)`
- CentOS 6.X / RHEL 6.X: `dracut -f`

### 如何查看bond0的状态?
```bash
cat /proc/net/bonding/bond0
```

### linux中的/proc文件系统有什么用？
- /proc 文件系统是一个虚拟文件系统, 它只存在内存当中, 而不占用外存空间, Linux 内核提供了一种通过 /proc 文件系统, 在运行时访问内核内部数据结构, 改变内核设置的机制.
  ```bash
  查看proc信息: ls /proc/
  查看内核信息: ls /proc/sys
  查看网卡信息: ls /proc/net
  查看SCSI信息: ls /proc/scsi
  查看已加载模块: cat /proc/modules
  ```

### lvs的工作模式及工作过程(LB 集群是 load balance 集群的简写)？
- DR(Direct Routing)模式: LB收到请求包后, 将请求包中`目标MAC地址`转换为`某个选定RS的MAC地址`后将包转发出去, `RS收到请求包后, 可直接将应答内容传给用户.` 此时要求LB和所有RS都必须在一个物理段内, 且LB与RS群共享一个虚拟IP.
- NAT(Network Address Translation)模式: LB收到用户请求包后, LB将`请求包中虚拟服务器的IP地址`转换为某个选定`RS的IP地址`, 转发给RS; RS将应答包发给 LB, LB将应答包中RS的IP转为虚拟服务器的IP地址, 回送给用户.
- IP隧道(IP Tunneling)模式: LB收到用户请求包后, 根据IP隧道协议封装该包, 然后传给某个选定的RS; RS解出请求信息, 直接将应答内容传给用户。此时要求RS和LB都要支持IP隧道协议.

### Mysql数据库的备份和还原？
```bash
mysqldump -hhostname -uusername -ppassword databasename > backupfile.sql
mysql -hhostname -uusername -ppassword databasename < backupfile.sql
```

### Linux系统是由哪些部分组成？
+ Linux系统一般有4个主要部分：`内核、shell、文件系统和应用程序`
  * Linux内核: 内核是操作系统的核心，具有很多最基本功能，如虚拟内存、多任务、共享库、需求加载、可执行程序和TCP/IP网络功能。Linux内核的模块分为以下几个部分：存储管理、CPU和进程管理、文件系统、设备管理和驱动、网络通信、系统的初始化和系统调用等。
  * Linux shell: shell是系统的用户界面，提供了用户与内核进行交互操作的一种接口。它接收用户输入的命令并把它送入内核去执行，是一个命令解释器。另外，shell编程语言具有普通编程语言的很多特点，用这种编程语言编写的shell程序与其他应用程序具有同样的效果。
  * Linux文件系统: 文件系统是文件存放在磁盘等存储设备上的组织方法。Linux系统能支持多种目前流行的文件系统，如EXT2、EXT3、FAT、FAT32、VFAT和ISO9660。
  * Linux应用程序: 标准的Linux系统一般都有一套都有称为应用程序的程序集，它包括文本编辑器、编程语言、XWindow、办公套件、Internet工具和数据库等。

### 用一条命令查看目前系统已经启动服务监听的端口？
- `netstat -antl |grep "LISTEN"`

### 统计出一台web srver上各个状态(ESTABLISHED/SYN_SENT/SYN-RECV等)的个数
```bash
cat access_log | awk ‘{print $1}’ | uniq -c|sort -rn|head -10
netstat -antl |grep "ESTABLISHED" |wc -l
netstat -antl |grep "SYN_SENT" |wc -l
netstat -antl |grep "SYN_RECV" |wc -l
netstat -n |grep ^tcp |awk '{print $NF}' |sort -r |uniq -c
```

### 什么是默认登录shell?
+ 一般Linux默认的用户shell都是bash, 可以登录进去敲命令, 修改 `usermod -s /bin/zsh username`; `useradd -s /bin/zsh username`

### shell中可以使用那些类型的变量?
+ set: 查看所有变量(全局、局部和函数), 删除: unset name
+ 局部(标准、普通)变量: 生效范围为当前shell进程, 对当前shell之外的其他shell进程, 包括当前shell的子进程均无效
+ 环境(全局)变量: 生效范围为当前shell进程及其子进程(export=declare -x|env命令可以查看系统中环境变量)
+ 本地变量: 生效范围为当前shell进程中某代码片断，通常指函数
+ 位置变量: `$1 $2 $3...`
+ 特殊变量: `$? $0 $* $@ $# $$`

### 监控？特点
+ 使用nagios 对服务器进行监控，其特点可实时实现手机短信、电子邮件、MSN、飞信报警。
+ 使用cacti 对流量进行监控。zabbix可以执行脚本监视，图形界面监控。

### shell如何定义函数?
```bash
#!/bin/bash
function sum(){
    val1=$1
    val2=$2
    val3=$(($1+$2))
    echo $val3
}
ret_val=$(sum 10 20)
echo $ret_val
```

### OSI七层模型，tcp三次握手过程，tcp连接断开过程，什么情况下tcp进入time_wait?
+ 物理层 数据链路层 网络层 传输层 会话层 表示层 应用层

+ TCP三次握手的过程:
  * `第一次握手：`客户端发送syn包(seq=x)到服务器，并进入SYN_SEND状态，等待服务器确认;
  * `第二次握手：`服务器收到syn包，必须确认客户的SYN(ack=x+1)，同时自己也发送一个SYN包(seq=y)，即SYN+ACK包，此时服务器进入SYN_RECV状态;
  * `第三次握手：`客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=y+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。

+ TCP四次挥手:
  * `第一次挥手：`主动关闭方发送一个FIN，用来关闭主动方到被动关闭方的数据传送，也就是主动关闭方告诉被动关闭方：我已经不会再给你发数据了(当然，在fin包之前发送出去的数据，如果没有收到对应的ack确认报文，主动关闭方依然会重发这些数据)，但此时主动关闭方还可以接受数据。
  * `第二次挥手：`被动关闭方收到FIN包后，发送一个ACK给对方，确认序号为收到序号+1(与SYN相同，一个FIN占用一个序号)。
  * `第三次挥手：`被动关闭方发送一个FIN，用来关闭被动关闭方到主动关闭方的数据传送，也就是告诉主动关闭方，我的数据也发送完了，不会再给你发数据了。
  * `第四次挥手：`主动关闭方收到FIN后，发送一个ACK给被动关闭方，确认序号为收到序号+1，至此，完成四次挥手。
  * time_wait状态是四次挥手中服务器向客户端发送FIN终止连接后进入的状态

### 什么是跨站脚本攻击，有何危害，sql注入攻击如何防范
+ 跨站脚本攻击(也称为XSS)：指利用网站漏洞从用户那里恶意盗取信息。用户在浏览网站、使用即时通讯软件、甚至在阅读电子邮件时，通常会点击其中的链接。攻击者通过在链接中插入恶意代码，就能够盗取用户信息。
+ sql注入：即通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令

### 重置MySQL root密码？
```bash
vim /etc/my.cnf, 在[mysqld]下添加skip-grant-tables
UPDATE mysql.user SET password=PASSWORD('新密码') where USER='root';
flush privileges
```

### apache 和 MySQL两种服务能否装在同一台机器上，如何查看apache和MySQL端口和进程
```bash
netstat -an |grep :80   ps -ef |grep httpd   ps -aux |grep httpd
netstat -an |grep :3306
```

### 统计/var/log/nginx/access.log日志访问量最多的前十个IP
+ nginx
  ```bash
  awk '{print $1}' urlogfile |sort |uniq -c |sort -nr -k1 |head -n 1
  awk '{print $1}' /usr/local/nginx/logs/localhost.access.log |sort |uniq -c |sort -nr -k1 |head -n 10
  ```
+ apache
  ```bash
  cd /var/log/httpd/; cat access_log |awk '{print $1}' |uniq -c |sort -rn -k1 |head -n 10
  ```

### 怎么查看当前系统中每个IP的连接数，怎么查看当前磁盘IO(sysstat)，怎么查看当前网络的IO(iftop iotop)
```bash
netstat -n |awk '/^tcp/ {print $5}' |awk -F: '{print $1}' |sort |uniq -c |sort -rn
iostat
iftop
```

### 如何将本地80 端口的请求转发到8080 端口，本地主机IP 为192.168.16.1  
```bash
iptables -t nat -A PREROUTING -d 192.168.16.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.16.1:8080  
iptables -t nat -A PREROUTING -d 192.168.16.1 -i eth0 -p tcp -dport 80 -j REDIRECT --to-port 8080
```

### 什么是NAT?常见的分为哪几种？DNAT和SNAT有什么不同，应用示例有哪些？
`SNAT，DNAT，MASQUERADE`都是NAT，MASQUERADE是SNAT的一个特例。`SNAT`是指在数据包从网卡发送出去的时候，把数据包中的源地址部分替换为指定的IP，这样，接收方就认为数据包的来源是被替换的那个IP的主机；`MASQUERADE`是用发送数据的网卡上的IP来替换源IP，因此，对于那些`IP不固定的场合`，比如拨号网络或者通过dhcp分配IP的情况下，就得用MASQUERADE；`DNAT`，就是指数据包从网卡发送出去的时候，修改数据包中的目的IP，表现为如果你想访问A，可是因为网关做了DNAT，把所有访问A的数据包的目的IP全部修改为B，那么，你实际上访问的是B；

因为，路由是按照目的地址来选择的，因此，`DNAT是在PREROUTING链`上来进行的，而SNAT是在数据包发送出去的时候才进行，因此是在POSTROUTING链上进行的  

 ### 磁盘显示有空间但是却不能touch文件？原因及解决方法？
[参考](https://www.cnblogs.com/kevingrace/p/5577201.html)  

+ 两种情况，一种是`磁盘配额`问题，另外一种就是EXT3文件系统的设计不适合很多小文件跟大文件的一种文件格式，出现很多小文件时，容易导致`inode耗尽`了

+ `df -h`命令查看磁盘占用空间，若显示充足的空间，用`df -i`查看索引节点(inode)是否用尽，`inode译成中文就是索引节点，每个存储设备（例如硬盘）或存储设备的分区被格式化为文件系统后，应该有两部份，一部份是inode，另一部份是Block，Block是用来存储数据用的。而inode呢，就是用来存储这些数据的信息，这些信息包括文件大小、属主、归属的用户组、读写权限等。inode为每个文件进行信息索引，所以就有了inode的数值。操作系统根据指令，能通过inode值最快的找到相对应的文件。`  

+ 解决方案:
    1）查找`find /var/spool/clientmqueue/ -type f -exec rm {} \;`删除/data/cache目录中的部分文件
    2）用软连接将空闲分区目录连接到inode占满的目录
    
### top 命令？
+ 系统运行时间和平均负载（uptime），当前时间，系统已运行的时间，当前登录用户的数量，相应最近5、10和15分钟内的平均负载；`load average`数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了；
+ CPU 状态：us, user： 运行(未调整优先级的) 用户进程的CPU时间，sy，system: 运行内核进程的CPU时间

### RZY
+ 命令运行时使用CTRL+c，强制终止当前进程
+ 命令运行时使用CTRL+z，强制当前进程转为后台，并使之挂起（暂停）
+ `find perl5/ -name "*.pm" -print |xargs wc -l`
+ `find perl5/ -name "*.pm" -print -exec wc -l {} \;`
+ wc -l \`find ./perl5 -name "*.pm"\`
+ `wc -l $(find ./perl5 -name "*.pm")`

### 如何实现nginx代理的节点访问日志记录是真实访客IP，不是代理的IP配置nginx.conf配置文件增加不同的标记内容 
### 如何在linux上重建初始化内存盘镜像文件？
### 如何让shell就脚本得到来自终端的输入？
### web服务器的调优要点？web服务器负载架构？
### linux 优化的12个步骤
### linux 运行级别0-6含义
+ 0：halt
+ 1：single user mode，直接以管理员身份切入
+ 2：multi user mode，no NFS
+ 3：multi user mode，text
+ 4：保留未定义
+ 5：multi user mode，graphic mode
+ 6：reboot

### A文件，编写脚本判断A文件大于5的数字
```bash
#!/bin/bash
for i in `sed s/[^0-9]//g ./A`
do
    if [ $i -gt 5]; then 
        echo $i
    fi
done

#!/bin/bash
for i in `grep -oE "[1-9][0-9]{0,9}" ./A`
do
    if [ $i -gt 5 ]; then
        echo "$i > 5"
    else
        echo "$i < 5"
    fi
done
```
### linux 服务中程序经常自动停止如何处理
### nginx 如何配置多域名？防盗链
### Apache、nginx、Lighttpd有哪些优缺点
### 什么是7层负载均衡和4层负载均衡，访问请求经过这两种负载均衡都做了那些处理
### 如何将标准输出与错误输出同时重定向到同一位置？
### TCP和UDP分别有什么优缺点？
+ [参考](https://blog.csdn.net/xiaobangkuaipao/article/details/76793702)  

+ `TCP面向连接`（如打电话要先拨号建立连接）；`UDP是无连接`的，即发送数据之前不需要建立连接；
+ `TCP提供可靠的服务`，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达；UDP尽最大努力交付，即不保证可靠交付；
+ UDP具有较好的实时性，工作效率比TCP高，适用于对高速传输和实时性有较高的通信或广播通信；
+ 每一条TCP连接只能是点到点的；UDP支持一对一，一对多，多对一和多对多的交互通信；
+ TCP对系统资源要求较多，UDP对系统资源要求较少；

### HTTP请求过程与原理？
+ 首先作为发送端的客户端在`应用层（HTTP协议）`发出一个想看某个web也页面的HTTP请求；
+ 接着，为了传输方便，在`传输层（TCP协议）`把从应用层处收到的数据（HTTP请求报文）进行分割，并在各个报文上打上标记序号及端口号后转发给网络层；
+ 在`网络层（IP协议）`，增加作为通信目的MAC地址后转发给链路层。这样一来，发往网络的通信请求就准备齐全了。
+ 接收端的服务器在链路层接收到数据，按序往上层发送，一直到应用层。当传输层到应用层才能算真正接收到由客户端发送来的HTTP请求；

### 影响mysql主从同步因素可能有哪些？
### MySQL的主从复制过程是同步的还是异步的？MySQL主从同步原理是什么？MySQL是通过什么来保证主从数据的同步?
### 存储引擎InnoDB和MyISAM有什么区别？
### Nginx代理负载均衡的调度算法有哪些?具体实现时的现象是什么？
### 为MySQL添加一个新用户
+ `create user 'xy'@'%' identified by '123456';`  
+ `insert into mysql.user(Host,User,Password) values('%','xy','password('123456')')` 

### MySQL数据库主从同步延迟解决方案
### 什么是冷热备份？各自优缺点？
### 实现一键还原前提条件？还原后有什么后果？
### 除DNS外，还可以通过什么方式进行域名解析，缺点？
### DNS的空间结构是怎样的？

### 排障
#### 步骤:
+ 故障特征；复现故障
+ 排除不可能原因
+ 定位
  * 从简单问题入手
  * 一次尝试一种方式
+ 备份源文件；借助工具
#### 例如
+ MBR损坏
  * 借助别的主机修复
  * 使用紧急模式
    - boot.iso
    - 使用完整的系统光盘，`boot: linux rescure /mnt/sysimage`
+ grub文件丢失
  * grub> find (hd0, 0)/    # 找到内核
  * grub> root (hd0, 0)
  * grub> kernel /vmlinuz
  * grub> initrd /initrd-
  * grub> boot
+ 另外故障
  * 单用户模式编辑inittab文件
