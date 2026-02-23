---
title: Linux Tools
copyright: true
tags:
  - Linux
categories: Linux
abbrlink: df044da8
date: 2019-11-29 15:19:44
---
2019-11-29-LinuxTools
<!-- more -->

## Linux 系统观测工具

## 系统负载及性能
### 1. uptime 
```bash
root@192.168.1.161:~$uptime
10:30:06 up 10 days, 19:26,  1 user,  load average: 3.05, 2.81, 2.15

1、当前时间 10:30:06
2、系统已运行的时间 10 days, 19:26
3、当前前在线用户 1 user
4、平均负载：0.54, 0.40, 0.20，最近1分钟、5分钟、15分钟系统的负载
5、load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了；
```

### 2. dmesg
### 3. top
* [top参考](https://www.cnblogs.com/zhoug2020/p/6336453.html)
* `系统运行时间和平均负载`
* `Tasks — 任务（进程）`，系统现在共有137个进程，其中处于运行中的有1个，136个在休眠（sleep），stoped状态的有0个，zombie状态（僵尸）的有0个
* `CPU 状态`
  + us, user: 运行(未调整优先级的) 用户进程的CPU时间 √
  + sy，system: 运行内核进程的CPU时间 √
  + ni，niced: 运行已调整优先级的用户进程的CPU时间
  + wa，IO wait: 用于等待IO完成的CPU时间
  + hi: 处理硬件中断的CPU时间
  + si: 处理软件中断的CPU时间
  + st：这个虚拟机被hypervisor偷去的CPU时间（译注：如果当前处于一个hypervisor下的vm，实际上hypervisor也是要消耗一部分CPU处理时间的）
* `内存使用`
  + 第一行是物理内存使用，第二行是虚拟内存使用(交换空间)
  + 第四行的free + 第四行的buffers + 第五行的cached √

```bash
top - 10:41:25 up 10 days, 19:37,  1 user,  load average: 0.34, 1.03, 1.56
Tasks: 137 total,   1 running, 136 sleeping,   0 stopped,   0 zombie
%Cpu(s): 16.6 us,  1.7 sy,  0.0 ni, 81.2 id,  0.3 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem :  7730168 total,   204536 free,  7324360 used,   201272 buff/cache
KiB Swap:  5242876 total,   399728 free,  4843148 used.    81848 avail Mem

  PID USER PR  NI  VIRT    RES    SHR  S  %CPU %MEM   TIME+   COMMAND
21257 xy   20  0   4757972 990552 3384 S  12.0 12.8   3173:53 /opt/xy/java+

16.6% us — 用户空间占用CPU的百分比；1.7% sy — 内核空间占用CPU的百分比。
0.0% ni  — 改变过优先级的进程占用CPU的百分比
81.2% id — 空闲CPU百分比
0.3% wa — IO等待占用CPU的百分比
0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
0.3% si — 软中断（Software Interrupts）占用CPU的百分比
```

| 命令 | 描述                         |
|------|----------------------------|
| d    | 指定刷新时间（s）              |
| M    | 按内存占用率显示             |
| P    | 按CPU占用率显示              |
| T    | 按累计时间显示               |
| c    | 切换显示命令名称和完整命令行 |

### 4. htop

## CPU使用率
### 1. vmstat(Virtual Meomory Statistic)
> 对系统的整体情况进行统计，对系统的整体情况进行统计  
> r经常大于4，id经常少于40，表示`cpu的负荷很重`; bi，bo长期不等于0，表示`内存不足`; disk经常不等于0，且在b中的队列大于3，表示`io性能不好`    
> `-f,--fork` 显示从系统启动至今的fork数量  
> `-s` 显示内存相关统计信息及多种系统活动数量  
> `-d,--disk` 查看磁盘的读写 

* Procs（进程）
  + r: 运行队列中进程数量
  + b: 等待IO的进程数量
* Memory（内存）
  + swpd: 使用虚拟内存大小
  + free: 可用内存大小
  + buff: 用作缓冲的内存大小
  + cache: 用作缓存的内存大小
* Swap
  + si: 每秒从交换区写到内存的大小
  + so: 每秒写入交换区的内存大小
* IO(现在的Linux版本块的大小为1024bytes）
  + bi: 每秒从块设备接收到的块数，单位：块/秒 也就是读块设备
  + bo: 每秒发送到块设备的块数，单位：块/秒  也就是写块设备
* 系统
  + in: 每秒中断数，包括时钟中断[interrupt]
  + cs: 每秒上下文切换数[count/second]
* CPU(以百分比表示)
  + us: 用户进程执行时间(user time)
  + sy: 系统进程执行时间(system time)
  + id: 空闲时间(包括IO等待时间),中央处理器的空闲时间,以百分比表示
  + wa: 等待IO时间

```bash
root@192.168.1.161:~$vmstat 5 5  # 5s内采集5个样本
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
4  0 4216792 187800      0 212148   88   40   746    84    9    4 10  2 86  2  0

root@192.168.1.161:~$vmstat -a 2 5  # -a,--active
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b   swpd   free  inact active   si   so    bi    bo   in   cs us sy id wa st
1  0 4215656 196520 1385284 5829956   88   40   746    84    9    5 10  2 86  2  0
```

### 2. mpstat  
> 主要用于多CPU环境下，它显示各个可用CPU的状态  
> mpstat -P CPU 时间间隔 采集次数  

```bash
root@192.168.1.161:~$mpstat -P ALL
Linux 3.10.0-693.el7.x86_64 (centos) 	2019年11月29日 	_x86_64_	(4 CPU)

12时10分00秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
12时10分00秒  all    9.62    0.00    1.44    2.34    0.00    0.35    0.01    0.00    0.00   86.24
```

### 3. sar(System Activity Reporter)
> [参考sar](https://blog.51cto.com/11555417/2138903)  
> sar [options][-o file] [t] [n]     
> t为采样间隔，n为采样次数，默认值是1；  

* %nice：显示在用户级别，用于nice操作，所占用 CPU 总时间的百分比
* %steal：管理程序(hypervisor)为另一个虚拟进程提供服务而等待虚拟 CPU 的百分比。
* %idle：显示 CPU 空闲时间占用 CPU 总时间的百分比。

```bash
root@192.168.1.161:~$sar -u -o test 10 3  # 周期 次数
Linux 3.10.0-693.el7.x86_64 (centos) 	2019年11月29日 	_x86_64_	(4 CPU)

12时16分46秒     CPU     %user     %nice   %system   %iowait    %steal     %idle
12时16分56秒     all     12.16      0.00      1.90      5.22      0.00     80.72

root@192.168.1.161:~$sar -u -f test	# 查看  
```

### 4. pidstat
> 主要用于监控全部或指定进程占用系统资源的情况，如CPU，内存、设备IO、任务切换、线程等  
> cpu使用情况统计(-u)  
> 内存使用情况统计(-r) 
> IO情况统计(-d)  

### 5. dstat
> 综合了vmstat，iostat，ifstat，netstat    
> dstat [option] [time] [count]    
> 常用选项：-l,--load; -m,--memory; -tcp  

## 内存使用率
* free -m

## 进程
* vmstat 1：查看进程是否阻塞（r）
* pidstat 1：查看每个进程的情况"

## 磁盘IO
* iostat -xzm 1 

## 网络IO
* sar -n DEV 1
* sar -n TCP,ETCP 1

### 6. 磁盘扩容
* `df -h`       # 查看分区空间使用情况 `-T`,显示**文件系统**类型
  + `parted -l`
  + `lsblk -f`
  + `file -s /dev/sda1`
* `du -sh *`    # 查看文件所占用大小
* `fdisk -l`    # 查看磁盘

#### 6.1 新增加的磁盘
```bash
[root@anyue ~]# pvs             # 查看新增加的盘
[root@anyue ~]# fdisk /dev/sdb  # 建立新分区更改类型为8e
[root@anyue ~]# mkfs.ext4 /dev/sdb1
[root@anyue ~]# pvcreate /dev/sdb1      # pvscan
[root@anyue ~]# vgextend VolGroup /dev/sdb1     # 当前需要扩容的LVM的组名，通过lvs查看
[root@anyue ~]# lvextend -L +20G /dev/mapper/VolGroup-lv_root 
[root@anyue ~]# resize2fs -p /dev/mapper/VolGroup-lv_root
```

#### 6.2 从其他分区获得空间,/home 299G
```bash
[root@anyue ~]# umount /home    # 提到无法卸载，fuser -m /home
[root@anyue ~]# e2fsck -f /dev/mapper/VolGroup-lv_home
[root@anyue ~]# resize2fs -p /dev/mapper/VolGroup-lv_home 279G
[root@anyue ~]# mount /home     # 此时可查看 df -h /home
[root@anyue ~]# lvreduce -L 279G /dev/mapper/VolGroup-lv_home   # -L,减小到20G；-l,减小20G；+
[root@anyue ~]# lvextend -L +20G /dev/mapper/VolGroup-lv_root
[root@anyue ~]# resize2fs -p /dev/mapper/VolGroup-lv_root  
```