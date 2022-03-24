# Linux 命令 —— dd


## 说明

dd：用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。

## 块大小可以使用的计量表
|单元大小|表示|
|---|---|
|字节(1B)|c|
|字节(2B)|w|
|块(512B)|b|
|千字节(1024B)|k|
|兆字节(1024KB)|M|
|吉字节(1024MB)|G|

## 参数解释
|参数|说明|
|---|---|
|if=<文件名>|输入文件名，默认为stdin标准输入，用来指定读取的源文件|
|of=<文件名>|输出文件名，默认为stdout标准输出，用来指定输出的目的文件|
|ibs=<字节>|一次读入bytes个字节，即指定一个块大小为bytes个字节|
|obs=<字节>|一次输出bytes个字节，即指定一个块大小为bytes个字节|
|bs=<字节> |同事设置读入/读出的块大小为 bytes 个字节|
|cbs=<字节>|一次转换bytes个字节，即指定转换缓存区大小|
|skip=<字节>|从输入文件开头跳过blocks个块后再开始复制|
|seek=<字节>|从输出文件开头跳过blocks个块后再开始复制|
|count=<字节>|仅拷贝blocks个块，块大小等于ibs指定的字节数|
|conv=<转换参数>|用指定的参数转换文件。<br/><br/>ascii:转换ebcdic为ascii<br/>ebcdic:转换ascii为ebcdic<br/>ibm:转换ascii为alternate ebcdic<br/>block:把每一行转换为长度为cbs，不足部分用空格填充<br/>unblock:使没一行的长度都为cbs，不足部分用空格填充<br/>lcase:把大写字符转换为小写字符<br/>ucase:把小写字符转换为大写字符<br/>swab:交换输入的每对字节<br/>noerror:出错时不停止<br/>notrunc:不截断输出文件<br/>sync:将每个输入块填充到ibs个字节，不足部分也能够(NUL)字符补齐|

## dd应用实例
### 将本地 /dev/hdb 盘备份到 /dev/hdd
```
dd if=/dev/hdb of=/dev/hdd
```
### 备份/dev/hdb全盘数据，并利用gzip工具进行压缩，保存到指定路径
```
dd if=/dev/hdb | gzip > /root/image.gz
```

### 将压缩的备份文件恢复到指定盘
```
gzip -dc /root/image.gz | dd of=/dev/hdb
```

### 备份与恢复MBR
备份磁盘开始的512个字节大小的MBR信息到指定文件：
```
dd if=/dev/hda of=/root/image count=1 bs=512
```
count=1指仅拷贝一个块；bs=512指块大小为512个字节

### 增加swap分区文件大小
第一步：创建一个大小为256M的文件：
```
dd if=/dev/zero of=/swapfile bs=1024 count=262144
```
第二步：把这个文件变成swap文件：
```
mkswap /swapfile
```
第三步：启用这个swap文件：
```
swapon /swapfile
```
第四步：编辑/etc/fstab文件，使在每次开机时自动加载swap文件：
```
/swapfile    swap    swap    default   0 0
```

### 销毁磁盘数据
```
dd if=/dev/urandom of=/dev/hda1
```
> 注意：利用随机的数据填充硬盘，在某些必要的场合可以用来销毁数据

### 测试硬盘的读写速度
```
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
dd if=/root/1Gb.file bs=64k | dd of=/dev/null
```
通过以上两个命令输出的命令执行时间，可以计算出硬盘的读、写速度

### 确定硬盘的最佳块大小
```
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
dd if=/dev/zero bs=2048 count=500000 of=/root/1Gb.file
dd if=/dev/zero bs=4096 count=250000 of=/root/1Gb.file
dd if=/dev/zero bs=8192 count=125000 of=/root/1Gb.file
```
通过比较以上命令输出中所显示的命令执行时间，即可确定系统最佳的块大小

### 修复硬盘
```
dd if=/dev/sda of=/dev/sda 
# 或
dd if=/dev/hda of=/dev/hda
```
当硬盘较长时间(一年以上)放置不使用后，磁盘上会产生magnetic flux point，当磁头读到这些区域时会遇到困难，并可能导致I/O错误。当这种情况影响到硬盘的第一个扇区时，可能导致硬盘报废。上边的命令有可能使这些数 据起死回生。并且这个过程是安全、高效的

### 利用netcat远程备份
```
dd if=/dev/hda bs=16065b | netcat < targethost-IP > 1234
```
在源主机上执行此命令备份/dev/hda
```
netcat -l -p 1234 | dd of=/dev/hdc bs=16065b
```
在目的主机上执行此命令来接收数据并写入/dev/hdc
```
netcat -l -p 1234 | bzip2 > partition.img
netcat -l -p 1234 | gzip > partition.img
```
以上两条指令是目的主机指令的变化分别采用bzip2、gzip对数据进行压缩，并将备份文件保存在当前目录

## 另附

### /dev/null 和/dev/zero的区别

/dev/null，外号叫无底洞，你可以向它输出任何数据，它通吃，并且不会撑着。它是空设备，也称为位桶（bit bucket）。任何写入它的输出都会被抛弃。如果不想让消息以标准输出显示或写入文件，那么可以将消息重定向到位桶

/dev/zero，是一个输入设备，你可你用它来初始化文件。该设备无穷尽地提供0，可以使用任何你需要的数目——设备提供的要多的多。他可以用于向设备或文件写入字符串0

### 使用/dev/null
把/dev/null看作"黑洞"， 它等价于一个只写文件，所有写入它的内容都会永远丢失.，而尝试从它那儿读取内容则什么也读不到。然而， /dev/null对命令行和脚本都非常的有用

#### 禁止标准输出
```
cat $filename >/dev/null
```
#### 禁止标准错误
```
#rm $badname 2>/dev/null
```
#### 禁止标准输出和标准错误的输出
```
cat $filename 2>/dev/null >/dev/null
cat $filename &>/dev/null
```

#### 自动清空日志文件的内容
```
cat /dev/null > /var/log/messages
# : > /var/log/messages   有同样的效果， 但不会产生新的进程.（因为:是内建的）
```

#### 隐藏cookie而不再使用
```
if [ -f ~/.netscape/cookies ]  # 如果存在则删除.
then
rm -f ~/.netscape/cookies
fi
ln -s /dev/null ~/.netscape/cookies
```
现在所有的cookies都会丢入黑洞而不会保存在磁盘上了

#### 使用/dev/zero
像/dev/null一样， /dev/zero也是一个伪文件， 但它实际上产生连续不断的null的流（二进制的零流，而不是ASCII型的）。 写入它的输出会丢失不见， 而从/dev/zero读出一连串的null也比较困难， 虽然这也能通过od或一个十六进制编辑器来做到。 /dev/zero主要的用处是用来创建一个指定长度用于初始化的空文件，就像临时交换文件。



