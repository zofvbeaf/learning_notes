## [libevent](https://libevent.org/)

### 简介

> 跨平台，底层支持/dev/poll, kqueue(2), event ports, POSIX select(2), Windows select(), poll(2), and epoll(4).
>
> `libevent` 本身不是多线程安全的，它只是简单的对`epoll`等进行了封装。
>
> 采用`libevent`的项目：[Chromium](http://www.chromium.org/Home)，[Memcached](http://www.memcached.org/)，[tmux](http://tmux.sourceforge.net/)，[Tor](https://www.torproject.org/) 等

 ### 接口

```c
struct event_base *base = event_base_new(); 
struct event *example_ev = event_new(base, STDIN_FILENO, 
                                     EV_READ | EV_PERSIST, example_cb, NULL);
event_add(example_ev, NULL); /* 没有超时 */
event_base_dispatch(base);   // 执行事件循环

int event_add(struct event *ev, const struct timeval *timeout);
int event_del(struct event *ev);
int event_base_loop(struct event_base *base, int loops);
void event_active(struct event *event, int res, short events);
void event_process_active(struct event_base *base);
void event_set(struct event *ev, int fd, short events,
			   void (*callback)(int, short, void *), void *arg);
int event_base_set(struct event_base *base, struct event *ev);
int event_priority_set(struct event *ev, int pri);
```

### 数据结构

#### `struct event_base`

```c
struct event_base {
    // 指向了全局变量 static const struct eventop *eventops[]中的一个如，epollops
	const struct eventop *evsel; 
    /** Pointer to backend-specific data. */
    void *evbase; 
    // 二级指针,其中的元素 activequeues[priority]是一个链表，
    // 链表的每个节点指向一个优先级为 priority 的就绪事件 event
    struct event_list *activequeues; 
    int nactivequeues; // activequeues 数组的长度
    struct event_list eventqueue; // 所有注册到event_base中的事件
    int event_count;  // event_base 中所有 added 事件总数
    int event_count_active; // event_base 中所有 active 事件总数

    // 以下两个用于管理时间
    struct timeval tv_cache; 
    struct timeval event_tv; 
    
    struct evsig_info sig; // 管理信号
    struct min_heap timeheap; // 管理超时
};

// event_list的声明
TAILQ_HEAD (event_list, event); 
#define TAILQ_HEAD(name, type)			\
struct name {					\
	struct type *tqh_first;			\
	struct type **tqh_last;			\
}

// eventop 实现了对底层epoll，select，poll等的封装
struct eventop {
	const char *name; // 后台方法名字，即epoll，select，poll等
	void *(*init)(struct event_base *);
	int (*add)(struct event_base *, evutil_socket_t fd, short old, short events, void *fdinfo);
	int (*del)(struct event_base *, evutil_socket_t fd, short old, short events, void *fdinfo);
	int (*dispatch)(struct event_base *, struct timeval *);
	void (*dealloc)(struct event_base *);
    int need_reinit; // fork 的时候是否需要重新 init
	enum event_method_feature features;
    size_t fdinfo_len;
};

const struct eventop epollops = {
	"epoll",
	epoll_init,
	epoll_nochangelist_add,
	epoll_nochangelist_del,
	epoll_dispatch,
	epoll_dealloc,
	1, /* need reinit */
	EV_FEATURE_ET|EV_FEATURE_O1, // select 和 poll 不支持此选项，O(1)地处理事件触发
	0
};
```

#### `struct event`

```c
/* 每次当有事件event转变为就绪状态时，
libevent就会把它移入到activequeues[priority]中，其中 priority 是 event 的优先级；
接着 libevent 会根据自己的调度策略选择就绪事件，调用其 ev_callback()函数执行事件处理；
并根据就绪的句柄和事件类型填充 ev_callback 函数的参数。*/
struct event {
	TAILQ_ENTRY(event) ev_active_next;
	TAILQ_ENTRY(event) ev_next;
    
    evutil_socket_t ev_fd; // int
    ev_uint8_t ev_pri;	/* 优先级，数字越小优先级越高  */
    // event关注的事件类型,例如 #define EV_READ		0x02
    // #define EV_ET       0x20
    short ev_events; 
    short ev_flags; //  表明其当前的状态，例如 #define EVLIST_ACTIVE 0x08 // event在激活链表中
	void (*ev_callback)(evutil_socket_t, short, void *arg);
};
```

#### `event_base_loop`

```c
int event_base_loop(struct event_base *base, int flags) {
    const struct eventop *evsel = base->evsel;
    int res, done, retval = 0;
    while (!done) {
        //...
        evsel->dispatch(base, tv_p);
        // 最终调用 (*ev->ev_callback)(ev->ev_fd, ev->ev_res, ev->ev_arg);
        event_process_active(base); 
        //...
    }
}

static int epoll_dispatch(struct event_base *base, struct timeval *tv) {
	struct epollop *epollop = base->evbase;
	struct epoll_event *events = epollop->events;
	int i, res;
	long timeout = -1;
	res = epoll_wait(epollop->epfd, events, epollop->nevents, timeout);
	for (i = 0; i < res; i++) {
		int what = events[i].events;
		short ev = 0; 
		
		if (what & (EPOLLHUP|EPOLLERR)) {
			ev = EV_READ | EV_WRITE;
		} else {
			if (what & EPOLLIN)
				ev |= EV_READ;
			if (what & EPOLLOUT)
				ev |= EV_WRITE;
		}

		if (!ev)
			continue;
		evmap_io_active(base, events[i].data.fd, ev | EV_ET); // 将事件放到对应 active 队列
	}
}
```

