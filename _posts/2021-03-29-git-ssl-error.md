---
layout: post
title: Git - SSL_ERROR_SYSCALL 问题解决
date: 2021-03-29
Author: Hyperzsb
tags: [git]
comments: true
toc: true
---

最近在使用 Git 时发现在使用 `git clone` 或 `git pull` 等需要访问远程仓库的操作时，总是无法连接 GitHub 服务器，很是郁闷。在查阅了多方资料后，总结了这一问题的解决方法，希望能解决这一问题。

<!-- more -->

---

## 问题再现

### 系统版本

macOS Big Sur 11.2.3

### 使用工具

Terminal

### 使用命令

```sh
$ git clone https://github.com/xxx/xxx.git
fatal: unable to access 'https://github.com/xxx/xxx.git/': LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443 
```

---

## 解决方案

经过查阅各方资料，发现这个问题并非一个少见的问题，有大量的 Git 用户（尤其是 macOS 用户）遇到了这个问题，还有一些用户在使用 Gem 等工具时也出现了同样的问题。

目前可以查到的解决方案包括以下几种：

### 重启计算机

众所周知，重启解决 90% 的问题，有时候重启电脑可以直接解决这一问题。

>  该方法对我无效。

### 修改 Git 网络配置

对于 Git 的网络设置，可以采用以下方式进行修改：

- 删除 HTTP / HTTPS 代理设置：

  - 可以直接修改全局 Git 配置文件进行修改：

    ```sh
    $ vim ~/.gitconfig
    ```

    删除文件中 HTTP / HTTPS 的相关配置即可

  - 也可以使用命令进行修改：

    ```sh
    $ git config --global --unset http.proxy
    $ git config --global --unset https.proxy
    ```

  > 该方法对我无效。

- 更改 HTTP / HTTPS 加密库：

  由于该问题是 LibreSSL 库报错，可以修改 Git 使用 OpenSSL 库进行 HTTPS 的通信。

  ```
  $ git config --global http.sslBackend "openssl"
  ```

  > 该方法对我无效。

### 修改计算机网络配置

由于使用 IPv6 的原因，可能会导致这一问题的出现。可以配置计算机不使用 IPv6，故使用以下命令：

```sh
$ networksetup -setv6off Wi-Fi
```

如果有需要，可以再将配置修改回来：

```sh
$ networksetup -setv6automatic Wi-Fi
```

> 该方法好像对我有效

---

## 结论

再试了很多方式之后，只有第三种起了作用，但是问题的解决是否是由于同时使用了多个解决方案，还难有定论。希望可以在将来彻底解决这一问题。

---

## 参考

- [SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443](https://stackoverflow.com/questions/48987512/ssl-connect-ssl-error-syscall-in-connection-to-github-com443)

