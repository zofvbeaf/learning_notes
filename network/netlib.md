## 相关概念

### [同步和异步](https://blog.csdn.net/historyasamirror/article/details/5778378)

+ `IO`发生时涉及的对象和步骤：以`read`为例，一个是调用这个`IO`的进程或线程，另一个就是系统内核。

+ 当一个`read`操作发生时，它会经历两个阶段：
  + 等待数据准备
  + 将数据从内核拷贝到进程中
    + **同步、异步的关键**：如果此步骤由内核完成则为*异步*；如果为用户完成，如调用`recvfrom`，则为*同步*）
  
+ [异步IO(`aio_read`)](https://www.ibm.com/developerworks/cn/linux/l-async/index.html)

  + 应用进程发起异步读取操作后，内核会立即返回，不会阻塞应用进程。等内核的数据准备完毕、内核负责将数据拷贝到用户内存后，通知应用进程操作完毕。异步模式用于实现`Proactor`模式。

    ![img](http://hi.csdn.net/attachment/201007/31/0_1280551287S777.gif)

    ```c
    #include <aio.h>
    // aio_read 示例
    {
      int fd, ret;
      struct aiocb my_aiocb;
     
      fd = open( "file.txt", O_RDONLY );
      if (fd < 0) perror("open");
     
      /* Zero out the aiocb structure (recommended) */
      bzero( (char *)&my_aiocb, sizeof(struct aiocb) );
     
      /* Allocate a data buffer for the aiocb request */
      my_aiocb.aio_buf = malloc(BUFSIZE+1);
      if (!my_aiocb.aio_buf) perror("malloc");
     
      /* Initialize the necessary fields in the aiocb */
      my_aiocb.aio_fildes = fd;
      my_aiocb.aio_nbytes = BUFSIZE;
      my_aiocb.aio_offset = 0;
     
      ret = aio_read( &my_aiocb );
      if (ret < 0) perror("aio_read");
     
      while ( aio_error( &my_aiocb ) == EINPROGRESS ) ;
     
      if ((ret = aio_return( &my_iocb )) > 0) {
        /* got ret bytes on the read */
      } else {
        /* read failed, consult errno */
      }
    }
    ```

### 阻塞和非阻塞

+ **阻塞**：`IO`执行的两个阶段都是`block`的

  ![](http://ww1.sinaimg.cn/large/77451733gy1g4fs7ph03vj20fc097q3c.gif)

+ **非阻塞**：进程需要不断地询问内核数据是否准备好，IO执行的第二阶段是block的，属于同步模式

  ![img](http://hi.csdn.net/attachment/201007/31/0_128055089469yL.gif)

### IO复用

+ 单进程可以处理多个IO连接，比如套接字、管道等。IO复用一般和非阻塞一起使用，监听的文件描述符一般设置为非阻塞模式，这是为了避免错误使用阻塞`fd`，导致阻塞，IO复用只有一个阻塞点，比如说`epoll`会阻塞在`epoll_wait`。当有事件发生，通知用户进程处理事件。IO复用中`read`的第二阶段仍需要用户进程`block`，故属于同步模式。

  ![img](http://hi.csdn.net/attachment/201007/31/0_1280551028YEeQ.gif)

### Reactor和Proactor

> 一般地,I/O多路复用机制都依赖于一个事件多路分离器(Event Demultiplexer)。分离器对象可将来自事件源的I/O事件分离出来，并分发到对应的read/write事件处理器(Event Handler)。开发人员预先注册需要处理的事件及其事件处理器（或回调函数）；事件分离器负责将请求事件传递给事件处理器。两个与事件分离器有关的模式是Reactor和Proactor。
>
> Reactor模式采用同步IO，而Proactor采用异步IO。

#### [Reactor模式](https://www.jianshu.com/p/96c0b04941e2)

+ 事件分离器负责等待文件描述符或`socket`为读写操作准备就绪，然后将就绪事件传递给对应的工作线程，最后由工作线程负责完成实际的读写工作。

  ![img](https://upload-images.jianshu.io/upload_images/2100026-a2e15f5fd17e96d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/884)

#### Proactor模式

+ 工作线程--或者兼任事件分离器，只负责发起异步读写操作。IO操作本身由内核来完成。传递内核的参数需要包括用户定义的数据缓冲区地址和数据大小，内核才能从中得到写出操作所需数据，或写入从`socket`读到的数据。事件分离器捕获IO操作完成事件，然后将事件传递给对应工作线程。

  ![img](https://upload-images.jianshu.io/upload_images/2100026-273b3206fddb27fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/920)

+ `boost asio`实现的`proactor`，并不是真正意义上的异步IO，底层是用`epoll`实现的，用`io_servie`模拟异步IO的。

### 单例模式

+ 由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象，单例模式无疑可以提高系统的性能。

### 工厂模式

## UML类图

### 类的关系

#### 泛化（generalization）

> 继承非抽象类，箭头指向父类

![泛化](https://design-patterns.readthedocs.io/zh_CN/latest/_images/uml_generalization.jpg)



#### 实现（realize）

> 继承抽象类，箭头指向接口

![实现](https://design-patterns.readthedocs.io/zh_CN/latest/_images/uml_realize.jpg)

#### 聚合（aggregation）

> 整体和部分不是强依赖的，B不存在，A仍可存在

![聚合](https://design-patterns.readthedocs.io/zh_CN/latest/_images/uml_aggregation.jpg)

#### 组合（composition）

> 强依赖的，B不存在，A也不存在

![组合](https://design-patterns.readthedocs.io/zh_CN/latest/_images/uml_composition.jpg)

#### 关联（association）

> 是一种拥有的关系,它使一个类知道另一个类的属性和方法。箭头指向被拥有者。
>
> 分为双向关联和单向关联。
>
> 老师与学生是双向关联，老师有多名学生，学生也可能有多名老师。但学生与某课程间的关系为单向关联，一名学生可能要上多门课程，课程是个抽象的东西他不拥有学生。
>
> | Indicator | Meaning         |
> | :-------- | :-------------- |
> | 0..1      | Zero or one     |
> | 1         | One only        |
> | 0..*      | Zero or more    |
> | *         | Zero or more    |
> | 1..*      | One or more     |
> | 3         | Three only      |
> | 0..5      | Zero to Five    |
> | 5..15     | Five to Fifteen |

![关联](http://hi.csdn.net/attachment/201104/22/0_1303436801W1kf.gif)

#### 依赖（dependency）

> 描述一个对象在运行期间会用到另一个对象的关系

![依赖](https://design-patterns.readthedocs.io/zh_CN/latest/_images/uml_dependency.jpg)

### 类的成员

> 画图顺序：类名，类成员变量，类的方法

+ `public`：`+`

+ `private`：`-`

+ `protected`：`#`

+ `static`：underlined

## 基类架构

![](https://ws1.sinaimg.cn/large/77451733gy1g56jqyviucj23f01s5n4c.jpg)

## 测试用例

> 实现`echo`服务器和客户端，`server`对每个`client`，每当收到其发来的序号为`1000`的消息，就写一条日志。

