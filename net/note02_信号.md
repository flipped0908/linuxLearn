
# 我们讲了信号的机制。在某些紧急情况下，我们需要给进程发送一个信号，紧急处理一些事情。


一旦有信号产生，我们就有下面这几种，用户进程对信号的处理方式。

1.执行默认操作。Linux对每种信号都规定了默认操作，例如，上面列表中的Term，就是终止进程的意思。

2.捕捉信号。我们可以为信号定义一个信号处理函数。

3.忽略信号。当我们不希望处理某些信号的时候，就可以忽略该信号，不做任何处理。



这也是很多人看信号处理的内核实现的时候，比较困惑的地方。例如，内核代码注释里面会说，系统调用signal是为了兼容过 去，
系统调用sigaction也是为了兼容过去，连参数都变成了struct compat_old_sigaction，所以说，
我们的库函数虽然调用的 是sigaction，到了系统调用层，调用的可不是系统调用sigaction，而是系统调用rt_sigaction。



在rt_sigaction里面，我们将用户态的struct sigaction结构，拷⻉为内核态的k_sigaction，然后调用do_sigaction。

总结时刻
这一节讲了如何通过API注册一个信号处理函数，整个过程如下图所示。
在用户程序里面，有两个函数可以调用，一个是signal，一个是sigaction，推荐使用sigaction。 用户程序调用的是Glibc里面的函数，
signal调用的是__sysv_signal，里面默认设置了一些参数，使得signal的功能受到了限 制，sigaction调用的是__sigaction，参数用户可以任意设定。 
无论是__sysv_signal还是__sigaction，调用的都是统一的一个系统调用rt_sigaction。 
在内核中，rt_sigaction调用的是do_sigaction设置信号处理函数。在每一个进程的task_struct里面，
都有一个sighand指向 struct sighand_struct，里面是一个数组，下标是信号，里面的内容是信号处理函数。