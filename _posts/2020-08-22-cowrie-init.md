---
layout: post
title: Cowrie - 安装配置
date: 2020-08-22
Author: Hyperzsb
tags: [cybersecurity, honeypot]
comments: true
toc: true
---

最近在搞 Honeypot / Honeynet 相关技术，无意间发现了 Cowrie 这个极其契合我需求的开源蜜罐，也打算利用它进行自己的课题，这里就来写一下 Cowrie 的安装与配置。

<!-- more -->



---

## Cowrie 简介

> "Cowrie is a medium interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by the attacker. Cowrie also functions as an SSH and telnet proxy to observe attacker behavior to another system."

Cowrie 是一款中、高交互的 Telnet / SSH 蜜罐，支持多种工作模式。用户可以通过配置实现极高的定制性。同时它还支持多种数据持久化工具（MySQL、SQLite、MongoDB、Redis等数据库）、数据可视化工具（ELK stack、Kippo stack）、日志分析工具（splunk）和恶意文件 / 软件分析工具（VirusTotal、cuckoo、csirtg）。总的来说十分强大。

**更多内容详询[ Cowrie 仓库](https://github.com/cowrie/cowrie)**。



---

## Cowrie 手动安装流程

> 此安装教程基本上沿用了[官方教程](https://cowrie.readthedocs.io/en/latest/INSTALL.html)，对于一些可能出现问题的地方加入了详细的说明。

> 此安装针对基于 Debian 的 Linux 发行版（Ubuntu、Debian 等）。

### 安装依赖

安装 Python 相关依赖：

```shell
$ sudo apt-get install git python-virtualenv \
	libssl-dev \
    libffi-dev \
    build-essential \
    libpython3-dev \
    python3-minimal \
    virtualenv \
    authbind
```

### 创建 Cowrie 专用账户

>  由于安全原因，Cowrie 被设计成只能在非 root 用户下运行。

运行如下命令创建一个名为 `cowrie` ，密码自定的账户：

```shell
$ sudo adduser cowrie
Adding user `cowrie' ...
Adding new group `cowrie' (1002) ...
Adding new user `cowrie' (1002) with group `cowrie' ...
Creating home directory `/home/cowrie' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: # 输入用户密码
Retype new UNIX password: # 再次输入用户密码
passwd: password updated successfully
Changing the user information for hyperzsb
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
```

### 下载 Cowrie 源代码

将代码从仓库 `clone` 下来：

```shell
$ git clone http://github.com/cowrie/cowrie
Cloning into 'cowrie'...
remote: Counting objects: 2965, done.
remote: Compressing objects: 100% (1025/1025), done.
remote: Total 2965 (delta 1908), reused 2962 (delta 1905), pack-reused 0
Receiving objects: 100% (2965/2965), 3.41 MiB | 2.57 MiB/s, done.
Resolving deltas: 100% (1908/1908), done.
Checking connectivity... done.
```

### 创建 Python 虚拟环境

> Cowrie 是基于 Python 虚拟机运行的。

执行如下命令：

```shell
# 第一个 cowrie 目录是 cowrie 用户主目录，第二个 cowrie 目录是 clone 的代码目录
$ pwd
/home/cowrie/cowrie
# 创建虚拟机环境
$ virtualenv --python=python3 cowrie-env
New python executable in ./cowrie/cowrie-env/bin/python
Installing setuptools, pip, wheel...done.
```

**值得注意的是**，在执行 `virtualenv` 命令时，因为众所周知的原因有很大概率会出现 `Read time out` 等网络超时错误，导致虚拟环境创建失败，这时可以使用如下命令：

```shell
$ virtualenv --python=python3 cowrie-env --no-setuptools --no-pip --no-wheel
```

这样在初始化虚拟环境时配置 `pip` 等工具，就避免了网络超时的问题。

### 激活虚拟环境

执行如下命令：

```shell
$ pwd
/home/cowrie/cowrie
$ source cowrie-env/bin/activate
# 升级 pip
(cowrie-env) $ pip install --upgrade pip
# 安装 cowrie 相关依赖
(cowrie-env) $ pip install --upgrade -r requirements.txt
```

**值得注意的是**，在执行 `pip` 命令时，因为因为众所周知的原因有很大概率会出现 `Read time out` 错误或者下载速度过慢，导致安装依赖失败，此时可以使用如下命令：

```shell
# 临时使用清华的 pip 软件源加速下载
(cowrie-env) $ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade -r requirements.txt
```

### 自定义配置文件

Cowrie 的配置文件为 `cowrie/etc` 目录下的 `cowrie.cfg.dist`、`cowrie.cfg` 和 `userdb.example`。其中：

- `cowrie.cfg.dist` ：该文件是 cowrie 默认的配置文件，包括了所有 cowrie 的可配置项。用户可以直接在这个文件进行修改以自定义配置。

- `cowrie.cfg` ：该文件默认情况下在 `cowrie/etc` 目录中是不存在的，需要用户自己创建。自定义配置只需将 `cowrie.cfg.dist` 文件中的相应内容复制到该文件中即可。`cowrie.cfg` 具有第一优先级。

  例如想启用 Telnet 蜜罐功能，只需在 `cowrie.cfg` 文件中添加如下内容：

  ```
  [telnet]
  enabled = true
  ```

- `userdb.example` ：该文件存储了默认的账户密码对，用于攻击者登录蜜罐。可以根据需要进行修改和添加。

### 启动 Cowrie

使用如下命令

```shell
$ pwd
/home/cowrie/cowrie
$ bin/cowrie start
Using activated Python virtual environment "/home/cowrie/cowrie/cowrie-env"
version check
Starting cowrie: [twistd   --umask=0022 --pidfile=var/run/cowrie.pid --logger cowrie.python.logfile.logger cowrie ]...
```

### 停止 Cowrie

使用如下命令：

```shell
$ bin/cowrie stop
Stopping cowrie...
```

`cowrie` 命令还支持其它选项：

```shell
$ bin/cowrie --help
usage: bin/cowrie <start|stop|force-stop|restart|status>
```

### 监听 22 端口

> 这一部分根据需要来配置，这里就不再赘述，请参考[官方文档](https://cowrie.readthedocs.io/en/latest/INSTALL.html#step-7-listening-on-port-22-optional)。



---

## Cowrie Docker 安装流程

Cowrie 十分贴心地提供了 Docker 版本，方便规模化部署和 Honeynet 的建设。

> 这里基本沿用官网文档的流程，**但是有一些潜在问题文档中并没有给出，后期会持续更新**。

### 运行 Cowrie 容器

执行如下代码：

```shell
$ docker run -p 2222:2222/tcp cowrie/cowrie
```

### 测试 Cowrie 容器

```shell
$ ssh -p 2222 root@localhost
```

### 进入 Cowrie 容器

```shell
$ docker exec -it <container_id> /bin/bash
```

### 一些问题

- 挂载卷：例如挂载包含配置文件的 `etc`  目录，应当采用如下命令：

  ```shell
  $ docker run --rm -p 2222:2222/tcp -v E:\cowrie\etc:/cowrie/cowrie-git/etc cowrie/cowrie
  ```

  其中 `E:\cowrie\etc` 目录必须已经包含有 `cowrie.cfg.dist`、`cowrie.cfg` 和 `userdb.example` 等文件，cowrie 容器才能成功运行。

  > `/cowrie/cowrie-git/etc` 目录为 cowrie 容器配置文件目录。

  > `/cowrie/cowrie-git/var` 目录为 cowrie 容器日志文件目录。



---

## Cowrie 配置流程

> 这一部分的内容会后续更新。