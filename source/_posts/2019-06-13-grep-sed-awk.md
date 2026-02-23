---
title: 关于grep、sed和awk的基础用法
tags:
  - Linux
  - shell
categories: Linux
abbrlink: a796df41
date: 2019-06-13 21:23:48
---
2019-06-13-grep/sed/awk
<!-- more -->

#### Linux报“ '/usr/bin' is not included in the PATH environment variable ”的解决方法:
```bash
利用/usr/bin vi 编辑之前改过环境变量:
/usr/bin vi ~/.bashrc
```

#### find(文件目录的查找)
> +n(n天以前), -n(n天以内)   

| 选项                    | 说明                   |
|-------------------------|------------------------|
| -name                   | 根据文件名匹配         |
| -iname                  | 忽略大小写             |
| -regex pattern          | 基于正则进行文件名匹配 |
| -user                   | 根据属主匹配           |
| -group                  | 根据属组匹配           |
| -uid                    | 根据UID匹配            |
| -gid                    | 根据GID匹配            |
| -nouser                 | 匹配没有属主文件       |
| -nogroup                | 匹配没有属组文件       |
| -type f d c b l p s     | 匹配文件类型           |
| -size k M G             | 根据文件大小           |
| -atime -mtime -ctime    | 访问 修改 改变         |
| -perm -mode /mode       | 根据权限匹配           |
| -print -ls, ls -l       | --                     |
| -exec/-ok command {} \; | --                     |
| -a -o -not              | 逻辑上的与 或 非       |

```bash
find /data/ -name "*.log" -type f -size +10k -exec cp {} /tmp \;   # 区分大小写
find /data/ -iname "*.log" -type f -size +10k -m perm 644 -exec rm -rf {} \;  
find /data/ -name "*.log" -type f -mtime +30 -size +10M -exec mv {} /tmp/ \; # +30 30天以前，-30 30天之内   
find /data -not \( -user user1 -a -not -type d \)
```

#### grep(文本搜索工具,行)
+ grep [OPTION] PATTERN [FILE]
+ Search for PATTERN in each FILE.
+ Example: 
  ```bash
  grep -i 'hello world' menu.h main.c
  ```

| 选项           | 说明                   |
|----------------|----------------------|
| `--color=auto` | 对匹配到的文本着色显示 |
| -v             | 反选                   |
| -i             | 忽略大小写             |
| -n             | 显示行号               |
| -c             | 统计匹配的行数         |
| -f             | 根据模式文件处理       |
| -E             | 使用ERE                |
| -w             | 匹配整个单词           |
| -e             | 实现多个选项           |
| -o             | 仅显示匹配到的字符串   |
| -A #           | after, 后#行(tail -n)  |
| -B #           | before, 前#行(head -n) |
| -C #           | 前后各#行              |

##### 示例:
```bash
# -f, --file=FILE    # 从文件每一行获取匹配模式
grep -f a.txt b.txt  # 输出b文件在a文件相同的行

# -v, --invert-match # 打印不匹配的行
grep -v -f a.txt b.txt # 输出b文件中在a文件不同的行

# -e, --regexp=PATTERN # 使用模式匹配，可指定多个模式匹配
echo "a bc de" |xargs -n1 |grep -e 'a' -e 'bc'
grep -E -v "^$|^#" /etc/httpd/conf/httpd.conf

# -o, --only-matching  # 只打印匹配的内容
echo "this is a test" |grep -o 'is'   # is
ifconfig eno16777736 |grep -o "\<\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}\>"
grep -Eo "(([1-9]?[0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([1-9]?[0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])"

# -m, --max-count=NUM # 输出匹配的结果num数
seq 1 20 |grep -m 5 -E '[0-9]{2}'   # 10 11 12 13 14 15

# -w, --word-regexp # 模式匹配整个单词
# xargs 能够捕获一个命令的输出，然后传递给另外一个命令
echo "a ab abc abcd abcde" |xargs -n1 |grep -E -w -o '[a-z]{2,3}' # ab abc

# -c, --count 
seq 1 20 |grep -c -E '[0-9]{2}'   # 11

# -n, --line-number
echo "a ab abc abcd abcde" |xargs -n1 |grep -n 'de$'  # 5:abcde

# -B, --before-context=NUM
seq 1 10 |grep 5 -B 3  # 2 3 4 5  
```

#### sed(处理行,一次读取一行内容)
> 流编辑器, 过滤和替换文本.   
> `sed [选项] '模式1动作1; 模式2动作2; ...' 待处理File`  

> 工作原理: sed命令将当前处理的行存储在临时缓冲区--**模式空间(临时缓冲区)**进行处理, 处理完把结果输出, 并清空模式空间. 然后再将下一行读入模式空间进行处理输出, 以此类推, 直到最后一行；
> 还有一个空间叫**保持空间, 又称暂存空间**, 可以暂时存放一些处理的数据,但不能直接输出, 只能放到模式空间输出；这两个空间其实就是在内存中初始化的一个内存区域, 存放正在处理的数据和临时存放的数据；   
> 默认工作模式输出模式空间内容  

