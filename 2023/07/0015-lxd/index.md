# LXD


## 简介

LXD 是一个现代、安全、强大的系统容器和虚拟机管理器。

它为在容器或虚拟机中运行和管理完整的Linux系统提供了统一的体验。LXD为大量Linux发行版提供映像，并围绕一个非常强大但非常简单的REST API构建。LXD可以从单台机器上的一个实例扩展到完整数据中心机架上的一个集群，使其适合于在开发和生产中运行工作负载。

LXD允许您轻松地设置一个感觉像小型私有云的系统。您可以以有效的方式运行任何类型的工作负载，同时保持资源优化。

如果您希望将不同的环境容器化或运行虚拟机，或者以经济有效的方式运行和管理基础设施，则应该考虑使用LXD。

### 安全

要确保LXD安装的安全性，请考虑以下几个方面:

保持您的操作系统是最新的，并安装所有可用的安全补丁。

只使用受支持的LXD版本(LTS版本或每月功能版本)。

限制对LXD守护进程和远程API的访问。

除非必要，否则不要使用特权容器。如果您使用特权容器，请设置适当的安全措施。有关更多信息，请参阅LXC安全性页面。

将网络接口配置为安全的。

## LXD 开始使用

### 容器和虚拟机

LXD为两种不同类型的实例提供支持：`系统容器`和`虚拟机`。

在运行系统容器时，LXD模拟完整操作系统的虚拟版本。为此，它使用运行在主机系统上的内核提供的功能。

在运行虚拟机时，LXD使用主机系统的硬件，但是内核由虚拟机提供。因此，虚拟机可以用来运行不同的操作系统。

### 应用程序容器 VS 系统容器

应用程序容器(如Docker提供的)将单个进程或应用程序打包。另一方面，系统容器模拟一个完整的操作系统，并允许您同时运行多个进程。

<center> <img src='https://linuxcontainers.org/lxd/docs/master/_images/application-vs-system-containers.svg' /></center>

因此，应用程序容器适合提供单独的组件，而系统容器则提供库、应用程序、数据库等的完整解决方案。此外，您可以使用系统容器来创建不同的用户空间，并隔离属于每个用户空间的所有进程，这不是应用程序容器的目的。

### 虚拟机 与 系统容器

虚拟机模拟物理机，使用来自完全隔离的操作系统的主机系统的硬件。另一方面，系统容器使用主机系统的操作系统内核，而不是创建自己的环境。如果运行多个系统容器，它们都共享相同的内核，这使得它们比虚拟机更快、更轻量。

使用LXD，您可以创建系统容器和虚拟机。如果所需的所有功能都与主机操作系统的内核兼容，则应该使用系统容器来利用更小的大小和更高的性能。如果您需要主机系统的操作系统内核不支持的功能，或者您希望运行一个完全不同的操作系统，请使用虚拟机。

<center> <img src='https://linuxcontainers.org/lxd/docs/master/_images/virtual-machines-vs-system-containers.svg' /></center>

### 运行环境依赖

1. Go 语言环境 >= 1.18 且 RAM >= 2GB
2. 内核

    2.1 Namespaces (pid, net, uts, ipc and mount)

    2.2 Seccomp

    2.3 Native Linux AIO (io_setup(2), etc.)

    2.4 Namespaces (user and cgroup)

    2.5 AppArmor (including Ubuntu patch for mount mediation)

    2.6 Control Groups (blkio, cpuset, devices, memory, pids and net_prio)

    2.7 CRIU (exact details to be found with CRIU upstream)

3. LXC >= 4.0.0

    3.1 apparmor (if using LXD’s AppArmor support)

    3.2 seccomp

4. QEMU >= 6.0

5. 其它库和头文件

    5.1 dqlite

    5.2 libacll

    5.3 libcap2

    5.4 liblz4(for dqlite)

    5.5 libuvl(for dqlite)

    5.6 libsqlite3 >= 3.25.0 (for dqlite)

    > 确保拥有这些依赖的 `-dev` 包

### 如何安装 LXD

`sudo snap install lxd --channel=5.0/stable`

### 初始化 LXD

在创建LXD实例之前，必须配置和初始化LXD。

#### 交互式配置

```shell
lxd init
```

该工具询问一系列问题以确定所需的配置。问题会根据你给出的答案动态调整。它们涵盖以下方面:

- Clustering：集群由多个LXD服务器组成。集群成员共享相同的分布式数据库，可以使用LXD客户机(lxc)或REST API统一管理。默认答案是no，这意味着没有启用集群。如果回答是，则可以连接到现有集群或创建一个集群。
- MAAS support：MAAS是一个开源工具，允许您从裸机服务器构建数据中心。默认答案是no，这意味着未启用MAAS支持。如果回答是，则可以连接到现有的MAAS服务器，并指定名称、URL和API密钥。
- Networking：为实例提供网络访问。您可以让LXD创建一个新的桥(推荐)，或者使用现有的网络桥或接口。您可以创建额外的桥并稍后将它们分配给实例。
- Storage pools：实例(和其他数据)存储在存储池中。出于测试目的，您可以创建一个环支持的存储池。但是，对于生产用途，您应该使用空分区(或完整磁盘)而不是循环支持的存储(因为循环支持的池速度较慢，并且无法减小其大小)。推荐使用zfs和btrfs作为后端。您可以稍后创建其他存储池。
