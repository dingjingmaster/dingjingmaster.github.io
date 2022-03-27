# 文件系统格式转换


## 说明

- `fstransform` 命令可以将文件系统从一种类型转换为另一种类型而不丢失数据(即非破坏性的)。更重要的是它可以直接执行，而无需格式化或者复制数据。
- `tune2fs` 命令可以把 `ext2` 转为 `ext3` 或 `ext4`；也可以把 `ext3` 转为 `ext4`

## `fstransform`文件系统格式转换
### 需要注意的事

##### 1. Linux内核必须支持转换前和转换后的文件系统
##### 2. ext2升级到ext3、ext4无需 fstransform，要使用 tune2fs
##### 3. 源文件系统的设备至少有5%的可用空间
##### 4. 开始执行转换前卸载文件系统
##### 5. 源文件系统存储的数据越多，转换时间越长。实际取决于设备
##### 6. 虽然fstransform被证明是稳定的，但是也要备份好数据
##### 7. 无法转换Linux根文件系统格式
##### 8. 设备上的文件系统限制
- 1. 必须支持 SPARSE 文件(即:带有孔的文件)
- 2. 至少有两个系统调用"ioctl"(`FS_IOC_FIEMAP`和`FIBMAP`)中的一个

### 操作
```
fstransform /dev/xxx <目标系统>
```
> 亲测速度太慢，而且还有很高的失败率。建议如果是`ext`系列的文件系统使用`tune2fs`转换。

## `tune2fs` 转换`ext`系列文件系统格式

### `ext2` 转 `ext3`
```
tune2fs  -j   /dev/sda
```

### `ext2` 转 `ext4`
```
tune2fs -O extents,uninit_bg,dir_index,has_journal /dev/sda1
```

### `ext3` 转 `ext4`
```
tune2fs -O extents,uninit_bg,dir_index,has_journal /dev/sda1
```

