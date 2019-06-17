块设备需要mknod吗?对于启动盘，你可能觉得，启动了就在那里了。可是如果我们要插进一块新的USB盘，还是要有这个操 作的。

mknod还是会创建在/dev路径下面，这一点和字符设备一样。/dev路径下面是devtmpfs文件系统。这是块设备遇到的第一个文 件系统。

要调用mount，将这个块设备文件挂载到一个文件夹下面 。这是块设备遇到的第二个文件系统，



mount_bdev主要做了两件大事情。第一，blkdev_get_by_path根据/dev/xxx这个名字，找到相应的设备并打开它;第二，sget 根据打开的设备文件，填充ext4文件系统的super_block，从而以此为基础，建立一整套咱们在文件系统那一章讲的体系。


# 总结

从这一节我们可以看出，块设备比字符设备复杂多了，涉及三个文件系统，工作过程我用一张图总结了一下，下面带你总结一 下。

1. 所有的块设备被一个map结构管理从dev_t到gendisk的映射;

2. 所有的block_device表示的设备或者分区都在bdev文件系统的inode列表中;

3. mknod创建出来的块设备文件在devtemfs文件系统里面，特殊inode里面有块设备号;

4. mount一个块设备上的文件系统，调用这个文件系统的mount接口;

5. 通过按照/dev/xxx在文件系统devtmpfs文件系统上搜索到特殊inode，得到块设备号;

6. 根据特殊inode里面的dev_t在bdev文件系统里面找到inode;

7. 根据bdev文件系统上的inode找到对应的block_device，根据dev_t在map中找到gendisk，将两者关联起来; 8. 找到block_device后打开设备，调用和block_device关联的gendisk里面的block_device_operations打开设备; 9. 创建被mount的文件系统的super_block。