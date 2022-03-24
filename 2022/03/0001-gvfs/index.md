# gvfs


## gvfs 介绍

gvfs 是 GNOME 用户空间虚拟文件系统的缩写，从 GLib 2.15.1 开始加入系统，主要是对 I/O 的一层抽象。gvfs 提供了一些模块，这些模块由使用 libgio 的 API 应用程序自动使用。也通过 fuse，允许不适用 gio 的应用程序可以访问 gvfs 文件系统。

访问虚拟文件系统的前提是挂载文件系统，gvfs 提供了一个守护进程——`gvfsd`来协调处理模块，每个模块都与 `gvfsd` 通过 gio 进行通信信。gvfs带有一些后端，这些后端实现了回收站、sftp、ftp、webdav、以及本地数据相关支持，都是作为 gvfs 功能实现的一部分。

gvfs 还包括用于 gio 实现卷监视器。

### gvfs守护进程说明
|守护进程|功能|
|:---|:---|
|`gvfsd`|是gvfs主要守护程序。它通过`org.gtk.vfs.Daemon`名连接到dbus会话总线上，当gvfsd没有启动时候，可以通过gio客户端自动拉起gvfsd守护进程。<br/>gvfsd主要任务是充当模块的安装器和跟踪器。<br/>在需要的时候，它会产生新的后端，并跟踪它们的生命周期，维护已挂载的列表并创建与它们的直接连接。<br/>gvfsd还将启动gvfsd-fuse 并向其提供应在其中安装 fuse 文件系统的安装点|
|`gvfsd-fuse`|`gvfsd-fuse`维护FUSE(用户空间的文件系统)挂载，以使gvfs后端可用于 POSIX 应用程序。fuse文件系统的挂载点由 [PATH] 参数提供。其主要由 gvfsd 启动|
|`gvfsd-metadata`|`gvfsd-metadata`是一个守护进程，充当内部 gvfs 元数据存储的写入序列化程序。它是在 gio 客户端更改元数据时候自动启动。读取操作直接由客户端 gio 完成，并不需要运行守护程序|
|`gvfs-goa-volume-monitor`|支持 GNOME 在线账户|
|`gvfs-gphoto2-volume-monitor`|支持图片传输协议，如:gPhoto|
|`gvfs-mtp-volume-monitor`|支持媒体传输协议|
|`gvfs-udisks2-volume-monitor`|gvfs-udisks2-volume-monitor负责磁盘、介质、挂载和fstab桌面用户界面中显示的项目。特别的是gnome-shell以及使用 GLib API 的任何其它应用程序都在使用此过程中的信息。<br>*请注意：不要把它与udisks软件包中的udisksd和udisksctl混淆*|
|`gvfs-afc-volume-monitor`|苹果文件系统|
|`gvfsd-afc`|挂载IPhone/Ipod touch音量|
|`gvfsd-afp`|苹果文件协议卷|
|`gvfsd-afp-browse`|浏览apple归档协议卷|
|`gvfsd-archive`|挂载各种格式的归档文件|
|`gvfsd-burn`|提供刻录 CD 的位置|
|`gvfsd-cdda`|挂载音频 CD|
|`gvfsd-computer`|提供计算机 `computer:///` 支持|
|`gvfsd-dav`|挂载 DAV 文件系统|
|`gvfsd-dnssd`|浏览域名解析|
|`gvfsd-ftp`|挂载 FTP 文件系统|
|`gvfsd-gphoto2`|通过PTP挂载，这意味着gvfs通过libgphoto2通过 VFS 将相机上的照片显示给GNOME程序|
|`gvfsd-http`|通过 HTTP 挂载|
|`gvfsd-localtest`|测试后端|
|`gvfsd-mtp`|通过 mtp 挂载|
|`gvfsd-network`|提供 `network:///`支持|
|`gvfsd-nfs`|提供 nfs 协议支持|
|`gvfsd-recent`|提供最近访问支持|
|`gvfsd-sftp`|提供sftp支持|
|`gvfsd-smb`|提供samba支持|
|`gvfsd-smb-browse`|浏览windows共享文件系统的卷|
|`gvfsd-trash`|提供回收站支持|