| 选项          | 说明                                     |
|---------------|----------------------------------------|
| -n            | 只输出命令匹配的内容(不显示模式空间内容) |
| -r, --reg     | 开启支持扩展正则                         |
| -i            | sed的修改结果直接应用于文件              |
| -e            | 多个命令组合 或者用`;`分割               |
| -f 脚本文件名 | 从sed脚本读入sed操作                     |

| 动作(定位)            | 说明                            |
|-----------------------|-------------------------------|
| x,y                   | x~y之间的行                     |
| x,+N                  | 从x行开始向后的N的行            |
| x,y!                  | 查询不包括x和y行之间的行        |
| $                     | 最后一行                        |
| /pattern1/,/pattern2/ | 查询从匹配模式1~模式2之间的行   |
| /pattern/pattern/     | 查询包含两个模式的行            |
| x,/pattern/           | 从与x行~与pattern匹配行之间的行 |

| 动作(编辑命令)    | 说明                                     |
|-------------------|----------------------------------------|
| p                 | 打印,输出指定行                          |
| d                 | 删除指定行                               |
| a \string         | 在指定行后追加                           |
| i \string         | 在指定行前插入                           |
| c \string         | 行替换,用c后边的字符串替换原数据         |
| r FILE            | 指定行后添加文件内容                     |
| w FILE            | 指定范围的内容另存至指定文件中           |
| s/pattern/string/ | 查找并替换字串                           |
| n                 | 读取下一个输入行, 用下一个命令处理新行   |
| N                 | 将当前读入行的下一行读取到当前的模式空间 |
| g                 | 将保持缓冲区的内容复制到模式缓冲区       |
| G                 | 将保持缓冲区的内容追加到模式缓冲区       |

##### 示例:
```bash
# -n, --quiet, --silent # 只输出匹配的字串到屏幕
sed -n '/^blp5/p' /etc/services     # 打印匹配blp5开头的行
tail /etc/services |sed -n '1,3p'   # 打印1-3行
sed -n '1p;$p' test.txt`            # 打印第一行与最后一行

# a i c r w命令
tail /etc/services |sed '/blp5/i \test'   
tail /etc/services |sed '/blp5/a \test'   
tail /etc/services |sed '/blp5/c \test'   # 在blp5替换新行
tail /etc/services |sed '2a \test'        # 在指定行下一行添加
sed '1,2r /etc/issue' /etc/fstab 
sed '/oot/w /tmp/oot/txt' /etc/fstab  

# d 命令
tail /etc/services |sed '/blp5/,$d'   # 删除匹配行直到最后一行 
sed '/^#/d;/^$/d' /etc/httpd/conf/httpd.conf
sed '1,3d' test.txt                   # 删除1-3行

# s 命令
# 默认只替换每行中第一次被模式匹配的字串，g-全局替换，i-忽略大小写， 
tail /etc/services |sed 's/blp5/test/'  # 替换blp5字符串为test s/old/new/g
tail /etc/services |sed '1,4s#blp5#test#'
sed -i '/SELINUX/s/enforceing/disabled/g' /etc/selinux/config  
sed 's/l..e/&r/g'
sed 's#\(l..e\)#\1r#g'  
sed 's#l\(..e\)#L\1#g'    # like love

# N,D,P--处理多行模式空间; H,h,G,g,x--将模式空间内容放入存储空间以便编辑
  # N: 不会清空模式空间内容，并从输入文件中读取下一行数据，追加到模式空间中，两行数据以换行符\n连接；

