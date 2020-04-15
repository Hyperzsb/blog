---
layout: post
title: GitLab - 北京理工GitLab添加SSH密钥指南
date: 2020-04-15
Author: Hyperzsb
tags: [git, gitlab, encrypt, BIT]
comments: true
toc: true
---

众所周知 GitHub、GitLab 等服务在远程进行`pull`或`push`操作时需要需要向服务端添加与本地机匹配的 SSH 公钥才能进行。

但是**我校的 GitLab **在公钥验证的时候往往会出现一些原因不明的问题...

下面来讨论一下相关问题的解决方案。

<!-- more -->

---

## SSH 密钥生成与装配

### 背景

在谈论问题及解决方案之前，先来看一下 SSH 密钥的在 GitHub 或 GitLab 中的用途：

>  Git 是一种分布式版本控制系统，意味着你可以在本地工作并将你的更改通过`push`操作共享到其它服务端上。在允许你进行`push`操作到 GitLab 服务器之前，你需要一个安全的通信频道进行信息交互。SSH 协议提供了这种安全性，并且允许你向 GitLab 远程服务器验证身份，而不用每次连接都提供你的用户名和密码。

目前GitLab支持 **RSA**、**DSA**、**ECDSA **和 **ED25519** 密钥。

但是根据**最佳实践**，你最好始终选用 **ED25519** 密钥（因为它的安全性和实际实用性都高于其它的密钥）。

当然，**RSA** 作为最流行、使用最广泛的非对称加密算法，会在一些老旧、不支持**ED25519**的服务器上工作地更好。根据你的实际需要进行选择。

### 生成密钥

1. 在Linux或macOS上打开一个终端，或者在Windows打开Power Shell、Git Bash或者WSL；

2. 生成一个 **ED25519** SSH密钥：

   ```shell
   $ ssh-keygen -t ed25519 -C "email@example.com"
   ```

   或者生成一个 **RSA** SSH密钥：

   ```shell
   $ ssh-keygen -t rsa -b 4096 -C "email@example.com"
   ```

   `-c`参数是可选的，用于添加一个密钥的说明注释以明确密钥信息；

3. 接下来你将会被要求输入密钥的存储路径（在你没有已存在的密钥或没有特殊要求的情况下下，直接选择系统默认的存储路径即可，即按下`Enter`键继续）；

   >  如果你想更改存储路径，需要进行一些配置，因为没有进行过实践，在此就不再赘述，有兴趣可以自行查看相关文档

4. 在你选定了存储路径后，系统会要求你输入一个密码来保护你的密钥，根据**最佳实践**最好设置一个密码，当然没有也完全没有问题，按下`Enter`继续；

   > 当你想添加或者更改你的 SSH 密钥密码时，可以使用如下命令：
   >
   > ```shell
   > $ ssh-keygen -p -f <keyname>
   > ```

### 添加密钥到 GitHub / GitLab

1. 复制你生成密钥的公钥，这里以 **ED25519** 公钥为例：

   **Windows：**

   ```shell
   # 如果你更改过密钥的存储路径，则以新的路径为准
   # clip命令是将输出流重定向到剪贴板
   $ cat ~/.ssh/id_ed25519.pub | clip
   ```

   **WSL / GNU/Linux（需要xclip包）：**

   ```shell
   $ xclip -sel clip < ~/.ssh/id_ed25519.pub
   ```

   **macOS：**

   ```shell
   $ pbcopy < ~/.ssh/id_ed25519.pub
   ```

   当然你可以直接找到对应的公钥文件（*.pub），打开并复制它。**但是要注意不要改变公钥文件的任何内容**；

2. 向你的 GitHub / GitLab 账户添加密钥：

   进入网站 -> 点击你的头像 -> 进入你的账户设置（**Settings**）

   找到 **SSH Keys** 选项 -> 创建新的 SSH Keys -> 将你的公钥粘贴进文本框中 -> 添加说明注释 -> 保存

### 测试密钥设置是否正确

可以使用如下命令

```shell
$ ssh -T <your_repository_url>
```

## 我校 GitLab 验证问题

### 密钥不定期失效

**这个是发生频率最高的问题**，而且并不是密钥设置时所设置的过期时间所产生的问题，我所设置的密钥为**永不过期**。具体错误如下：

```shell
# 之前已经成功地进行了push操作
# 在经过一段时间后
$ git push bit master
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:GPQ*******************************. /*每个密钥指纹都不同*/
Please contact your system administrator.
Add correct host key in /c/Users/Administrator/.ssh/known_hosts to get rid of this message.
Offending RSA key in /c/Users/Administrator/.ssh/known_hosts:5
RSA host key for git.proxy.bitnp.net has changed and you have requested strict checking.
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

？？？中间人攻击？那为什么 GitHub 就没事呢？？？

根据我的理解出现这个错误的可能原因是**双方签名不一致，或者公私钥不匹配**，导致系统认为身份认证出现问题。

在经历了向GitLab删除添加删除添加 SSH Keys 并不起作用后，歪打正着找到了一种解决方式：

### 解决方式

删除`~.ssh/known_hosts`文件中关于 GitLab 的相关配置后，再次运行`git push`重新建立认证。

> 删除掉相关配置后，我运行了`ssh -T <your_repository_url>`命令，提示建立认证关系，建立成功后，再次使用`git push`命令，发现依旧存在同样的问题（？？？）。
>
> 我的解决方式是在删除相关配置后，直接运行`git push`建立认证关系。

### IP 认证

我校在向外代理GitLab时为了安全性考虑需要定期向服务器注册你的IP地址，不注册的话也会拒绝访问。

### 解决方式

出于某些原因，就不说明具体的解决方式了。

只要你拥有网协通行证并且有相应权限，添加完 SSH Keys 并完成认证后，系统会进行认证方法说明。

## 参考

GitLab SSH README 帮助文档