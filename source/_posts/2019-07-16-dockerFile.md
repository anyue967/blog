---
title: dockerFile
copyright: true
tags:
  - dockerfile
categories: docker
abbrlink: b94b563a
date: 2019-07-16 18:25:16
---
2019-07-16-DockFile
<!-- more -->

### DockerFile 语法
#### [docker_practice](https://github.com/yeasy/docker_practice)
#### 运行
```bash
docker run -it --rm -p 4000:80 dockerpracticesig/docker_practice
# localhost:4000
```

#### FROM -- 指定基础image
+ 构建指令，必须指定且需要在 DockerFile 其他指令的前面，后续指令依赖指定的 image，FROM指定的基础 image 既可以是本地 Repo 也可以是官方的远程仓
  ```yaml
  FROM centos:7.2
  FROM centos
  ```

#### MAINTAINER --指定镜像创建者信息
+ 构建指令，用于将 image 的制作者相关信息写入到 image 中，当执行`docker inspect`，输出中有相应的字段记录该信息
  ```
  MAINTAINER username "username@xxx.com"
  ```

#### RUN --安装软件用
+ 构建指令，RUN 可以运行任何被基础 image 支持的命令，若基础 image 为 centos，则软件管理部分只能使用 centos 的包管理命令
  ```
  RUN cd /tmp && curl -L 'http://archive.apache.org.dist/tomcat/tomcat-7/v7.0.8/bin/apache-tomcat-7.0.8.tar.gz' | tar -xz
  RUN [ "/bin/bash", "-c", "echo helllo" ]
  ```

#### CMD --设置 container 启动时执行的操作(覆盖)
+ 设置指令，用于 container 启动时指定的操作，该操可以是自定义的脚本，也可以是执行系统的命令，该指令只能在文件中存在一次，若有多个，只执行最后一条
  ```
  CMD echo "Hello，World！"
  ```

#### ENTRYPOINT --设置 container 启动执行的操作(追加)
+ 设置指令，指定容器启动时执行的操作，可以多次设置，但是只有最后一个有效
  ```
  ENTRYPOINT ls -l
  ```

+ 该指令独自使用, 若还有CMD命令且CMD是一条完整的可执行的命令，则CMD指令和ENTRYPOINT会相互覆盖，只有最后一个CMD 或 ENTRYPOINT 有效
  ```
  CMD echo "Hello World"
  ENTRYPOINT ls -l
  # CMD 不会执行, ENTRYPOINT有效
  ```

+ 该指令与CMD配合使用指定 ENTRYPOINT 的默认参数，则 CMD 指令不是完整的可执行命令，仅仅是参数部分；ENTRYPOINT 指令只能使用 JSON 方式 指定执行命令，而不能指定参数
  ```
  FROM ubuntu
  CMD ["-l"]
  ENTRYPOINT ["/usr/bin/ls"]
  ```

#### USER --设置 container 用户
+ 设置指令，设置启动容器的用户，默认 root 用户
  ```
  USER daemon = ENTRYPOINT ["memcached", "-u", "daemon"]
  ```

#### EXPOSE --指定容器需要映射到宿主机的端口
+ 设置指令，会将容器中端口映射成宿主机中的某个端口
  ```
  EXPOSE 22
  docker run -p port1 image
  EXPOSE port1 port2 port3
  docker run -p port1 port2 port3 image
  docker run -p host_port1:port1 -p host_port2:port2 -p host_port3:port3 image
  ```

#### ENV --用于设置环境变量
+ 构建指令，在 image 中设置一个环境变量，后续的 RUN 可以使用，container 启动后，通过 `docker inspect` 查看环境变量，或者 `docker run --env key=value` 时设置或修改环境变量
  ```
  ENV JAVA_HOME /path/to/java/dirent
  ```

#### ADD --从 src 复制文件到 container 的 dest 路径(解包后拷贝)
+ ADD [src] [dest]
  * [src] 是相对被构建的源目录的相对路径，可以是 文件或目录 路径，也可以是远程的文件 url
  * [dest] 是container 中的绝对路径

#### COPY --从 src 复制文件到 container 的 dest 路径
+ COPY [src] [dest]

### VOLUME --指定挂载点
+ 设置指令，使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用，因为容器使用的是UFS技术，这种文件系统不能持久化，当容器关闭后，所有更改都会丢失
  ```
  FROM base
  VOLUME ["/tmp/data"]
  ```

