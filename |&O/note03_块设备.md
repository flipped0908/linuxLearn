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



# 直接I/O如何访问块设备?


# 缓存I/O如何访问块设备?

# 如何向块设备层提交请求?
既然无论是直接I/O，还是缓存I/O，最后都到了submit_bio里面，我们就来重点分析一下它。

块设备队列结构
如果再来看struct block_device结构和struct gendisk结构，我们会发现，每个块设备都有一个请求队列struct request_queue，用于处理上层发来的请求。


# 块设备的初始化
在blk_init_allocated_queue中，除了初始化make_request_fn函数，我们还要做一件很重要的事情，就是初始化I/O的电梯算 法。

电梯算法有很多种类型，定义为elevator_type。下面我来逐一说一下。 

struct elevator_type elevator_noop

Noop调度算法是最简单的IO调度算法，它将IO请求放入到一个FIFO队列中，然后逐个执行这些IO请求。 

struct elevator_type iosched_deadline

Deadline算法要保证每个IO请求在一定的时间内一定要被服务到，以此来避免某个请求饥饿。为了完成这个目标，算法中引入 了两类队列，
一类队列用来对请求按起始扇区序号进行排序，通过红黑树来组织，我们称为sort_list，按照此队列传输性能会 比较高;另一类队列对请求按它们的生成时间进行排序，由链表来组织，称为fifo_list，并且每一个请求都有一个期限值。


struct elevator_type iosched_cfq

又看到了熟悉的CFQ完全公平调度算法。所有的请求会在多个队列中排序。同一个进程的请求，总是在同一队列中处理。时 间片会分配到每个队列，通过轮询算法，我们保证了I/O带宽，以公平的方式，在不同队列之间进行共享。
elevator_init中会根据名称来指定电梯算法，如果没有选择，那就默认使用iosched_cfq。




# 请求提交与调度




# 请求的处理




# 总结时刻
这一节我们讲了如何将块设备I/O请求送达到外部设备。 对于块设备的I/O操作分为两种，一种是直接I/O，另一种是缓存I/O。无论是哪种I/O，最终都会调用submit_bio提交块设备I/O
请求。 对于每一种块设备，都有一个gendisk表示这个设备，它有一个请求队列，这个队列是一系列的request对象。每个request对象
里面包含多个BIO对象，指向page cache。所谓的写入块设备，I/O就是将page cache里面的数据写入硬盘。 对于请求队列来讲，还有两个函数，一个函数叫make_request_fn函数，用于将请求放入队列。submit_bio会调用
generic_make_request，然后调用这个函数。 另一个函数往往在设备驱动程序里实现，我们叫request_fn函数，它用于从队列里面取出请求来，写入外部设备。




















