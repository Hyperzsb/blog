---
layout: post
title: Docker - 安装配置
date: 2020-06-29
Author: Hyperzsb
tags: [docker]
comments: true
toc: true
---

阿里云的服务器今天莫名挂掉了，思前想后，打算重装服务器，一是为了往后的部署做准备，同时也可以借此机会温习一下相关环境的安装配置流程。第一步就从 Docker 开始吧。

<!-- more -->



---

## Docker 安装流程

**系统版本：Ubuntu 18.04**

**流程基本上同[ Docker 官方文档](https://docs.docker.com/engine/install)中给出的一致，对于一些安装细节进行了进一步的说明。**

### Docker 系统要求

> 仅以 Ubuntu 为例

支持的系统版本：

- Ubuntu Focal 20.04 (LTS)
- Ubuntu Eoan 19.10
- Ubuntu Eoan 19.10
- Ubuntu Xenial 16.04 (LTS)

支持的内核版本：

- x86_64 (amd64)
- armhf
- arm64

### 卸载旧版本 Docker

之前的Docker发行版名称有`docker`、`docker.io`和`docker-engine`，为保证使用最新版本的 Docker ，可以将旧版卸载：

```shell
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 安装新版本 Docker

> 官网给出的安装方法很多，包括使用仓库安装、离线 deb 包手动安装和自动化脚本安装等。
>
> 这里给出第一种方法，即使用仓库安装。
>
> 其他方式详见[ Docker 官方文档](https://docs.docker.com/engine/install)。

#### 初始化Dockers仓库

由于 Docker 无法直接在 Ubuntu 的下载源中获取，需要添加 Docker 自己的下载源。

1. 更新`apt`包索引并允许`apt`通过 HTTPS 连接到仓库：

   ```shell
   $ sudo apt-get update
   
   $ sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg-agent \
       software-properties-common
   ```

2. 添加 Docker 官方 GPG 密钥：

   ```shell
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

   确认添加是否成功可使用如下命令：

   ```shell
   $ sudo apt-key fingerprint 0EBFCD88
   # 若添加成功将产生如下的输出
   pub   rsa4096 2017-02-22 [SCEA]
         9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
   uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
   sub   rsa4096 2017-02-22 [S]
   ```

3. 添加`apt`仓库

   首先需要查看当前系统的内核版本：

   ```shell
   $ arch
   ```

   添加仓库：

   ```shell
   # x86_64 / amd64 版本使用
   $ sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
      
   # armhf 版本使用
   $ sudo add-apt-repository \
      "deb [arch=armhf] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
      
   # arm64 版本使用
   $ sudo add-apt-repository \
      "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   ```

#### 安装 Docker Engine

1. 更新`apt`包索引，并安装**最新版本**的 Docker Engine ：

   ```shell
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

2. 同样可以选择具体版本的 Docker 进行安装，这里就不再赘述，详见[ Docker 官方文档](https://docs.docker.com/engine/install)。

3. 验证 Docker 安装是否成功：

   ```shell
   $ sudo docker run hello-world
   # 若成功将下载 Hello World 示例镜像并运行
   ```

#### 卸载 Docker Engine

1. 卸载 Docker Engine 、 Docker CLI 以及 Containerd 相关包

   ```shell
   $ sudo apt-get purge docker-ce docker-ce-cli containerd.io
   ```

2. 已下载的镜像、容器、卷和自定义配置将会被保留，可以进行手动删除：

   ```
   $ sudo rm -rf /var/lib/docker
   ```



---

## Docker 配置流程

由于 Docker 的默认配置有时不能满足我们实际开发需求，需要进行进一步的自定义配置。

### Docker 国内源加速

Docker 默认源的速度谁用谁知道，别的先不管，镜像源必须换。

1. 添加配置文件

   ```shell
   $ sudo vim /etc/docker/daemon.json
   ```

   一般来说安装好 Docker 后这个文件是不存在的，直接创建即可。

2. 向配置文件中添加内容

   ```json
   {
       "registry-mirrors": [
           "https://registry.docker-cn.com", # Docker 中国区
           "http://hub-mirror.c.163.com", # 网易
           "http://docker.mirrors.ustc.edu.cn" # USTC
       ],
   }
   ```

   如果你有阿里云账号的话，强烈建议使用阿里云容器镜像服务中提供的加速镜像站，效果极佳。具体操作请自行 Google。