# 例子:  
seq 6 |sed '/^$/d; 1,3G'      # 前3行每行后加入空行 ^$(空行)
seq 6 |sed 'n; d'             # 偶数行删除 1 3 5
seq 6 |sed '/5/{x; p; x}'     # 匹配内容的前一行插入空行
seq 6 |sed 'N; s/\n/ /'       # 1 2   3 4    5 6
sed 'N;s#\n#=#g'       
seq 6 |sed '$!N; $!D'         # 5 6
seq 6 |sed -n '/3/,/6/'p      # 3 4 5 6
```

#### awk (先读取行, 再提取列)
> awk是一个处理文本的编程语言工具, 能用简短的程序处理标准输入或文件、数据排序、计算以及生成报表等等  

> awk 引用shell变量使用 `-v` 或者 `双引号+单引号`即可  

  ```bash
  awk -v STR=hello '{print STR, $NF}' test.txt`  
  STR='Hello'; echo |awk '{print "'${STR}'"}'`  
  ```

{% asset_img awk.jpg [awk原理说明] %}

#### Usage:
+ `awk 选项 '条件1{动作1} 条件2{动作2} ...' file`    
+ `awk -F: '{print $1 >>"/tmp/awk.log"}' test.txt` # -F '[ :\t;]'
+ 条件/模式: 条件也即模式: `BEGIN, 空模式, END模式`
+ 动作: 
  * { print 动作1,动作2, ... }       # 打印后则默认空格隔开; 若没有逗号隔开, 则会连在一起
  * { print 动作1; print 动作2 }     # 多个动作用分号隔开
    ```bash
    awk -F ":" '{ if($3<500) { print $1,"系统用户" } else{ print $1,"普通用户" }}' /etc/passwd
    ```

| 变量名               | 描述                                                 |
|----------------------|------------------------------------------------------|
| FS, Field Separator  | 输入字段(列)分隔符，默认是空格或制表符  BEGIN{FS=":"} |
| OFS                  | 输出字段(列)分隔符，默认是空格                        |
| RS, Record Separator | 输入记录(行)分隔符，默认是换行符\n                    |
| ORS                  | 输出记录(行分隔符，默认是换行符\n                     |
| --                   | --                                                   |
| NF, Number Field     | 统计当前记录中字段(列)个数，列号                      |
| NR, Number Record    | 行号，当前处理的文本行的行号                          |
| FNR                  | 同NR，各文件分别计数的行号                            |
| $0                   | 当前读入整行数据                                     |
| $n                   | 当前读入行的第n个字段，第n列                          |
| FILENAME             | 显示文件名                                           |

##### 示例:
+ FS OFS(输入输出字段分割符)
  ```bash
  awk 'BEGIN{FS=":"}{print $1,$2}' /etc/passwd |head -n5	        # root x
  awk 'BEGIN{FS=":";OFS=":"}{print $1,$2}' /etc/passwd |head -n5	# root:x
  awk 'BEGIN{FS=":"}{print $1"#"$2}' /etc/passwd |head -n5        # root#x
  ```
+ RS ORS(输入输出记录分隔符)
  ```bash
  echo "www.baidu.com/user/test.html" |awk 'BEGIN{RS="/"}{print $0}'   # user test.html
  ```
+ NF(列号,统计当前字段个数) 
  ```bash
  echo "a b c d e f"|awk '{print NF}'    # 6
  echo "a b c d e f"|awk '{print $NF}'   # f
  ```
+ NR FNR(行号,统计记录编号)
  ```bash
  tail -n5 /etc/services |awk '{print NR,$0}'  
  tail -n5 /etc/services |awk 'END{print NR}'	   # 5
  cat /etc/passwd |grep "/bin/bash" |awk 'BEGIN{FS=":"} {printf $1 "\t" $3 "\t 行号:"NR "\t 字段数:"NF "\n"}'
  ```
+ 格式化输出
  ```bash
  awk -F: '{print "%-12s %-6s %-8s\n", $1, $2, $NF}' /etc/passwd
  netstat -an |awk '$6 ~ /LISTEN/&&NR>=1&&NR<=10 {print NR, $4, $5,$6}' OFS="\t"
  ```

>须知: 在awk中, 有3种情况表达式为假: 数字是0, 空字符串和未定义的值. 数值运算, 未定义变量初始值为0; 字符运算; 未定义变量初始值为空.

+ awk是按行处理的，每次读取一行，并遍历打印每个字段
  ```bash
  awk '{i=1;while(i<=NF){print $1;i++}}' file
  awk '{for(i=NF;i>=1;i--){print $i" "};print""}' file
  awk 'BEGIN{for(i=1;i<=5;i++){if(i==3){break};print i}}'	file    # 1 2
  awk 'BEGIN{for(i=1;i<=5;i++){if(i==3){continue};print i}}' file # 1 2 4 5
  ```

+ 统计passwd文件用户数
  ```bash
  awk -F':' 'BEGIN{count=0;} {name[count]=$1; count++;} END{for(i=0; i<NR; i++) print i, name[i]}' /etc/passwd
  ```

#### Nginx 日志分析:
> 114.242.26.65 - - [22/Mar/2017:08:41:08 +0800] "GET /css/20151103Style.css HTTP/1.1" 200 9041 "http://sz.cdn-my.mobiletrain.org/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) ApplwWebKit/537.36 (HTML, like Gecko) Chrome/50.02661.102 Safari/537.36"  

+ 1.统计2017年9月5日 PV量
```bash
grep '05/Sep/2017' cd.mobiletrain.org.log |wc -l  
awk '$4>="[05/sep/2017:08:00:00" && "[05/sep/2017:09:00:00" {print $0}' sz.mobiletrain.log |wc -l
```

+ 2.统计2017年9月5日 一天访问最多的10个IP(top 10)  
```bash
grep '05/Sep/2017' sz.mobiletrain.org.log |awk '{ips[$1]++} END{for(i in ips){print i,ips[i]}}' |sort -k2 -nr |head -n10    
awk '05\/Sep\/2017' sz.mobiletrain.org.log |awk '{ips[$1]++} END{for(i in ips){print i,ips[i]}}' |sort -k2 -nr |head -n10
```

+ 3.统计2017年9月5日 访问>100次的IP
```bash
grep '05/Sep/2017' sz.mobiletrain.org.log |awk '{ips[$1]++} END{for(i in ips){if(ips[i]>100){print i,ips[i]}}}'
```

+ 4.统计2017年9月5日 访问最多的10个页面($request top 10)  
```bash
awk '/05\/Sep\/2017/ {urls[$7]++} END{for(i in urls){print i,urls[i]}}' sz.mobiletrain.org.log |sort -k1 -rn |head -n10
```

+ 5.统计2017年9月5日 每个url访问内容总大小($body_bytes_sent)  
```bash
awk '/05\/Sep\/2017/ {urls[$7]++; size[$7]+=$10} END{for(i in urls){print urls[i],size[i],i}}' sz.mobiletrain.org.log |sort -k1 -rn |head -n10
```

+ 6.统计2017年9月5日 每个IP访问状态码数量($status)  
```bash
awk '/05\/Sep/\2017/ {ip_code[$1" "$9]++} END{for(i in ip_code){print i,ip_code[i]}}' sz.mobiletrain.org |sort -k1 -rn |head -n10
```

+ 7.统计2017年9月5日 访问状态码为404及出现次数($status)
```bash
awk '/05\/Sep\/2017/ {if($9=="404"){ip_code[$1" "$9]++} END{for(i in ip_code){print i,ip_code[i]}}}' sz.mobiletrain.org.log
```

+ 8.统计前一分钟的PV量(带入外部变量)
```bash
date=$(date -d '-1 minute' +%d/%b/%Y:%H:%M); awk -v date=$date '$0 ~ date {i++} END{print i}' sz.mobiletrain.org.log
```

+ 9.统计2017年9月5日 8:30-9:00, 访问状态码是404  
```bash
awk '$4>="[05/Sep/2017:08:30:00 && [05/Sep/2017:09:00:00" {if($9=="404"){ip_code[$1" "$9]++} for(i in ip_code){print i,ip_code[i]}}' sz.mobiletrain.org.log
```

+ 10.统计2017年9月5日各种状态码数量  
```bash
awk '/05\/09\/2017/ {code[$9]++;total++} END{for(i in code){printf i"\t"; printf code[i]"\t"; printf "%.2f",code[i]/total*100; print "%"}}' sz.mobiletrain.org.log
```

+ 11.统计独立IP数
```bash
awk '{print $1}' access.log |sort -r |uniq -c |wc -l
```

+ 12.统计总PV量
```bash
awk '{print $7}' access.log |wc -l
```

+ 13.UV统计
```bash
awk '{print $11}' access.log |sort -r |uniq -c |wc -l
```

+ 14.截至目前为止访问量前20的IP
```bash
awk '{print $1}' access.log |sort |uniq -c |sort -nr |head -n20
```

+ 15.早上9-12总请求量
```bash
sed -n "/2019:09:00"/,/2019:12:00/"p access.log
`awk '/2019:09:00/,/2019:12:00' access.log |wc -l
```

