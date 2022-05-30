# 自动挂载支持


## 自动挂载支持
希望实现自动挂载支持的文件系统也可以获得支持(例如可以在 `fs/afs/` 中找到kAFS，在 `fs/nfs/` 中找到NFS)。这个功能包括允许执行核内挂载以及请求降级挂载点。降级后的挂载点用户空间也可以访问。

### 核内自动挂载
> 参见: [autofs的“挂载陷阱”部分](https://www.kernel.org/doc/html/latest/filesystems/autofs.html)-它是如何工作的

在用户空间中，你可以这样做:
```shell
[root@andromeda root]# mount -t <文件系统> <挂载设备> <挂载点>
[root@andromeda root]# ls /afs
asd  cambridge  cambridge.redhat.com  grand.central.org
[root@andromeda root]# ls /afs/cambridge
afsdoc
[root@andromeda root]# ls /afs/cambridge/afsdoc/
ChangeLog  html  LICENSE  pdf  RELNOTES-1.2.2
```

然后如果你看一下挂载点目录，你会看到这样的东西:
```shell
[root@andromeda root]# cat /proc/mounts
...
#root.afs. /afs afs rw 0 0
#root.cell. /afs/cambridge.redhat.com afs rw 0 0
#afsdoc. /afs/cambridge.redhat.com/afsdoc afs rw 0 0
```

### 自动挂载点过期
挂载点的自动过期是很容易的，前提是您已经在单独列出的自动挂载过程中挂载了将要过期的挂载点。

要想过期，你需要遵循以下步骤:
1. 创建至少一个列表，以便将过期的`vfmount`挂起。
2. 当在`->d_automount`方法中创建一个新的挂载点时，使用`mnt_set_expiry()`将mnt添加到列表中:
```c
mnt_set_expiry(newmnt, &afs_vfsmounts);
```
3. 当您希望挂载点过期时，使用指向该列表的指针调用`mark_mounts_for_expiry()`。这将处理列表，将其上的每个`vfmount`标记为下一次调用的潜在到期。

如果一个`vfmount`已经被标记为过期，并且它的使用计数为`1`(它只被它的父`vfmount`引用)，那么它将从命名空间中删除并丢弃(实际上是卸载)。

最简单的方法可能是定期调用它，使用某种定时事件来驱动它。

通过调用`mntput`来清除过期标志。这意味着只有在上次访问挂载点之后的第二次过期请求时才会发生过期。

如果一个挂载点被移动，它将从过期列表中删除。如果绑定挂载是在一个可过期挂载上进行的，那么新的`vfmount`将不在过期列表中，也不会过期。

如果复制了一个名称空间，那么其中包含的所有挂载点都将被复制，过期列表中挂载点的副本将被添加到相同的过期列表中。

### 用户空间驱动到期
作为一种替代方案，用户空间可以请求任何挂载点的过期(尽管有些挂载点会被拒绝——例如卸载rootfs)。它通过将`MNT_EXPIRE`标志传递给`umount()`来实现这一点。该标志被认为与`MNT_FORCE`和`MNT_DETACH`不兼容。

如果有问题的挂载点不是由`umount()`或它的父挂载点引用的，那么将返回一个`EBUSY`错误，并且该挂载点不会被标记为过期或卸载。

如果挂载点在那时还没有被标记为过期，将会给出一个`EAGAIN`错误，它不会被卸载。

否则，如果它已经被标记并且没有被引用，卸载将照常进行。

同样，每当umount()以外的任何程序查看挂载点时，过期标志将被清除。