## gvfsd

### gvfsd 代码模块功能梳理

### gvfsd 依赖文件及库分析
```
# 源码
daemon/main.c
daemon/mount.c

# 依赖库
gvfsddaemon
```

#### gvfsd 代码梳理

1. 解析命令行，是否使用 fuse、是否debug、是否仅仅打印版本信息 ...
2. 创建gvfs daemon
  - `g_vfs_daemon_new (TRUE, <replace>)`
  - `gvfs_get_socket_dir()`创建文件夹
  - 监听 `daemon` 的 `shutdown` 信号
  - session dbus总线上注册，名字为：`org.gtk.vfs.Daemon`
  - 如果需要 fuse 支持则启动 `gvfsd-fuse` 进程
  - session 上 dbus 注册成功就会调用 `gboolean mount_init()`
3. gvfsd 结束执行 `mount_finalize()`

#### `g_vfs_daemon_new (TRUE, <replace>)`
> `g_vfs_daemon_new ()` 声明与定义位于`daemon/gvfsdaemon.[hc]`文件里

1. 创建线程: `g_thread_pool_new()`
2. 初始化互斥锁: `g_mutex_init()`
3. 挂载数量初始化0、jobs初始化NULL、registered\_paths(hash)、client\_connections(hash)、session dbus的conn
4. `g_dbus_auth_observer_new()`
    1. `allow-mechanism`
    2. `authorize-authenticated-peer`
5. `gvfs_dbus_daemon_skeleton_new()`
    1. `handle-get-connection`
    2. `handle-cancel`
    3. `handle-list-monitor-implementations`
6. `gvfs_dbus_mountable_skeleton_new()`
    1. `handle-mount`

#### `gboolean mount_init ()`
> `gboolean mount_init ()`声明与定义在`daemon/mount.[hc]`文件里

1. `read_mountable_config ()`
    1. 获取环境变量 `GVFS_MOUNTABLE_EXTENSION`(默认是`.mount`文件)、`GVFS_MOUNTABLE_DIR`
    2. 读取 `GVFS_MOUNTABLE_DIR` 文件名需要有 `.mount` 后缀，文件夹是：`/usr/share/gvfs/mounts/` ~~/sys/fs/cgroup/~~ 解析 `xxx.mount`并保存到链表 `mountables` 全局变量
2. 创建管道 `pipe(reload_pipes)` (其中 `reload_pipes` 是全局的`static int reload_pipes[2];`)
    1. 监控管道，当管道有数据读入就会重新执行第一步流程`read_mountable_config ()`
3. 获取SESSION bus 的实例 `conn = g_bus_get_sync(SESSION, NULL, NULL)`
4. `mount_tracker = gvfs_dbus_mount_tracker_skeleton_new ()` 并分别监听信号(这些信号分别对应相应dbus功能)
    1. `handle-register-fuse`
    2. `handle-register-mount`
    3. `handle-mount-location`
    4. `handle-lookup-mount`
    5. `handle-lookup-mount-by-fuse-path`
    6. `handle-list-mounts`
    7. `handle-list-mounts2`
    8. `handle-list-mountable-info`
    9. `handle-list-mount-types`
    10. `handle-unregister-mount`
5. `g_dbus_interface_skeleton_export(mount_tracker,conn,G_VFS_DBUS_MOUNTTRACKER_PATH,NULL)`
    > `G_VFS_DBUS_MOUNTTRACKER_PATH` 是 `/org/gtk/vfs/mounttracker` 通过d-feet查看，此dbus提供了一系列接口，主要功能包括：支持的挂载类型、挂载信息、挂载点、通过 fuse 挂载的位置；同时也提供了挂载和卸载信号

#### `gvfs_dbus_mount_tracker_skeleton_new()`
一系列dbus对应的操作及一些接口声明


## udisks2

