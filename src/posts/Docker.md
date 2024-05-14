---
title: 初识Docker
description: 这里讲述了Docker的基础知识，在这里你可以了解到Docker是什么，有什么用等等基础性的知识
author: Dadajia
date: 2024-03-01
tags:
  - basic
  - Docker
---

# Docker
### 一、初识 Docker

在写代码时，常常会接触到好几个环境：开发环境、测试环境以及生产环境

在不同的环境上运行代码时，因为环境不同可能会导致代码运行错误，因此我们常常把代码和环境打包起来一起移送，这个打包起来的东西，我们就叫做 **容器**。

Docker 是一种**容器技术**，解决**软件跨环境迁移**的问题。

#### 1.1 Docker 的概念

- Docker 是一个开源的应用容器引擎
- 诞生于 2013 年初，基于 Go 语言实现，dotCloud 公司出品（后改名为 Docker Inc）
- Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上
- 容器是完全使用沙箱机制，相互隔离
- 容器性能开销极低
- Docker 从 17.03 版本之后分为 CE（Community Edition：社区版）和 EE（Enterprise Edition：企业版）

#### 1.2 安装 Docker

Docker 可以运行在 MAC、Windows、CentOS、Unbuntu等操作系统上，本文基于 CentOS 7 安装 Docker。

安装网址：https://www.docker.com

```shell
# 1. yum 包更新到最新
yum update
# 2. 安装需要的软件包 yum-util 提供 yum-config-manager功能，另外两个是devicemapper驱动依赖的
yum install -y -yum-utils device-mapper-persistent-data lvm2
# 3. 设置 yum 源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 4. 安装docker,出现的输入界面都按y
yum install -y docker-ce
# 5. 查看docker版本，检验是否成功
docker -v
```

#### 1.3 Docker 架构

![nipaste_2024-03-11_14-54-0](Docker.assets/Snipaste_2024-03-11_14-54-02.png)

上图中的 Clients 代表的是 客户端，我们后续的一些操作都是在客户端中进行的。

Host 是 docker 的核心，其中 local host 显然就代表着本机，而remote host 则代表远程机器。

 在我们安装完 docker 后他会以 daemon（守护进程）的形式存在（后台运行的进程）。

 在本机或者远程机器中，container 代表 容器，image 代表 镜像。其中 container 和 image 的定位有点类似与 类与对象，image 是根据 container 创建出的实例。

下面我们详细介绍一下 Image 与 Container：

- **Dokcer镜像（Image）**，镜像是对静态的定义。它就相当于一个 root 文件系统，比如官方镜像 ubuntu:16.04 就包含了完整一套 Ubuntu16.04最小系统的root文件系统。

- **Dokcer容器（Container）**，容器是镜像运行时的实体，它可以被创建、启动、停止、删除、暂停等。

而所谓**仓库（Repository）**，就是一个代码控制中心，用来保存镜像。

显然镜像源于 docker 提供的远程仓库，而这个远程仓库又分为 官方提供的 Docker hub 和 自己搭建的 private registry。

#### 1.4 配置 Docker 镜像加速器

因为 docker 仓库位于国外，在不使用科学上网的手段，访问速度较慢，所以这里我们先配置镜像加速器。

而镜像加速器，比较常用的则就是以下几种，自己可以看着配置：

- USTC:中科大加速器（https://docker.mirrors.ustc.edu.cn）
- 阿里云
- 网易云
- 腾讯云

这里我们使用 阿里云：

首先登录，随后搜索 容器镜像服务。

![nipaste_2024-03-25_20-52-0](Docker.assets/Snipaste_2024-03-25_20-52-01.png)

随后打开 管理控制台，找到 镜像加速器 复制（每个人的地址都是特有，这里就不显示了）

![nipaste_2024-03-25_20-53-1](Docker.assets/Snipaste_2024-03-25_20-53-12.png)