+ 16.状态码404、502、503、500、499等错误信息页面，打错误出现次数大于20的IP地址
```bash
awk '{if($9~/502|499|500|503|404/) print $1, $9}' access.log |sort |uniq -c |sort -nr |awk '{if($1>20) print $2}'
```

+ 17.访问最多的页面
```bash
awk '{print $7}' access.log |sort |uniq -c |sort -nr |head -n20
```

+ 18.请求处理时间大于5s的URL，打印出时间、URL、访客、IP
```bash
awk '{if($NF>5) print $NF, $7, $1}' access.log |sort -nr |more
```

+ continue: 在循环中不执行continue下面的代码，转而进入下一轮循环
+ break: 结束并退出整个循环
+ exit: 退出脚本，常带一个整数给系统，如 exit 0
+ return: 在函数中将数据返回或返回一个结果给调用函数的脚本
+ break: 是立马跳出循环, continue: 是跳出当前条件循环, 继续下一轮条件循环, exit: 是直接退出整个脚本

#### 琐碎知识点:  
| 项目 | 说明                                          |
|------|---------------------------------------------|
| $0   | 脚本自身名字                                  |
| $?   | 返回上条命令是否执行成功, 0执行成功, 非零失败 |
| S#   | 参数总数                                      |
| $*   | 参数都被看作一个字符串                        |
| $@   | 每个参数被看作独立字符串                      |
| $$   | 当前进程PID                                   |
| $!   | 上一条运行后台进程PID                         |