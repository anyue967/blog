---
title: 搭建个人博客
tags:
  - Blog
categories: CRH
abbrlink: c442673f
date: 2019-06-08 09:53:34
---
2019-06-08-搭建博客
<!-- more -->

## 基础搭建
### 创建一个仓库，用于存放个人博客文件
+ 在Github首页右上角头像左侧加号点选择 New repositor(新存储库)或点击这里进行创建一个仓库.
{% asset_img 01-CreateBlog.jpg 创建仓库 %}  

+ 开启Github Pages
{% asset_img 02-CreateBlog.jpg 创建仓库 %}  

+ 进入仓库设置项，进行主题的选择
{% asset_img 03-CreateBlog.jpg 创建仓库 %}  

## Hexo 的搭建
> Hexo是一个简单、快速、强大的基于 Github Pages 的博客发布工具，支持`Markdown`格式，有众多优秀插件和主题。

### 安装 Node.js 
##### Linux/类Linux下利用nvm安装Node比较好，[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

```bash
$ curl https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
$ wget -qO- https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
```

##### Windows下nvm安装配置，[https://github.com/coreybutler/nvm-windows](https://github.com/coreybutler/nvm-windows)

  > nvm 安装好后，利用 nvm 相关命令安装node.js  
  > [参考配置 nvm](https://blog.csdn.net/qq_27626333/article/details/77857223)   
  > [参考cnblog博客](https://www.cnblogs.com/wyy1234/p/9727142.html)

+ 配置nvm安装目录下的 settings.txt
```bash
$ root: F:\Design\nvm
$ path: F:\Design\nodejs
$ arch: 64 
$ proxy: none 
$ node_mirror: https://npm.taobao.org/mirrors/node/
$ npm_mirror: https://npm.taobao.org/mirrors/npm/
```

+ 配置用npm下载包时全局安装的包路径
```
$ npm config set prefix “F:\Design\nvm\npm”
$ %NPM_HOME%=F:\Design\nvm\npm
```

+ 配置国内仓库  
```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

:link:[淘宝Node镜像](https://npm.taobao.org/mirrors/node/)  
:link:[淘宝Npm镜像](https://npm.taobao.org/mirrors/npm/)  

### nvm 命令  
| 命令                              | 说明                                                         |
| :-------------------------------- | :----------------------------------------------------------- |
| nvm arch 32/64                    | 显示Node是以32位还是64位模式运行                             |
| nvm install <version> [arch]      | 下载需要的Node版本,默认64位                                  |
| nvm list [available]              | 列出node.js安装,在末尾输入可用的类型以显示可供下载的版本列表 |
| nvm on                            | 启用版本管理                                                 |
| nvm off                           | 禁用版本管理                                                 |
| nvm proxy [url]                   | 设置代理,留空以查看当前代理,设置为“none”以删除代理           |
| nvm uninstall <version>           | 卸载指定版本                                                 |
| nvm use <version> [arch]          | 使用指定Node版本,可以指定位数                                |
| nvm root [path]                   | 设置nvm应存储不同版本node.js的目录                           |
| nvm version                       | 显示当前的 nvm 版本                                          |
| nvm node_mirror <node_mirror_url> | Set the node mirror(China)                                   |
| nvm npm_mirror <npm_mirror_url>   | --                                                           |

### 安装 Hexo 
[Hexo官方参考文档](https://hexo.io/zh-cn/docs/)
```bash
$ npm install -g hexo-cli
$ hexo init <folder>  // 必须为空目录
$ cd <folder>
$ npm install
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes

$ hexo new [layout] <title>  # hexo new "post title with whitespace"  
$ hexo g  # generate 网页文件
$ hexo s  # server https://localhost:4000 预览网页
$ hexo clean 
$ hexo d  # deployer push 到仓库
```

### 配置优化Hexo(命令都是在博客主目录执行)
#### 配置_config.yml
:link:[参考配置](https://github.com/cczeng/BlogBackup/blob/master/_config.yml)
#### 安装`hexo-deployer-git`扩展
```bash
$ npm install hexo-deployer-git --save
  
deploy:
type: git
repo: git@github.com:anyue967/anyue967.github.io.git
branch: master
```

#### 添加插件
```bash
$ npm install hexo-helper-live2d --save
```

:link:[参考官方 live-2d配置文件](https://github.com/EYHN/hexo-helper-live2d)

```bash
$ npm install hexo-generator-search --save

search:
path: search.xml
field: post
format: html
limit: 10000
```

```bash
$ npm install hexo-filter-optimize --save
```

## npm (Nodejs Package Manager)相关命令  
| 命令                    | 说明                         |
| :---------------------- | :--------------------------- |
| npm -v                  | 查询node版本                 |
| npm install xx -g       | 全局安装模块,如:`gulp@3.9.1` |
| npm install xx --save   | 本地安装模块并写入 json配置  |
| npm search xx           | 搜索模块                     |
| npm outdated            | 过时模块                     |
| npm update xx           | 更新模块                     |
| npm search xx           | 搜索模块                     |
| npm cache clear         | 清空本地缓存                 |
| npm uninstall xx        | 卸载模块                     |
| npm list -g \| grep xx | 查找全局安装模块             |
| npm list xx             | 查看模块版本号               |

```bash
$ npm config set prefix “F:\Design\nvm\npm”      # 配置用npm下载模块时全局安装的包路径
$ npm config set proxy=http://xxx               # 设置代理
$ npm config set registry="http://r.cnmpjs.org"     # 设置镜像仓库
$ npm install -g cnpm --registry=https://registry.npm.taobao.org`  # 使用淘宝镜像命令
```

```bash
$ npm cache add <tarball file>
$ npm cache add <folder>
$ npm cache add <tarball url> 
$ npm cache add <git url> 
$ npm cache add <name>@<version>  
$ npm cache clean 
$ npm cache verify  

npm start
npm stop
npm restart
```

## nrm 的安装和使用
```bash
npm install nrm -g    # 全局安装nrm  
nrm ls    # 列出可用的镜像源  

  * npm -------- https://registry.npmjs.org/
    yarn ------- https://registry.yarnpkg.com/
    cnpm ------- http://r.cnpmjs.org/
    taobao ----- https://registry.npm.taobao.org/
    nj --------- https://registry.nodejitsu.com/
    npmMirror -- https://skimdb.npmjs.com/registry/
    edunpm ----- http://registry.enpmjs.org/
nrm use taobao    # 选择国内淘宝镜像源
```