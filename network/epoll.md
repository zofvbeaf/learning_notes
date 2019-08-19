## epoll

> 内部监听的事件以`epitem`组织为一个红黑树
>
> 注册和事件添加到epoll的rdlist都是通过调用f_op->poll
>
> 注册时是通过将epoll添加到文件的wait_queue_head中
>
> rdlist上的epitem通常是由被监听文件自己挂上去的

+ EPOLLRDHUP：对端关闭
+ 对端正常关闭（程序里close()，shell下kill或ctr+c），触发EPOLLIN和EPOLLRDHUP，但是不触发EPOLLERR和EPOLLHUP。EPOLLHUP 是对端 reset 了连接

![](http://ww1.sinaimg.cn/large/77451733gy1g4tx3utmjij21fe0sk75f.jpg)

### data struct

+ `fs/eventpoll.c`
+ `epoll_create (SYSCALL_DEFINE1(epoll_create) -> do_epoll_create`

```c
// 有 strc
struct eventpoll {
  	spinlock_t lock; // 保护对eventpoll的访问
  	struct mutex mtx; // ctl 和 事件收集时用到

    /* Wait queue used by sys_epoll_wait() */
    wait_queue_head_t     wq;
    /* Wait queue used by file->poll() */
    wait_queue_head_t       poll_wait;
	
    struct file *file;
    
    struct list_head rdllist; // 所有 ready 的 fd
	struct rb_root_cached rbr; // 所有 add 进来的 fd -> epitem
    struct epitem *ovflist; // 单项链表存储所有 epitem
};

struct epoll_event {
    __poll_t events;
    __u64 data;
} EPOLL_PACKED;

struct epoll_filefd {
    struct file *file;
    int fd;
} __packed;

struct eppoll_entry {
    struct list_head llink; // 指向 epitem 中的 pwqlist，从而将 epitem 连成链表
    struct epitem *base; // 指向所属的 epitem
    wait_queue_entry_t wait; // 要加入被监听文件的wait_queue的回调函数
    wait_queue_head_t *whead;
};

struct epitem {
	struct eventpoll *ep; // 指向所属的 eventpoll
   	/* The file descriptor information this item refers to */
	struct epoll_filefd ffd; // epitem 对应的 struct file 和 fd
    struct list_head fllink; // epitem 中的 struct file 通过这个组织成一个链表
    struct list_head pwqlist; // poll wait queues，指向 epoll_entry 中的 llink
    struct epoll_event event;
};
```

+ 相关的数据结构
```c++
struct wait_queue_head {
    spinlock_t              lock;
    struct list_head        head;
};
typedef struct wait_queue_head wait_queue_head_t;

struct mutex {
    atomic_long_t           owner;
    spinlock_t              wait_lock;
    struct optimistic_spin_queue osq; // 优化 mutex 的性能
    struct list_head        wait_list;
};

/*
 * Leftmost-cached rbtrees.
 *
 * We do not cache the rightmost node based on footprint
 * size vs number of potential users that could benefit
 * from O(1) rb_last(). Just not worth it, users that want
 * this feature can always implement the logic explicitly.
 * Furthermore, users that want to cache both pointers may
 * find it a bit asymmetric, but that's ok.
 */
struct rb_root_cached {
    struct rb_root rb_root;
    struct rb_node *rb_leftmost;
};

struct file {
  struct path                     f_path;
  struct inode                    *f_inode;          /* cached value */
  const struct file_operations    *f_op;
  spinlock_t                      f_lock; 
  // 对于epollfd指向 struct event_poll
  void                      *private_data; 
#ifdef CONFIG_EPOLL
    /* Used by fs/eventpoll.c to link all the hooks to this file */
    struct list_head        f_ep_links; // 所有监听此文件的 epitem 的链表
    struct list_head        f_tfile_llink; 
#endif /* #ifdef CONFIG_EPOLL */
  // ....
};

struct callback_head {
    struct callback_head *next;
    void (*func)(struct callback_head *head);
} __attribute__((aligned(sizeof(void *))));
#define rcu_head callback_head
```

### epoll_create

+ `glibc`的`epoll_create` -> 系统调用的`epoll_create`  -> `do_epoll_create`
  + `ep_alloc(&ep)` 创建一个 `struct eventpoll` 即 `ep`，并初始化
  + `fd = get_unused_fd_flags` 获取一个可用的`fd`
  + `file = anon_inode_getfile("[eventpoll]", &eventpoll_fops, ep,O_RDWR | (flags & O_CLOEXEC));`
    + 创建一个名为`[eventpoll]`的匿名文件，文件的`fop`设置为`eventpoll_fops`（`staic`全局变量）
      + `file->private_data = ep`
      + `fd_install(fd, file);`  使 `task_struct->files->fdt->fd[fd] = file`
      + `eventpoll_fops->poll = ep_eventpoll_poll`
        + [poll函数](https://blog.csdn.net/qq69696698/article/details/7473517)用于驱动提供给应用程序探测设备文件是否可读或可写。当该函数指针为空时表示设备可对文件非阻塞的读写。调用poll函数的进程会`sleep`直到可读或可写。

### epoll_ctl

> 根据参数分别调用`ep_insert`，`ep_remove`，`ep_modify`。

+ `ADD`：
  +  `epoll_ctl`在`ADD`时会调用对监听`fd`的`f_op->poll`，里面将调用`poll_wait`创建一个`wait_queue_entry`，内含一个回调函数（在`poll_table`中 ），将此`wait_queue_entry`挂到被监听`fd`的`wait_queue_head`上，当设备对此`fd`进行读写时会唤醒被监听`fd`的`wait_queue_head`上监听对应事件的进行，调用注册进来的回调函数。
  +  `epoll`使用红黑树管理它监听的所有文件，它为每个监听的文件创建一个`epitem`，加入到`eventpoll`的`rbr`字段（红黑树root）中。
  +  如果要监听的是一个非`epoll`文件，内核会调用`epfd`的`f_op->poll`回调，内核创建一个`epoll_entry`，通过调用该回调函数，将`epoll_entry`加入到文件内部的`wait_queue_head`中，以便文件有事件到达时通过该`epoll_entry`，将对应的`epitem`加入到`eventpoll`的`rdlist`中。
  +  如果要监听的是一个`epoll`文件，`epfd2`对应的`eventpoll`结构体中有一个`wait_queue_head——poll_wait`。因此，内核同样会创建一个`epoll_entry`，然后将该`epoll_entry`加入到`poll_wait`中，以便`epfd2通`知`epfd1`。

### epoll_wait

+ 如果`eventpoll`的`rdlist`中没有`epollitem`，则说明监听的文件没有一个事件到达，这时需要将当前文件陷入阻塞（睡在`eventpoll->wq`上），等待有任意事件到达。
+ 如果有事件到达，则将`rdlist`中所有的`epollitem`一口气取下来，放到一个临时链表中（避免争用`rdlist`）。对于临时链表中的每一项`epollitem`：
  + 将`epollitem`从链表中取下
  + 调用`ep_item_poll`
    + 如果`epitem`对应的是一个非`epoll`文件，调用它的`f_op->poll`回调，获取该文件的事件，与`epitem`记录的要等待的事件取与。若取与得到的值不为0，
      + 说明确实有要等待的事件发生，首先将该事件拷贝到用户缓冲区
      + 然后，如果是`EPOLLONESHOT`，则在拷贝完事件后去掉对`EPOLLIN/EPOLLOUT`等的监听，只保留`EPOLLWAKEUP | EPOLLONESHOT | EPOLLET | EPOLLEXCLUSIVE`四个标志位的值。
      + 然后，如果是水平触发模式，则将该`epollitem`再放到`eventpoll`的尾部。
    +  如果`epitem`对应的是一个`epoll`文件（称为`epfd2`），则以`epfd2`为参数，递归进入第二项判断`rdlist`。
  + 如果从临时链表中取出了用户要求的最大事件数，则停止循环
+ 现在，将临时链表中剩下的没有处理完的那些事件重新加到`rdlist`中。
+ 如果`rdlist`中有还没有处理完的数据，唤醒睡在`eventpoll->wq`上的进程（处于`epoll_wait`阻塞的进程），并通知`eventpoll->poll_wait`上的进程（比如`epfd1`监听`epfd2`，那么`epfd2`会通过自己的`poll_wait`通知`epfd1`，这会导致`epfd1`的`rdlist`中出现一项关于`epfd2`的`epollitem`）。
+ 如果发现没有拿到任何事件（到达的事件和想要的事件不同，或者被其他进程提前一步拿走了），且还没有超过`epoll_wait`限制的时间，则跳转到第一步重新开始。
+ 通过上面的流程描述，可以知道，水平触发模式的文件，在缓冲区有数据时，会一直待在`rdlist`中，直到它的缓冲区数据为空。
+ 需要注意的是，`rdlist`上的`epitem`通常是由被监听文件自己挂上去的。比如，当监听一个管道时，管道文件自己有一个`wait_queue_head`，当我们将管道文件通过`EPOLL_ADD`加入到`epoll`中时，会调用一次管道文件的`poll`回调。poll回调通过调用`poll_wait`函数，将我们的`epoll`注册到它自己的`wait_queue_head`中。然后，当管道文件发现自己缓冲区可读可写时，它会通过自己的`wait_queue_head`通知`epoll`，将`epitem`挂在`epoll`的`rdlist`里面。这样，当`epoll_wait`被调用时，就会发现自己的`rdlist`里面有数据，然后，它会遍历`rdlist`中的每一项，对每一项调用`poll`回调，这样，管道的`poll`回调又一次被调用了。不过，这一次管道并不会将`epoll`注册到自己的`wait_queue_head`中去，它仅仅判断自己的缓冲区的可读可写事件，返回对于的`POLLIN/POLLOUT`，然后，`epoll_wait`根据返回的事件，与要监听的事件做与操作，来决定是否拷贝该事件。

### [epoll惊群](https://pureage.info/2015/12/22/thundering-herd.html)

+ 父进程创建socket，bind、listen后，通过fork创建多个子进程，每个子进程继承了父进程的socket，调用accpet开始监听等待网络连接。这个时候有多个进程同时等待网络的连接事件，当这个事件发生时，这些进程被同时唤醒，就是“惊群”。这样会导致什么问题呢？我们知道进程被唤醒，需要进行内核重新调度，这样每个进程同时去响应这一个事件，而最终只有一个进程能处理事件成功，其他的进程在处理该事件失败后重新休眠或其他。
+ 在Linux2.6版本以后，内核内核已经解决了accept()函数的“惊群”问题，大概的处理方式就是，当内核接收到一个客户连接后，**只会唤醒等待队列上的第一个进程或线程**。所以，如果服务器采用accept阻塞调用方式，在最新的Linux系统上，已经没有“惊群”的问题了。
+ 但是，对于实际工程中常见的服务器程序，大都使用select、poll或epoll机制，此时，服务器不是阻塞在accept，而是阻塞在select、poll或epoll_wait，这种情况下的“惊群”仍然需要考虑。
+ 有时候fork了很多个进程，看上去是只有部分进程唤醒了，而事实上，其余进程没有被唤醒的原因是你的某个进程已经处理完这个 accept，内核队列上已经没有这个事件，无需唤醒其他进程。你可以在 epoll 获知这个 accept 事件的时候，不要立即去处理，而是 sleep 下，这样所有的进程都会被唤起。
+ epoll存在惊群的场景如下：在worker保持工作的状态下，都会被唤醒，例如在epoll_wait后调用sleep一次。
+ accept 确实应该只能被一个进程调用成功，内核很清楚这一点。但 epoll 不一样，他监听的文件描述符，除了可能后续被 accept 调用外，还有可能是其他网络 IO 事件的，而其他 IO 事件是否只能由一个进程处理，是不一定的，内核不能保证这一点，这是一个由用户决定的事情，例如可能一个文件会由多个进程来读写。所以，对 epoll 的惊群，内核则不予处理。
+ Nginx就是同一时刻只允许一个nginx worker在自己的epoll中处理监听句柄。它的负载均衡也很简单，当达到最大connection的7/8时，本worker不会去试图拿accept锁，也不会去处理新连接，这样其他nginx worker进程就更有机会去处理监听句柄，建立新连接了。而且，由于timeout的设定，使得没有拿到锁的worker进程，去拿锁的频繁更高。
+ accept 不会有惊群，epoll_wait 才会。
+ Nginx 的 accept_mutex,并不是解决 accept 惊群问题，而是解决 epoll_wait 惊群问题。
+ 说Nginx 解决了 epoll_wait 惊群问题，也是不对的，它只是控制是否将监听套接字加入到epoll 中。监听套接字只在一个子进程的 epoll 中，当新的连接来到时，其他子进程当然不会惊醒了。

## poll

```c
int ppoll(struct pollfd *fds, nfds_t nfds,
			const struct timespec *tmo_p, const sigset_t *sigmask);
```

+ `poll`的实现和`select`类似，也是只有一个系统调用，不常驻内核。原来`select`要传位图，现在`poll`变成了要监听的那些文件的`fd`和事件掩码的数组（没有`select`对`FDSIZE`的限制，由用户保证），不过处理思路与`selelct`基本相同。

## [select](http://janfan.cn/chinese/2015/01/05/select-poll-impl-inside-the-kernel.html)

```c++
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
// 默认 #define __FD_SETSIZE    1024
void FD_CLR(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_SET(int fd, fd_set *set);
void FD_ZERO(fd_set *set);
```

+ 调用过程：`select -> kern_select -> core_sys_select`
  + 对于每个`fd`，如果在读/写/异常`fd_set`中的任意一个`fd_set`中置位了
    + 调用`fd`对于`file`的`f_op->poll`回调
      + 如果`poll_table`中有回调函数，它将负责创建一个`wait_queue_entry`，并将该`entry`挂在`file`提供的`wait_queue_head`中。然后，`poll`回调函数还将返回文件的状态（`POLLIN/POLLOUT/…`）
      + 如果`poll_table`没有回调函数，则`poll`回调仅仅返回文件的状态
      + 根据`poll`回调返回的文件状态，判断返回状态是否为想要监听的状态。比如返回了`POLLIN`，且`fd`恰好在读的`fd_set`中，则在返回给用户的读`fd_set`中，标记该位，**然后将`poll_table`的回调置为`NULL`**。这一步很重要，因为它会导致后续对fd的`f_op->poll`回调不再挂任何`wait_queue_entry`到剩下的`fd`的`wait_queue_head`中。
  + 当对于所有的`fd`都判断完毕后
    + 如果得到了想要监听的事件，那么就取下那些之前遍历时挂上去的`wait_queue_entry`，然后将对应的事件拷贝给用户。
    + 如果超时了，那么操作和上面一样
    + 否则，说明没有任何想要的事件达到，而且我们已经将`wait_queue_entry`挂在了每个想要监听的文件的`wait_queue_head`上。现在，根据`select`传递进来的超时时间，陷入一定时间的睡眠，等待被超时唤醒，或者被监听文件唤醒，然后重复第一步。
+ `struct timeval *timeout`内的内容会经历从用户态拷贝到内核再拷贝回来的过程。
+ `select`只有一个系统调用，而不像`epoll`拥有三个系统调用。因此，`select`对事件的监听没有像`epoll`那样，做到常驻内核。
+ 在使用`select`时，需要传递三个`fd_set`，`fd_set`实际上是关于`fd`的位图，所有需要监听的`fd`对于的位被置1。三个`fd_set`分别对应读/写/异常事件的监听

## select 和 epoll 对比

+ `select`比`epoll`慢的主要原因：
  + 对`fd_set`的拷贝开销：三个`fd_set`，不管结果成功与否，都要进行拷贝。
  + 没有常驻内核：每次调用`select`都需要重新挂`wait_queue_entry`，离开时还需要给取下来。
+ 另外，由于`select`是位图，所以允许监听的最大`fd`数量是有限制的，而`epoll`使用红黑树，它可以监听更多的`fd`。

## problems

+ `spinlock_t`在`4.2`之后内部实现改为了[`qspinlock`](http://tinylab.org/linux-kernel-4.2-highlights/)
+ `struct mutex`的`trylock`调用了`atomic_long_try_cmpxchg_acquire`
        + `struct optimistic_spin_queue osq;`的作用
+ `rcu`机制

