---
title: apache-nginx-tomcat
copyright: true
tags: devops
categories:
  - apache
  - nginx
  - tomcat
abbrlink: c9dd20a7
date: 2019-09-09 23:25:10
---
2019-09-09-ANT
<!-- more -->

## 进程与线程
> [参考文章](https://www.zhihu.com/question/25532384)  
> 进程是`资源分配`的最小单位，线程是`CPU调度`的最小单位； 

### 二者关系
> 一个线程只能属于一个进程，而一个进程可以有多个线程，但至少有一个线程（通常说的主线程）;  
> 资源分配给进程，`同一进程的所有线程共享该进程的所有资源`;  
> 线程在执行过程中，需要协作同步。不同进程的线程间要利用`消息通信`的办法实现同步;  
> 处理任务分给线程，即真正在处理任务上运行的是线程;  
> 线程是指进程内的一个执行单元，也是进程内的可调度实体; 

{% asset_img 进程与线程.png [进程与线程] %}  

{% asset_img 进程与线程01.png [进程与线程01] %}  

## 同步/异步 && 阻塞/非阻塞
### 同步/异步（被调用者）
+ `描述网络通信模式，适用于请求-响应模型`
+ 同步：发送方发送请求后，需要等待接收响应，否则将一直等待
+ 异步：发送方发送请求后，不需要等待响应，可以继续发送下一个请求，或者主动挂起线程并释放CPU

### 阻塞/非阻塞（调用者-应用程序）
+ `描述进程的函数调用方式`  
+ 阻塞：IO 调用会一直阻塞，直至结果返回才能继续执行
+ 非阻塞：IO 调用会立即返回，不需要等待结果，并可以执行下一个 I/O 调用

{% asset_img I/O模型.png [I/O模型] %}

## IO多路复用的三种模型:Select，Poll，Epoll
+ [参考文章](https://www.cnblogs.com/javalyy/p/8882066.html)  
+ IO有`内存IO`、`网络IO`和`磁盘IO`三种，通常说的IO指的是后两者; 以文件IO为例,一个IO读过程是文件数据从`磁盘→内核缓冲区→用户内存`的过程。同步与异步的区别主要在于数据从内`核缓冲区→用户内存`这个过程需不需要用户进程等待，即实际的IO读写是否阻塞请求进程。(网络IO把磁盘换做网卡即可)

| 项目       | select                              | poll           | epoll             |
|----------|-------------------------------------|----------------|-------------------|
| 操作方式   | 遍历                                | 遍历           | 回调              |
| 底层实现   | 数组                                | 链表           | 哈希表            |
| IO效率     | 线性遍历,时间复杂度O(n)             | 线性遍历, O(n) | 事件通知方式,O(1) |
| 最大连接数 | 1024(x86)/2048(x64)                 | 无上限         | 无上限            |
| fd拷贝     | 每次调用,fd集合从用户态拷贝到内核态 | 同select       | 拷贝进内核保存    |

## apache、nginx与tomcat区别
> Apache和nginx应该叫做`HTTP Server`，而tomcat是一个`Application Server`，JSP/Servlet容器应用的容器;
> Apache HTTP Server和Nginx都能够将某一个`静态资源`的内容通过HTTP协议返回到客户端，但是这个文本文件的内容是固定的——也就是说无论何时、任何人访问它得到的内容都是完全相同的; Tomcat能够`动态的生成资源`并返回到客户端; 

## apache
### [官方网站](http://httpd.apache.org/)
{% asset_img httpd_logo.png [https_logo] %}   

### 原理浅析
+ Apache基于`多进程的HTTP服务器`，它需要对每个用户请求创建一个子进程进行响应，并发的请求量大就会需要非常多的进程，从而占用极多的cpu资源和内存;
+ Apache是基于`模块化`设计的，`MPM`（Multi -Processing Modules，多重处理模块）是Apache的核心组件之一，Apache通过MPM来使用操作系统的资源，对进程和线程池进行管理;
+ MPM有三种工作方式：
   + [Prefork MPM](http://httpd.apache.org/docs/2.4/mod/prefork.html)
   + [Worker MPM](http://httpd.apache.org/docs/2.4/mod/worker.html)
   + [Event MPM](http://httpd.apache.org/docs/2.4/mod/event.html)

#### prefork 模式(预派生子进程)
+ `prefork`是一个`非线程型的、预派生的MPM`，使用多个进程彼此独立，且`每个进程只有一个线程`，每个进程在某个确定的时间只单独处理一个连接，`效率高`，但`内存使用比较大`。
+ 优点：适合于没有线程安全库，需要避免线程兼容性问题的系统。它是要求将每个请求相互独立的情况下最好的MPM，这样若一个请求出现问题就不会影响到其他请求。
+ 缺点：一个进程相对占用更多的系统资源，消耗更多的内存。而且，它并不擅长处理高并发请求，在这种场景下，它会将请求放进队列中，一直等到有可用进程，请求才会被处理。

#### worker 模式
+ `worker`基于多进程, 但是使用了`多进程和多线程`的混合模式，worker模式也同样会先预派生一些子进程，然后`每个子进程创建一些线程`，同时包括一个监听线程，每个请求过来会被分配到一个线程来服务。
+ 优点：线程比起进程会更轻量，因为`线程是通过共享父进程的内存空间`，因此，内存的占用会减少一些，在高并发，高流量的场景下会比prefork有更多可用的线程，表现会更优秀一些；
+ 缺点：如果一个线程出现了问题也会导致同一进程下的线程出现问题，如果是多个线程出现问题，也只是影响Apache的一部分，而不是全部。由于用到多进程多线程，需要考虑到线程的安全了，在使用keep-alive长连接的时候，某个线程会一直被占用，即使中间没有请求，需要等待到超时才会被释放（该问题在prefork模式下也存在）。

#### event 模式
+ Apache最新的工作模式，它和worker模式很像
+ 优点：不同的是在于它解决了`keep-alive长连接`的时候占用线程资源被浪费的问题（HTTP的Keepalive方式能减少TCP连接数量和网络负载），在event工作模式中，会有一些专门的线程用来管理这些keep-alive类型的线程，当有真实请求过来的时候，将请求传递给服务器的线程，执行完毕后，又允许它释放。这增强了在高并发场景下的请求处理；

## nginx
### [参考文章](https://www.cnblogs.com/strick/p/9336948.html)
### [官方网站](http://nginx.org/en/docs/)

### 原理浅析
#### 组成
> Nginx由一个`master进程和多个worker进程`组成，主进程接收客户端请求，再转交给工作进程处理，从而很好地利用多核心CPU的计算能力。

{% asset_img nginx.png [nginx_logo] %}   

#### 编译安装
```
# ./configure --prefix=/srv/nginx \
  --with-file-aio \
  --with-http_auth_request_module \  
  --with-http_ssl_module \
  --with-http_v2_module \
  --with-http_realip_module \ 
  --with-http_addition_module \ 
  --with-http_xslt_module=dynamic \ 
  --with-http_geoip_module=dynamic \
  --with-http_sub_module \
  --with-http_dav_module \
  --with-http_flv_module \
  --with-http_mp4_module \
  --with-http_gunzip_module \ 
  --with-http_gzip_static_module \ 
  --with-http_random_index_module \
  --with-http_secure_link_module \
  --with-http_degradation_module \
  --with-http_slice_module \
  --with-http_stub_status_module \ 
  --with-http_perl_module=dynamic \
  --with-pcre \
  --with-pcre-jit \ 
  --with-stream=dynamic \ 
  --with-stream_ssl_module \
```

#### 配置
```
sed -i '/^[[:space:]]*#/d' nginx.conf 
  user nginx;
  worker_process auto; 
  error_log /var/log/nginx/error.log;
  pid /run/nginx.pid;
  include /usr/share/nginx/modules/*.conf;

  event {
      worker_connections 1024;
  }
  http {
      include         mime.types;
      default_type    application/octet-stream;

      sendfile         on;
      keepalived       65;
      
      include /etc/nginx/conf.d/*.conf;
      include /etc/nginx/conf.d/vhosts/*.conf;    # 

      server {
          listen       80;
          server_name  localhost;

          location / {                # localhost:80/ 
              root     html;          # 路径
              index    index.html index.htm;
          }
          
          location /demo {        # localhost:80/demo/
              root /opt/test;     # 实际访问 /opt/test/demo
              alias /opt/test;    # 实际访问 /opt/test
          }

          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root html;
          }
      }
  }
```

### CGI、FastCGI、PHP-FPM
+ CGI（Common Interface，公共网关接口）是**Web服务器与外部程序之间**的接口标准，用于`两种不同程序之间的信息传递`。CGI规范允许Web服务器根据浏览器请求调用CGI程序，并将其输出结果通过响应发送给浏览器，从而使Web服务器支持处理复杂的网站业务需求;
+ `FastCGI`主要用于解决CGI性能上的缺陷。传统CGI方式是每当客户端请求CGI时，Web服务器就通过操作系统创建一个新的CGI进程，一个CGI进程完成一个请求处理后就退出，下次请求再创建一个新CGI进程。由于这种方式需要不断为每个请求创建进程，因此在网站并发量很大时显得非常低效。FastCGI优化了这种工作方式，它由一个`常驻的CGI进程管理器`，通过管理`一个进程池`来处理Web服务器的请求，由此提高了性能;
+ PHP提供的`PHP-FPM`（FastCGI Process Manager）就是一个`FastCGI进程管理器`，其可执行文件位于PHP安装的目录中;

## tomcat
### [参考文章](https://www.cnblogs.com/hggen/p/6264475.html)
### [官方网站](http://tomcat.apache.org/)
{% asset_img tomcat.png [tomcat_logo] %}  