## 如何创建一个进程  


fork是一个系统调用，根据系统调用的流程，流程的最后会在sys_call_table中找到相应的系统调 用sys_fork。   

sys_fork会调用_do_fork。  


####  fork的第一件大事:复制结构 

dup_task_struct主要做了下面几件事情:
调用alloc_task_struct_node分配一个task_struct结构;
调用alloc_thread_stack_node来创建内核栈，这里面调用__vmalloc_node_range分配一个连续的 THREAD_SIZE的内存空间，赋值给task_struct的void *stack成员变量;
调用arch_dup_task_struct(struct task_struct *dst, struct task_struct *src)，将task_struct进行复制，其实 就是调用memcpy;
调用setup_thread_stack设置thread_info。  


#### retval = copy_creds(p, clone_flags);  

#### 轮到权限相关了，copy_creds主要做了下面几件事情:
调用prepare_creds，准备一个新的struct cred *new。如何准备呢?其实还是从内存中分配一个新的 struct cred结构，然后调用memcpy复制一份父进程的cred;
接着p->cred = p->real_cred = get_cred(new)，将新进程的“我能操作谁”和“谁能操作我”两个权限都 指向新的cred。   


#### 接下来，copy_process重新设置进程运行的统计量。 


#### 接下来，copy_process开始设置调度相关的变量。 

sched_fork主要做了下面几件事情:


####  接下来，copy_process开始初始化与文件和文件系统相关的变量。


####  接下来，copy_process开始初始化与信号相关的变量。

####  接下来，copy_process开始复制进程内存空间。 

####  接下来，copy_process开始分配pid，设置tid，group_leader，并且建立进程之间的亲缘关系 


####  好了，copy_process要结束了，上面图中的组件也初始化的差不多了。



##  fork的第二件大事:唤醒新进程 


#### _do_fork做的第二件大事是wake_up_new_task 

####  首先，我们需要将进程的状态设置为TASK_RUNNING。 

activate_task函数中会调用enqueue_task。  

如果是CFS的调度类，则执行相应的enqueue_task_fair。  


在enqueue_task_fair中取出的队列就是cfs_rq，然后调用enqueue_entity。  
在enqueue_entity函数里面，会调用update_curr，更新运行的统计量，然后调用__enqueue_entity，将 sched_entity加入到红黑树里面，然后将se->on_rq = 1设置在队列上。  

check_preempt_curr，看是否能够抢占当前进程。  

已经将当前父进程的TIF_NEED_RESCHED设置了，则直接返回。
check_preempt_wakeup还是会调用update_curr更新一次统计量

如果新创建的进程应该抢占父进程，在什么时间抢占呢?别忘了fork是一个系统调用，从系统调用返回的时 候，是抢占的一个好时机，如果父进程判断自己已经被设置为TIF_NEED_RESCHED，就让子进程先跑，抢占 自己。   













































