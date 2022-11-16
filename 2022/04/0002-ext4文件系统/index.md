# ext4 文件系统


# ext4 文件系统

ext4文件系统被分成一系列块组。为了减少由于碎片造成的性能困难，块分配器非常努力地将每个文件的块保持在同一组中，从而减少寻道时间。一个块组的大小是用`sb.s_blocks_per_group` 块指定的，尽管它也可以计算为`8 * block_size_in_bytes`。默认块大小为`4KiB`，每个组将包含`32,768`($2^{16} / 2$)个块，长度为128MiB。块组个数是设备大小除以块组大小。

ext4中的所有字段都按小端顺序写入磁盘。然而，jbd2(日志)中的所有字段都是按大端顺序写入磁盘的。

https://www.kernel.org/doc/html/latest/filesystems/ext4/overview.html


