---

layout: post
title: Jenkins - 安装配置（阿里云）
date: 2020-07-03
Author: Hyperzsb
tags: [jenkins, docker, ci&cd, cloud]
comments: true
toc: true
---

装完了 Docker ，接下来安装 Jenkins 。Jenkins 是一个成熟的 CI&CD 套件，详询**[ Jenkins 官网](https://www.jenkins.io/)**。

<!-- more -->



---

## Jenkins 安装流程

**系统版本：Ubuntu 18.04**

**流程同[ Jenkins 官方中文文档](https://www.jenkins.io/zh/doc/book/installing/)中给出的有些许差异，对于一些安装细节采用了其它博客中的一些内容，同官网不一致的地方将会进行进一步的说明。**

Jenkins 支持多种安装方式，一下给出常用的两种安装方式流程：

- Linux 本地安装
- Docker 安装

> 还有通过 jar 包、 war 包进行安装的方式，其具体流程详询[ Jenkins 官网](https://www.jenkins.io/)以及相关博客。

### Linux 本地安装

#### 安装步骤

1. 需要提前安装Java 8（Java 11）环境：

   ```shell
   # 更新包索引
   $ sudo apt-get update
   # 安装 OpenJDK8
   $ sudo apt-get install openjdk-8-jdk
   # 验证安装
   $ java -version
   ```

   > Java 11 环境安装 Jenkins 方式有所不同，可以参照[这里](https://www.jenkins.io/doc/administration/requirements/jenkins-on-java-11)进行安装。一般建议直接使用 Java 8 ，毕竟是运行最稳定，支持最广泛的Java环境。此外 旧版本的 Java 、Java 9 、Java 10 、Java 12 都是不支持的（截止到2020.07.03）。
   >
   > 如果没有安装 Java 而直接安装了 Jenkins ， Jenkins 在启动时会报错（产生类似`Starting Jenkins bash: /usr/bin/java: No such file or directory`的错误日志）。此时可以直接安装 Java 而不用将 Jenkins 卸载重装。安装好 Java 后直接使用`service jenkins start`启动 Jenkins 即可。

2. 依次执行如下命令：

   ```shell
   # 添加下载源公钥
   $ wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
   # 添加下载源
   $ sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
   	  /etc/apt/sources.list.d/jenkins.list'
   # 更新包索引
   $ sudo apt-get update
   # 安装 Jenkins
   $ sudo apt-get install jenkins
   ```

   - 安装成功后会自动创建`jenkins`用户来运行 Jenkins 服务。（**这一点很重要**）
   - 控制台日志会输出到`/var/log/jenkins/jenkins.log`。
   - 如果无法启动 Jenkins 可能是因为端口占用导致的问题，可以去`/etc/default/jenkins`将`----HTTP_PORT=8080----`中的`8080`更改为合适的端口号。

#### 卸载步骤

1. 移除软件包和配置数据：

   ```shell
   $ sudo apt-get purge jenkins
   ```

2. 可能需要手动清理配置数据：

   > 这里的说法很模糊，在使用 Jenkins 的过程中**可能**会创建或更改一些`apt`无法自动清理的配置文件，这时就需要手动清理。但是具体的清理位置和清理方式还是根据情况来决定（我在卸载时并不清楚是否需要手动清理，也不清楚该清理哪些文件，只是单纯地使用了`apt-get purge`，也不清楚是否会影响二次安装）。

#### 评价

**这种方式是不推荐使用的，原因有三：**

- 需要安装和配置相应的依赖环境：在某些场景下即使安装了 JDK 都可能因为环境变量的设置问题导致 Jenkins 无法定位到 JDK ，需要手动进行配置，操作较为复杂。

- 后期迁移不便：网上也有很多关于 Jenkins 迁移的博客，虽说可行，但是相较于基于 Docker 的迁移，可以说是毫无优势。

- **操作权限不足**：这是我放弃使用 Linux 本地安装 Jenkins 的最大原因。在执行流水线脚本时，通常会遇到权限不足导致的`Permission Denied`错误。虽说理论上可以通过赋予`jenkins`用户相应权限来解决这一问题，**但是我尝试了网上的各种方法，未果，最后甚至导致 Jenkins 服务出现异常无法启动，只好将其卸载······**

**当然，该方式也有其优势：**

- 不需要 Docker 相关的知识，使用较为方便。

- **可以直接操作宿主机文件系统，可以直接执行宿主机上的脚本，方便编译打包和部署。**（当然由于权限问题的存在，这个优势可能会受到限制）

### Docker 安装

#### 安装步骤

1. 安装 Docker：

   > 这里不再赘述。具体流程可以参考我的另一篇博客。

2. 选择 Jenkins 镜像：

   Jenkins 官方提供的镜像有很多，这里只说两个：

   - `jenkins/jenkins`：包含了 Jenkins 基本常用的功能模块（**不包含 Blue Ocean 插件**）。
   - `jenkinsci/blueocean`：该镜像包含了 LTS（长期支持，Long-term Support）Jenkins 版本，并且捆绑了所有Blue Ocean插件和功能。这意味着你不需要单独安装Blue Ocean插件。**一般来说推荐使用这个**。

3. 运行 Jenkins Docker 容器：

   - 方法一：

     > 该方法来自于[ Jenkins 官方中文文档](https://www.jenkins.io/zh/doc/book/installing/#在docker中下载并运行jenkins)，也是目前各类博客常见的实践类型。**我采用了这种方法。**

     ```shell
     $ docker run \
     	# 使用 root 用户进行操作
         --user root \
         # （可选）关闭时自动删除 Docker 容器。如果您需要退出 Jenkins ，这可以保持整洁
         --rm \
         # 在后台运行容器（即“分离”模式）并输出容器 ID 
         --detach \
         # 将容器8080端口映射到本地8080端口。将使用该端口访问 Jenkins
         --publish 8080:8080 \
         # （可选）将容器50000端口映射到本地50000端口。在多节点模式下，agent节点将通过该端口同master节点通信，具体信息请查阅资料
         --publish 50000:50000 \
         # 将容器中的 /var/jenkins_home 映射到卷 jenkins-data 上
         # 或者可以将容器中的 /var/jenkins_home 直接映射到宿主机的物理路径上，如
         # -v /var/jenkins_home:/var/jenkins_home
         -v jenkins-data:/var/jenkins_home \
         # 关键的一步，将宿主机的 Docker 映射到容器中，这样容器中就可以使用 Docker 相关功能
         -v /var/run/docker.sock:/var/run/docker.sock \
         # （可选）将容器 /home 目录映射到宿主机 ~ 目录中，实现同宿主机进行文件操作，方便部署
         -v "$HOME":/home \
         # Jenkins 镜像名称
         jenkinsci/blueocean
     ```

   - 方法二：

     > 该方法来自于[ Jenkins 官方英文文档](https://www.jenkins.io/doc/book/installing/#downloading-and-running-jenkins-in-docker)。**这种方法我还没有进行过实践。**

     ```shell
     # 创建一个网桥
     $ docker network create jenkins-network
     # 创建卷
     $ docker volume create jenkins-data
     $ docker volume create jenkins-docker-certs
     # 为了能在容器中使用 Docker 功能，需要启用 DinD 功能
     # 运行 dind 镜像
     $ docker container run \
     	# （可选）定义容器名称
     	--name jenkins-docker \
     	# （可选）关闭时自动删除 Docker 容器，这可以保持整洁
     	--rm \
     	# 在后台运行容器（即“分离”模式）并输出容器 ID
     	--detach \
     	# 目前想使用 DinD 功能需要授权访问才能正常的工作，进行授权
     	--privileged \
     	# 连接之前创建的网桥
     	--network jenkins-network \
     	# 使得 DinD 容器可以在网桥中通过 docker 名称访问到
     	--network-alias docker \
     	# 启用 Docker 服务中的 TLS 功能，具体意义不明，请查询资料
     	--env DOCKER_TLS_CERTDIR=/certs \
     	# 挂在之前创建的卷，具体意义不明，请查询资料
     	--volume jenkins-data:/var/jenkins_home \
     	--volume jenkins-docker-certs:/certs/client \
     	# 暴露 DinD 的 daemon 端口到宿主机，使得可以在宿主机上操作 DinD 中的 Docker 命令
     	--publish 2376:2376 \
     	# DinD 镜像名称
     	docker:dind
     $ docker container run \
     	# （可选）定义容器名称
     	--name jenkins-blueocean \
     	# （可选）关闭时自动删除 Docker 容器。如果您需要退出 Jenkins ，这可以保持整洁
     	--rm \
     	# 在后台运行容器（即“分离”模式）并输出容器 ID
     	--detach \
     	# 使得之前的 DinD 中的 Docker daemon 可以在该 Jenkins 可以通过 docker 名称访问到
     	--network jenkins \
     	# 一些环境变量，方便其它 Docker 或者 Docker Compose 进行操作和访问，具体意义不明，请查阅资料
     	--env DOCKER_HOST=tcp://docker:2376 \
     	--env DOCKER_CERT_PATH=/certs/client \
     	--env DOCKER_TLS_VERIFY=1 \
     	# 将容器8080端口映射到本地8080端口。将使用该端口访问 Jenkins
     	--publish 8080:8080 \
     	# （可选）将容器50000端口映射到本地50000端口。在多节点模式下，agent节点将通过该端口同master节点通信，具体信息请查阅资料
     	--publish 50000:50000 \
     	# 将容器中的 /var/jenkins_home 映射到卷 jenkins-data 上
     	--volume jenkins-data:/var/jenkins_home \
     	# 通过挂在卷启用 TSL 功能，具体意义不明，请查询资料
      	--volume jenkins-docker-certs:/certs/client:ro \
      	# Jenkins 镜像名称
     	jenkinsci/blueocean
     ```

   - 两种方法的比较：

     - 方法一主要采用 **DooD (Docker Outside of Docker)** 模式，可以简单地理解为直接使用宿主机的 Docker 接口。
     - 方法二采用的是 **DinD (Docker in Docker)** 模式，可以简单地理解为在一个 Docker 容器中运行另一个同宿主机 Docker 完全无关的 Docker Daemon 。
     - 由于方法二我并没有进行实践所以不好评价究竟是哪种方法更优。方法一各类博客资料相对齐全，方便参考，比较成熟；方法二是官方文档给出的最佳实践，应该有其采用的道理。究竟采用哪一种还是根据实际需要来决定。

#### 卸载步骤

> **该卸载步骤针对于安装方法一**。方法二的卸载方式应该同方法一差别不大，请自行实践。

1. 停止相关容器：

   ```shell
   $ docker stop <container_id>
   ```

2. 删除容器和卷：

   ```shell
   $ docker rm -v <container_id>
   ```

   > 对于挂载到宿主机目录的情况，需要手动进行清理。

#### 评价

**通过 Docker 的安装具有很大优势：**

- 无需复杂的依赖环境：只要有 Docker 环境就能完美运行。
- 不涉及复杂的权限分配问题：一般情况下无需提升权限即可完成相应的操作。
- 方便迁移

**当然也有不足：**

- 无法直接与宿主机进行通信：需要通过 SSH 才能运行宿主机的脚本，不方便部署，有舍近求远的意味。



---

## Jenkins 配置流程

### 初始化配置

1. 访问 Jenkins 实例：

   - 通过`http://localhost:8080`进行访问。
   - 通过`http://公网IP:8080`进行访问。

2. 初始化时需要你提供 administrator 密码：

   - 如果通过 Linux 本地安装，密码位置在本地计算机的`/var/lib/jenkins/secrets/initialAdminPassword`下。

   - 如果通过 Docker 安装，密码位置在 Jenkins 容器的`/var/jenkins_home/secrets/initialAdminPassword`下。

3. 自定义 Jenkins 插件：

   - 安装建议的插件：安装推荐的一组插件，这些插件基于最常见的用例。**推荐使用该选项。**
   - 选择要安装的插件：选择安装的插件集。

4. 创建第一个管理员用户：

   Jenkins 插件安装好后可以进行第一个管理员用户的注册。在信息填写完成后，Jenkins 就准备好了！

### 插件配置

1. [Blue Ocean](https://plugins.jenkins.io/blueocean)：

   Blue Ocean UI 很漂亮，也是进行多分支流水线开发的利器。若默认没有安装 Blue Ocean ，可以安装该插件。

2. [Embeddable Build Status Plugin](https://plugins.jenkins.io/embeddable-build-status)：

   该插件可以生成 build 状态 badge，可以将其嵌入到 GitHub 仓库的 README 文件中，用来显示构建状态。

3. Docker：

   如果在流水线中需要进行 Docker 镜像的运行、打包和部署，则需要安装 Docker 相关插件：

   - [Docker plugin](https://plugins.jenkins.io/docker-plugin)
   - [Docker Pipeline](https://plugins.jenkins.io/docker-workflow)

4. [Localization: Chinese (Simplified)](https://plugins.jenkins.io/localization-zh-cn)：

   由官方维护的简体中文语言包。

5. [Role-based Authorization Strategy](https://plugins.jenkins.io/role-strategy)：

   该插件可以使用角色对象模型实现对 Jenkins 的权限管理和账号权限分配等功能，十分强大。

6. [SSH plugin](https://plugins.jenkins.io/ssh)：

   如果在流水线中需要连接远程主机（包括在 Jenkins 容器中访问宿主机），则需要安装 SSH 相关插件。

7. [Timestamper](https://plugins.jenkins.io/timestamper)：

   该插件可以在控制台输出中加入时间戳。

8. [Workspace Cleanup Plugin](https://plugins.jenkins.io/ws-cleanup)：

   该插件可以用来清理每次构建后的工作区。

### SSH 配置

1. 定位配置选项：

   Jenkins 主页 -> 系统管理 -> 系统配置 -> SSH remote hosts

2. 配置远程主机 IP ：

   在`Hostname`一栏中输入远程主机的 IP 地址。

3. 配置端口：

   在`Port`一栏中输入远程主机 SSH 端口，一般为`22`。

4. 配置凭据：

   在`Credentials`一栏右侧有一`添加`按钮，点击，选择`Jenkins`添加凭据：

   - `Domain` 一栏选择`全局凭据 (unrestricted)`；
   - `类型`一栏选择 `Username with password` ;
   - `范围`一栏选择`全局 (Jenkins, nodes, items, all child items, etc)`；
   - `用户名`一栏输入 SSH 登录用户用户名；
   - `密码`一栏输入 SSH 登录用户密码；
   - `ID`一栏输入自定义内容，用于唯一标识该凭据；
   - `描述`一栏输入自定义内容，用于描述该凭据。

5. 保存：

   点击`保存`按钮。

### Jenkins 时间设置

>  默认情况下 Jenkins 使用的是 UTC 时区，导致 Jenkins 时间显示同国内相差8小时，使用颇为不便。

可以在 `$JENKINS_HOME` 目录下创建文件 `init.groovy` 文件；或者创建目录 `$JENKINS_HOME/init.groovy.d/` ，并在这个目录下面创建任何以 `.groovy` 结尾的文件。在任意一个上述文件内填入下面内容即可，这些内容会在 Jenkins 每次启动后加载：

```java
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')
```



---

## Jenkins 使用技巧

### 删除状态为中止或失败的构建

在我们进行构建是，往往会产生被我们手动中止的（ABORTED）和失败的（FAILURE）的构建，这些构建往往会占据一定磁盘空间，并影响我们定位到关键构建，这时我们就需要将其进行删除。

但是手动删除每一个构建往往不是一个聪明的选择。幸运的是，Jenkins 为我们提供了通过编程来自动完成这一工作的接口。

> Type in an arbitrary [Groovy script](http://www.groovy-lang.org/) and execute it on the server. Useful for trouble-shooting and diagnostics. 

1. 进入脚本命令行界面：

   Jenkisn Homepage :arrow_right: Manage Jenkins :arrow_right: Script Console

2. 输入以下命令并运行：

   ```groovy
   def jobName = "your-job-name"
   def statusToDelete = "ABORTED"
    
   Jenkins.instance.getItemByFullName(jobName).builds.each {
     if(it.getResult().toString().equals(statusToDelete))
     	it.delete();
   }
   ```



---

## 参考

- [ Jenkins 官方中文文档](https://www.jenkins.io/zh/doc/book/)
- [ Jenkins 官方英文文档](https://www.jenkins.io/doc/book/)
- [jenkins如何配置ssh服务器](https://blog.csdn.net/liujingqiu/article/details/58584559)
- [永久修改以容器化方式运行的Jenkins系统时间](https://www.jianshu.com/p/47d767cf893d)