随后执行下面代码

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["你的镜像地址"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

如果失败，没关系，可以看这篇[文章](https://www.cnblogs.com/tianmingzh/articles/15861662.html)，安装一下 **docker desktop**。之后我们也会详细介绍 docker desktop。

### 二、Docker 命令

由之前的架构图，显然，我们有以下几种命令：

1. daemon 命令
2. container 命令
3. image 命令

#### 2.1 daemon 命令

| 命令                      | 效果           |
| ------------------------- | -------------- |
| systemctl start docker    | 启动docker     |
| stystemctl status docker  | 查看docker状态 |
| stystemctl stop docker    | 关闭docker     |
| stystemctl restart docker | 重启docker     |
| stystemctl enable docker  | 自动启动docker |

ps.上述的stystemctl 属于系统命令，要根据自己使用的操作系统变化

#### 2.2 image 命令

一般不确定是否有该版本的 镜像文件，可以上 docker hub 查找一下

| 命令                             | 效果                                       |
| -------------------------------- | ------------------------------------------ |
| docker images                    | 查看镜像文件（软件和需要的运行环境）       |
| docker search xxx                | 搜索xxx镜像文件                            |
| docker pull xxx[:版本号]         | 拉取对应版本的镜像文件，不写版本号默认最新 |
| docker rmi 镜像ID/仓库名         | 删除镜像                                   |
| docker rmi \`docker images -q \` | 删除所有镜像                               |

#### 2.3 container 命令

我们运行的 wsl 被叫做宿主机，而实际又会创建一个空间，即容器。

| 命令                                        | 效果                            |
| ------------------------------------------- | ------------------------------- |
| docker run -it --name=c1 centos:7 /bin/bash | 创建容器（centos7、容器名为c1） |
| exit                                        | 退出容器                        |
| docker ps [-a // 加上查看所有容器]          | 查看容器                        |
| docker exec -it c2 /bin/bash                | 进入容器                        |
| docker stop 容器名/id                       | 关闭容器                        |
| docker start 容器名/id                      | 启动容器                        |
| docker rm 容器名/id                         | 删除容器                        |
| docker inspect 容器名/id                    | 查看容器信息                    |

参数说明：

- -i：保持容器运行。通常与 -t 同时使用。加入 it 这两个参数后，容器创建后会自动进入容器中，退出容器后，容器自动关闭
- -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用
- -d：以 daemon（守护后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭
- -it 创建的容器一般被成为交互式容器，-id创建的容器一般被称为守护式容器
- --name：为创建的容器命名

### 三、Docker 容器的数据卷

#### 3.1 数据卷概念及作用

在某些情况下我们迫不得已删除了容器，那么我们曾经存在于docker当中的一些数据怎么办呢？同时在使用docker时，我们能和外部机器交换文件吗？还有，在同一个宿主机中，可能存在多个容器，我们能让容器之间的数据产生交互吗？

以上问题，都可以被数据卷解决。

那么，什么是数据卷？

**数据卷**，是**宿主机中**的一个目录或文件，当容器目录和数据卷目录绑定后，双方的修改立即会同步，同时一个数据卷可以被多个容器同时挂载，一个容器也可以被挂载多个数据卷。

因此对于以上问题：

1. 即使删除docker，其数据卷仍然存在
2. 可以通过数据卷与外部机器发生数据交换
3. 通过多个容器绑定一个数据卷，从而进行交互

总而言之，数据卷有以下作用：

- 容器数据持久化
- 外部机器和容器间接通信
- 容器之间数据交换

#### 3.2 配置数据卷

创建启动容器时，使用 -v 参数 设置数据卷

```shell
docker run ... -v 宿主机目录（文件）：容器内目录（文件）...
```

注意事项：

1. 目录必须是绝对路径
2. 如果目录不存在，会自动创建
3. 可以挂载多个数据卷

#### 3.3 配置数据卷容器

多容器进行数据交互时，通过上述文本可知，可以采用多个容器挂载一个数据卷，但是多个容器难免会产生一些问题和混乱，因此我们还可以通过数据卷容器来实现。

![nipaste_2024-03-25_22-43-5](Docker.assets/Snipaste_2024-03-25_22-43-56.png)

如上图所述，我们引入一个 Data Container 作为存储 Container1 和2的数据卷容器，即使c1或者c2出现了问题，我们还可以借助其他路径访问数据卷而不会对其余容器造成干扰。

以下是配置数据卷容器的步骤：

1. 创建启动c3数据卷容器，使用 -v 参数 设置数据卷

   ```shell
   docker run -it --name=c3 -v /volume centos:7 /bin/bash
   ```

2. 创建启动 c1 c2 容器，使用 --volumens-from 参数 设置数据卷

   ```shell
   docker run -it --name=c1 --volumes-from c3 centos:7 /bin/bash
   docker run -it --name=c2 --volumes-form c3 centos:7 /bin/bash
   ```







### 网络

#### 11.1 docker

初始化 docker 环境

```shell
docker rmi -f $(docker images -aq)
```

Linux 下获取 ip地址

```shell
ip addr
```

```shell
# 随机启动一个

docker run -d -P --name tomcat01

PS C:\Users\user> docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN group default qlen 1000
    link/tunnel6 :: brd ::
12: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 思考 ，linux 能不能 ping 通容器内部
#可以#

# linx 可以 ping 通 docker 容器内部
```

>  原理

1. 我们每启动一个 docker 容器，docker 就会给 docker 容器分配个 ip，我们安装了docker，就会有个网卡 docker0，桥接模式，使用的技术是 evth-path技术！
2. 在启动一个容器测试的时候，发现又多了一个网卡

```
# 我们发现这个容器带来的网卡，都是一对对的
# evth-path 就是一堆虚拟的设备接口，他们都是成对的出现，一段连着协议，一段彼此相连
# 正因为有这个特性，evth-path 充当一个桥梁，链接各种虚拟网络设备
# OpenStac，Docker 容器之间的连接，OVS 连接，都是使用 evth-pair 技术
```

1. 测试 tomcat01 和 tomcat02 是否可以ping 通    
   1. 容器和容器之间是可以互相 ping 通的

结论：

tomcat01 和 tomcat02 是公用个路由器：docker0，所有的容器不指定网络的情况，都是 docker 容器会给我们分配一个默认可用的 ip

>  小结

Docker 中的所有的网络接口都是虚拟的，虚拟的转发效率高！（内网传递文件！）

只要容器删除，对应网桥一对就没了！

#### 11.2 – link

解决两个独立的 docker 项目不能互相 ping 的问题

```
docker run -d -P --name tomcat03 --link tomcat02 tomcat
docker exec -it tomcat03 ping tomcat02

# 反向可以 ping 通吗？  不可以，需要配置
```

可以通过 insepect 可以看到很多东西

#### 11.3 自定义网络

>  查看所有 docker 网络

```
PS C:\Users\Anshu Chen> docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
de74c8203497        bridge              bridge              local
550920e46e30        host                host                local
105ce7ef023f        none                null                local
```

网络模式：桥接 docker （默认）

none：不配置网络

host：和宿主机共享网络

container：容器网络连通（用的少！局限很大）

测试

```
# 我们直接启动的命令 --net bridge，而这个就是我们的 docker0
docker run -d -P --name tomcat01 tomcat
docker -d -P --name tomcat01 --net bridge tomcat

# docker0 特点：默认域名不能访问， --link 可以打通连接

# --driver bridge 桥接
# --subnet 192.168.0.0/16
# --gateway 
PS C:\Users\Anshu Chen> docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
# 我们的网络就创建好了
c81d519694cb79a3b84dda1e20b0ce76521fcd7dbd9f25ff9db38c4717dc4ec1
PS C:\Users\Anshu Chen> docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
de74c8203497        bridge              bridge              local
550920e46e30        host                host                local
c81d519694cb        mynet               bridge              local
105ce7ef023f        none                null                local

# 启动
docker run -d -P --name tomcat-net-01 --net mynet tomcat

PS C:\Users\Anshu Chen> docker run -d -P --name tomcat-net-01 --net mynet tomcat
29efa1bcdb47c157b3737caa313aba619ef7c521f75babe6fff606443d7b0af5
PS C:\Users\Anshu Chen> docker run -d -P --name tomcat-net-02 --net mynet tomcat
29c71809d2ba670ea7ccc9d64e64a784b813458273187584aca030d6cc7af0cc
PS C:\Users\Anshu Chen> docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.115 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.108 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.067 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=4 ttl=64 time=0.088 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=5 ttl=64 time=0.077 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=6 ttl=64 time=0.046 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=7 ttl=64 time=0.052 ms
```

我们自定义的网络 docker都已经帮我们维护了对应的关系，推荐我们平时这样使用网络

好处：

redis - 不同的集群使用不同的网络，保证集群是安全和健康的

mysql - 不同的集群使用不同的网络，保证集群是安全和健康的

网络连通

```
# 测试连通
docker netword connect mynet tomcat01

# 连通之后，就是将 tomcat01 放了 mynet 网络下

# 一个容器两个 ip 地址
# 云服务器：公网 ip  私有 ip

# 01 连通
docker exec -it tomcat01 ping tomcat-net-01

# 02 是依然打不通的
docker exec -it tomcat02 ping tomcat-net-01
```

结论：

- 假设要跨操作别人，就需要使用 docker network connect 连通！