gvfs-udisks2-monitor 进程负责磁盘、media、挂载点和fstab在桌面环境挂载、访问。尤其是 gnome-shell 和文件管理器程序 (nautilus)以及其它使用 GLib 的 API 程序。
通常，只会显示可挂载文件系统的测盘或媒体，这些媒体在下边统称为 "设备"。

如果设备挂载点在 `/media/`、`$HOME`、`/run/media/$USER` 之外的文件夹，那么设备可能不会显示在用户界面。或者如果设备挂载点在 `/media/`、`$HOME`、`/run/media/$USER` 这些目录之下，但是挂载点是以 `.` 开头的，那么也不会显示。这需要使用使用挂载选项 `x-gvfs-show` 来强制显示，当然 `x-gvfs-hide` 也可以使挂载点隐藏。

设备的名称、图标、符号图标是根据某些特征选择的，比如：设备的文件系统标签、`x-gvfs-name=<value>`、`x-gvfs-icon=<value>` 和 `x-gvfs-symbolic-icon=<value>` 

在 `/etc/fstab` 添加自动挂载点时候，建议用户使用 `/dev/disk` 层次结构中稳定的符号连接，而不是内核名称 `sda`、`sdb` 等
### 热插拔
通过 `eSATA` 或 `USB` 连接的设备在物理机器上是可热插拔的。当物理设备连接到机器上或从机器上移除时候 `linux内核` 会把通知事件发送到用户空间，系统接受到此类事件并根据其配置对它们进行响应
- 设备驱动加载后会在 devfs (/dev) 下生成对应的设备节点，如果设备连接后 systemd-udevd 会根据配置对 /dev/ 下的设备节点进行增加
- 如果是块设备，systemd-udevd 通知 udisksd 和 gvfsd 以及 gvfs-udisks2-volume-monitor

### 一些 fstab 例子

#### 强制在用户界面隐藏设备
```
/dev/disk/by-id/ata-HITACHI_HTS723232A7A364_E3834563KRG2HN-part1   /home/davidz/Data  auto  defaults,x-gvfs-hide 0 0
```
#### 强制在用户界面显示设备(显示名字为 'My Movies')
```
/dev/disk/by-uuid/4CAE8E5B5AF47502   /Movies  auto   defaults,x-gvfs-show,x-gvfs-name=My%20Movies  0 0
```
#### 自定义设备图标
```
/dev/disk/by-uuid/4CAE8E5B5AF47502   /Movies  auto   defaults,x-gvfs-show,x-gvfs-name=My%20Movies,x-gvfs-icon=folder-videos,x-gvfs-symbolic-icon=folder-videos-symbolic  0 0
```
#### 强制在用户空间显示 NFS 挂载点
```
10.200.0.210:/tank/media  /mnt/Filer  nfs4  default,users,noauto,x-gvfs-show  0 0
```
### 一些[udev](http://udisks.freedesktop.org/docs/latest/udisks.8.html)例子
#### 不自动挂载金士顿U盘
```
SUBSYSTEMS=="usb", ENV{ID_VENDOR}=="*Kingston*", ENV{ID_MODEL}=="*DataTraveler*", ENV{UDISKS_AUTO}="0"
```
#### 自动挂载某设备但是不要求admin权限
```
ENV{ID_SERIAL}=="WDC_WD1002FAEX-00Y9A0_WD-WCAW30039835", ENV{UDISKS_AUTO}="1", ENV{UDISKS_SYSTEM}="0"
```
#### 给特殊设备特殊名字和图标
```
ENV{ID_MEDIA_PLAYER}=="apple-ipod", ENV{UDISKS_NAME}="David's iPod", ENV{UDISKS_ICON_NAME}="multimedia-player-ipod", ENV{UDISKS_SYMBOLIC_ICON_NAME}="multimedia-player-ipod-symbolic"
```
#### 确保此特殊设备不出现在用户界面
```
ENV{ID_SERIAL}=="HITACHI_HTS723232A7A364_E3834563KRG2HN", ENV{UDISKS_IGNORE}="1"
```



