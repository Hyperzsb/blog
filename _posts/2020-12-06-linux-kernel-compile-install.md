---
layout: post
title: Linux - 内核编译安装
date: 2020-12-06
Author: Hyperzsb
tags: [linux]
comments: true
toc: true
---

最近有一个 Linux 相关的实验课程需要进行 Linux 内核的编译和安装，感觉对于深入理解 Linux 系统是一个很好的契机。在网上查阅相关资料后，发现现有教程有很多错误和不足，同时也在实践过程中发现了一些常见问题，特做此记录。

<!-- more -->

---

## 系统环境

由于没有准备 Linux 的物理机，本实验通过虚拟机的方式进行，具体配置如下：

- 物理机系统：macOS Big Sur
- 虚拟机软件：VMware Fusion 专业版 12.1.0 (17195230)
- **虚拟机系统：Ubuntu 20.04.1 desktop amd64**
- **虚拟机资源分配：4个处理器内核、4GB RAM、至少60GB的磁盘空间（如果空间不足可能会造成编译失败）**
- **选择的内核版本：5.9.12**

其他说明：为了在编译和安装避免出现权限问题，可以在必要的情况下采用 `sudo` 命令



---

## 编译准备

### 安装编译所需依赖

- 更新软件源

  ```shell
  $ sudo apt update
  ```

- 安装所需依赖

  ```shell
  $ sudo apt install libncurses5-dev openssl libssl-dev build-essential pkg-config libc6-dev \
  				bison flex libelf-dev zlibc minizip libidn11-dev libidn11
  ```

  **值得注意的是，这些依赖可能并不是最小依赖集，可能在去掉一部分依赖后编译安装仍然能够进行，但是我并没有进行尝试**

### 下载 Linux 内核源代码

下载方法有很多，这里主要说两种：

- 进入 **[Linux 内核官网（kernel.org）](https://www.kernel.org/)**进行下载：选择一个合适的版本（最好是 `stable` 或者 `longterm`，当然其他版本的内核也是可以的，但是可能会造成问题）下载（`tarball`，即源代码）；或者直接点击那个硕大的下载按钮即可。

- 使用 `wget` 命令进行下载：

  ```shell
  $ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.9.12.tar.xz
  ```

  具体的版本号根据自己需要进行更改

### 解压 Linux 内核源代码

需要将源代码解压到指定的目录

```shell
$ sudo tar -xavf  linux-5.9.12.tar.xz  -C /usr/src
```



---

## 编译内核

### 配置内核编译选项

Linux 具有庞大的模块库，允许用户根据自身需求进行配置。但是其中大部分功能对于一般用户来说没有很大的意义，没有对其进行编译的必要，所以对编译选项进行合理的配置是极其必要的。

**为了简便和兼容性考虑，这里直接采用原有内核的配置文件来进行配置**

首先需要将现有内核的配置文件复制到新内核的配置目录中：

```shell
$ sudo cp /boot/config-5.4.0-56-generic /usr/src/inux-5.9.12/.config
```

需要注意的是，由于系统预装的内核版本往往不只有一个，在 `boot` 目录下往往有很多版本的启动配置文件。为了兼容性考虑，最好采用系统目前使用的内核版本配置文件。可以通过如下命令查看当前内核版本：

```shell
$ uname -r
5.4.0-56-generic
```

使用原有内核配置文件的方式有很多，譬如 `make config` 、`make menuconfig`、`make xconfig`、`make gconfig`、`make oldconfig`这里列举两种，**我采用了第一种方式**

- `make menuconfig`（基于文本的菜单式配置界面）

  这是一个图形化配置界面，在进入该界面后，只需依序选择 **Load -> OK -> Save -> OK -> Exit -> Exit** 即可完成配置，极其简单

- `make oldconfig`

  ```shell
  $ sudo make oldconfig
  ```

  由于新版本内核所需的配置同当前版本的配置文件并不完全兼容，所以在自动配置过程中，需要用户手动选择配置选项，在通常情况下，可以直接回车选择默认选项即可。**值得注意的是，这一部分的内容可能非常长**

### 编译内核

编译内核是一个较为费时的工作，**在我的虚拟机资源配置下，这一部分消耗了半个小时左右的时间**

```shell
# -j 参数后的数字根据你分配的核心数目选择合适的数字，代表同时进行编译的线程数
$ sudo make bzImage -j4
```

### 编译模块

同编译内核相比，编译模块将会花费更多的时间，**在我的虚拟机资源配置下，这一部分消耗了一个小时左右的时间**

```shell
# -j 参数后的数字根据你分配的核心数目选择合适的数字，代表同时进行编译的线程数
$ sudo make modules -j4
```

### 安装模块

在模块编译完成后，需要进行模块的安装

```shell
$ sudo make modules_install
```

在模块安装完成后，在 `/lib/modules` 目录下，可以看到新版内核的模块文件



---

## 安装内核

### 复制编译结果到指定目录

```shell
$ sudo mkinitramfs /lib/modules/5.9.12 -o /boot/initrd.img-5.9.12-generic
$ sudo cp /usr/src/linux-5.9.12/arch/x86/boot/bzImage /boot/vmlinuz-5.9.12-generic
$ sudo cp /usr/src/linux-5.9.12/System.map /boot/System.map-5.9.12 
```

### 更改引导信息

在更改引导信息前，最好将现有引导配置文件进行备份，以防不测

```shell
$ sudo cp /boot/grub/grub.cfg ~
```

自动更新引导信息可以在 `/boot/grub` 目录下执行 `update-grub2` 命令即可。在更新完成后，可以对比新旧 `grub.cfg` 文件，发现多了新版内核的相关配置



---

## 结束

在完成上述步骤后，如果没有意外，在系统重启之后，再此执行 `uname -r` 命令可以发现自己的内核已经更新成功了！



---

## 其他

如果在编译过程中出现中断、错误或其他导致编译未完成的情况，或者在编译完成后想重新编译内核，可以执行如下命令

```shell
# mrproper 选项将会删除包括配置文件在哪的一切编译信息及结果
$ sudo make mrproper
# clean 选项在删除编译结果的同时会保留配置文件
$ sudo make clean
```