#### WORKDIR --切换目录
+ 设置指令，可以多次切换，等同于 cd，对RUN、CMD、ENTRYPOINT生效
  ```
  WORKDIR /p1 WORKDIR p2 RUN vim a.txt  === RUN cd /p1/p2 && vim a.txt
  ```

#### ONBUILD --在子镜像中执行
+ 在构建镜像时并不执行，而是在它的子镜像中执行
  ```yml
  ONBUILD ADD . /app/src
  ONBUILD RUN /usr/local/bin/python-build --dir /app/src

  docker build -t imagename .
  FROM hub.c.163.com/public/centos:6.7
  MAINTAINER xingyue@163.com
  ADD ./apache-tomcat-7.0.42.tar.gz /root
  ADD ./jdk-7u25-linux-x64.tar.gz /root
  ENV JAVA_HOME /root/jdk1.7.0_25
  ENV PATH $JAVA_HOME/bin:$PATH
  EXPOSE 8080
  ENTRYPOINT /root/apache-tomcat-7.0.42/bin/startup.sh && tailf /root/apache-tomcat-7.0.42/logs/catalina.out
  ```

| 命令       | 描述                    | 命令        | 描述                                 |
|------------|-----------------------|-------------|------------------------------------|
| FROM       | 构建基于的镜像          | COPY        | 拷贝文件或目录到镜像                 |
| MAINTAINER | 镜像维护者信息          | ENTRYPOINT  | 运行容器执行的shell命令              |
| RUN        | 构建镜像运行的shell命令 | VOLUME      | 指定容器挂载点到宿主机自动生成的目录 |
| CMD        | 运行容器执行的shell命令 | USER        | 为RUN/CMD/ENTRYPOINT执行命令指定用户 |
| EXPOSE     | 容器运行服务端口        | WORKDIR     | 为RUN/CMD/ENTRYPOINT设置工作目录     |
| ENV        | 容器内部环境变量        | HEALTHCHECK | 健康检查                             |
| ADD        | 拷贝文件或目录到镜像    | ARG         | 构建镜像时指定参数(效果同ENV)        |

#### 构建示例(构建PHP)
```yml
FROM centos:7
MAINTAINER anyue967
RUN yum install -y gcc gcc-c++ make gd-devel libxml2-devel libcurl-devel libjpeg-devel libpng-devel openssl-devel
ADD php-5.6.31.tar.gz /tmp/
RUN cd /tmp/php-5.6.31 && \
    ./configure --prefix=/usr/local/php \
    --with-config-file-path=/usr/local/php/etc \
    --with-mysql --with-mysqli \
    --with-openssl --with-zlib --with-curl --with-gd \
    --with-jpeg-dir --with-png-dir --with-iconv \
    --enable-fpm --enable-zip --enable-mbstring && \
    make -j 4 && \
    make install && \
    cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf && \
    sed -i "s/127.0.0.1/0.0.0.0/" /usr/local/php/etc/php-fpm.conf && \
    sed -i "21a \daemonize = no" /usr/local/php/etc/php-fpm.conf  # 21行追加 daemonize = no 
COPY php.ini /usr/local/php/etc
RUN rm -rf /tmp/php-5.6.31* && yum clean all
WORKDIR /usr/local/php
EXPOSE 9000
CMD ["./sbin/php-fpm", "-c", "/usr/local/php/etc/php-fpm.conf"]
```

### 构建示例(构建JAVA)
```yml
FROM cnetos:7
MAINTAINER anyue967
ADD jdk-8u45-linux-64.tar.gz /usr/local
ENV JAVA_HOME /usr/local/jdk1.8.0_45
ADD apache-tomcat-8.0.16.tar.gz /usr/local
COPY server.xml /usr/local/apache-tomcat-8.0.46/conf
RUN rm -f /usr/local/*.tar.gz
WORKDIR /usr/local/apache-tomcat-8.0.46
EXPOSE 8080
ENTRYPOINT ["./bin/catalina.sh", "run"]
```

#### 构建示例(tomcat)
```yml
FROM centos
MAINTAINER anyu967@163.com
COPY test.txt /usr/local/container.txt
ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.8.tar.gz /usr/local
RUN yum install -y vim
ENV MYPATH /usr/local
WORKDIR $MYPATH
ENV JAVA_HOME /usr/local/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8
EXPOSED 8080
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.8/bin/startup.sh"]
# CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/log/catalina.out
```


