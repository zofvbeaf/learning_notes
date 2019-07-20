## 1. Course

+ [MIT 6.828](<https://pdos.csail.mit.edu/6.828/2018/schedule.html>)
+ [ustc os](<http://staff.ustc.edu.cn/~xlanchen/OperatingSystemConcepts2017Spring/OperatingSystem2017Spring.htm>)
+ [ustc linux操作系统分析](<http://staff.ustc.edu.cn/~xlanchen/ULK2014Fall/ULK2014Fall.html>)
+ [stanford cs240](http://www.scs.stanford.edu/17sp-cs240/syllabus/)
+ [cmu 15410](https://www.cs.cmu.edu/~410/)

## 2. 系统启动

> 现代计算机使用UEFI，可以一次加载超过512B的boot sector。
>
> 以下为JOS启动流程。

+ 计算机通电后先读取`BIOS`，加载到`960KB~1MB`地址出。
+ BIOS进行硬件自检，同时打印硬件信息在屏幕上。有问题蜂鸣器会响。
+ 之后计算机读取BIOS设置好的优先级最高的外部存储设备。读取第一个扇区（512字节，即主引导记录`MBR`）到`0x7c00`处。如果这个扇区最后两个字节是`0x55`和`0XAA`表明可以启动，否则不能。
+ 主引导记录即`boot.S`，其中的主要流程包括：
  + 关中断，开`A20`线（兼容性问题，寻找范围更大），加载`段表`(`lgdt gdtdesc`)（包含操作系统内核的段信息），寻址方式变为`segment:offset`
  + 设置`CR0`寄存器的`CR0_PE_ON`为1：从实模式（16bit寻址）切换到保护模式（32bit寻址）
    + `实模式`下寻址方式：`物理地址 = 段基址<<4 + 段内偏移`，早期寄存器是16位，地址线是20位，只能访问1MB
      +  例如：`%cs = xff00`，`%ax = 0x0110`，则物理地址为：`0xff00<<4 + 0x0110 = 0xff110`
    + 保护模式下寻址方式：`segment:offset`，segment找到该段的基址base，（检查flags可访问的话）之后直接与offset相加得到线性地址，如果没有分页则线性地址等于物理地址。
      + 程序员看到的是虚拟地址，但指令中出现的是逻辑地址即`segment:offset`
  + 跳转到保护模式代码段，该段主要执行：
    + 设置各寄存器的值，包括cs指令地址，ds数据段地址，ss栈指针等，jos设置其指向`0x7c00`
    + `call bootmain`：即`boot/main.c`中的代码
+ `bootmain`的主要工作：
  + 将硬盘上的`kernel`加载到内存里的`0x10000`处，其中`.text`段在`0x100000`处。并执行`ELFHDR->e_entry`(`0x10000C`处，`boot/entry.S`)。内核映像为ELF格式。
    + entry处的代码主要：这是CR3寄存器，开启分页，设置内核堆栈的起始地址（`0xf0110000`），之后`call  i386_init`

## 3. [内存管理](https://edsionte.com/techblog/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86)

### 3.1、分页

+ JOS采用二级页表，CR3指向`page dircectory`，虚拟地址高10位查找`page dircectory`中的`Page Table Entry（PTE）`，每个`PTE`存放了一个页地址的高20位，PTE的低12位用作标志位，物理地址的低12位是从原虚拟地址直接拷贝过来的。

+ 现代`X86`是flat模式，即只有一个段，即段基址为0，所以虚拟地址=线性地址。

  + 在平坦模式下，段机制在两个地方会被用上，一个是per-cpu变量（内核开发中使用），另一个是[线程局部存储](http://www.maxwellxxx.com/TLS)（用户态开发中使用），它们分别会用到gs段和fs段。

+ 现代`X86`是四级页表。

+ 32位三级页表

  + PDE和PTE的每一项为32位。PD和PT的大小为4K。

  + 这种模式下仅能访问到4G的物理地址空间，应用程序和内核能用到的线性地址空间为4G。

    ![1560843558017](http://ww1.sinaimg.cn/large/77451733gy1g4fwwl4iyhj20d907jq3f.jpg)

+ 32位PAE模式

  + PDE和PTE的每一项为64位。PD和PT的大小为4K

  + 这种模式下能访问到64G的物理地址空间，应用程序和内核能用到的线性地址空间为4G

    ![1560843571145](http://ww1.sinaimg.cn/large/77451733gy1g4fwwyn6poj20dj089wf3.jpg)

+ X86_64（IA32E/AMD64）下的4级页表

  + PDE和PTE的每一项为64位。PD和PT的大小为4K

  + 理论上，这种模式下能访问到2^64大小的物理地址空间，应用程序和内核能用到的线性地址空间为2^64。实际上，能访问的线性地址空间大小为`2^48`次方。

  + 在linux内核中，这四级页表分别被称为`PGD、PUD、PMD、PT`。

    ![1560843673406](http://ww1.sinaimg.cn/large/77451733gy1g4fwxk0nauj20df0aaaat.jpg)

  + `PTE`项各字段的作用

    + P位，表示PTE指向的物理内存是否存在(Present)，这可以用于标识物理页面是否被换出
    + R/W位，表示这个页是否是可读/可写的。这可以用于fork的cow机制
    + D位，表示这个页是否是脏页。一旦有数据写入到这个页面，CPU会自动将这个位置位。这可以用来在内核做页面回收及换入换出时，判断是否需要将这个页的数据写入到磁盘中。（实际的实现机制不清楚，但理论上是可以的）
    + U/S位，表示这个页面是否能够在用户态下被访问，内核通过此位来识别用户空间或内核空间的页
    + CPL（current privilege level），用两位表示，分别对应0-3共四个级别，放在CS段选择子的低两位，表示当前的运行等级（ring0-ring3）CPL<3为supervisor模式，CPL=3为user模式
    + 在用户态下，访问U/S位为0的页将引起page fault，这用来阻止用户程序对内核空间的非法访问。

    ![1560844318534](http://ww1.sinaimg.cn/large/77451733gy1g4fwxwnv4nj20fe0aywho.jpg)

### 3.2、页框管理

![1560848883315](http://ww1.sinaimg.cn/large/77451733gy1g4fwya12bnj214g0b0n27.jpg)

+ 页框号`pfn`与`struct page`之间的转换，配置了`SPARSEMEM_VMEMMAP`的情况下：
  + [SPARSEMEM_VMEMMAP](https://blog.csdn.net/richardysteven/article/details/64122964)下只有有效的`page`才会有对应的`struct page`存在，`page`被组织到多个`mem_section`(page数组)中，先找`mem_section`再找`page`。

```c
// 此处相当于做了一个优化，加速转换，但是mem_section还是存在的
#if defined(CONFIG_SPARSEMEM_VMEMMAP)   
#define __pfn_to_page(pfn)  (vmemmap + (pfn))
#define __page_to_pfn(page) (unsigned long)((page) - vmemmap)
#elif defined(CONFIG_SPARSEMEM)
#define __page_to_pfn(pg)                   \
({  const struct page *__pg = (pg);             \
    int __sec = page_to_section(__pg);          \
    (unsigned long)(__pg - __section_mem_map_addr(__nr_to_section(__sec))); \
})

#define __pfn_to_page(pfn)              \
({  unsigned long __pfn = (pfn);            \
    struct mem_section *__sec = __pfn_to_section(__pfn);    \
    __section_mem_map_addr(__sec) + __pfn;      \
})
```

```c
#define NR_SECTION_ROOTS  DIV_ROUND_UP(NR_MEM_SECTIONS, SECTIONS_PER_ROOT) //1<<(19/8)=2k
#define SECTIONS_SHIFT  (MAX_PHYSMEM_BITS - SECTION_SIZE_BITS) // 46 - 27 = 19
#define NR_MEM_SECTIONS   (1UL << SECTIONS_SHIFT)  // 1 << 19
#define SECTIONS_PER_ROOT       (PAGE_SIZE / sizeof (struct mem_section))  // 4k/16 = 256
// SECTION_SIZE_BITS 为 27 表示每个 section 为 128M
struct mem_section {
	unsigned long section_mem_map; 
    unsigned long *pageblock_flags;
};

#define PAGES_PER_SECTION       (1UL << PFN_SECTION_SHIFT)  // 1 << 15 = 32k
#define PFN_SECTION_SHIFT (SECTION_SIZE_BITS - PAGE_SHIFT)  // 27 - 12 = 15
// 即每个 section 管理 32k * 4k = 128M 的物理内存
// 每个 struct pages 大小为 64B, 即每个 section 所存的 struct page 数组的大小为 32k*64B = 2M
```

```c
// 全局的，每个 NUMA 节点一个 pglist_data
struct pglist_data *node_data[MAX_NUMNODES] __read_mostly; 
#define MAX_NUMNODES    (1 << NODES_SHIFT) 
#define NODES_SHIFT     CONFIG_NODES_SHIFT   // 自己的服务器上为 10， 阿里云单核为 4 ？

#define MAX_NR_ZONES 5
// MAX_ZONELISTS 为 1, NUMA下 为 2, 本节点的 node_zonelists 不够可使用备用的 node_zonelists 分配
typedef struct pglist_data {
	struct zone node_zones[MAX_NR_ZONES];
	struct zonelist node_zonelists[MAX_ZONELISTS];
	int nr_zones;
    unsigned long node_start_pfn;
	unsigned long node_present_pages; /* total number of physical pages */
	/* total size of physical page range, including holes */
    unsigned long node_spanned_pages;
	int node_id;
}pg_data_t;

struct zone {  
    struct free_area free_area[MAX_ORDER]; // 每个 free_area 即伙伴系统的一个2^i个连续页框的节点
}  

enum zone_type {
    ZONE_DMA,
    ZONE_DMA32,
    ZONE_NORMAL,
    ZONE_HIGHMEM,
    ZONE_MOVABLE,
    ZONE_DEVICE,
    __MAX_NR_ZONES
};

// strcut page 中的 flags 成员中的某些位可以反推出该 page 属于哪一个 node 及哪一个 zone
// 每个zone中都有
```

+ FLATMEM

  ```c
  #define __pfn_to_page(pfn)  (mem_map + ((pfn) - ARCH_PFN_OFFSET))
  #define __page_to_pfn(page) ((unsigned long)((page) - mem_map) + \
                   ARCH_PFN_OFFSET)
  ```

  + 全局存在唯一一个`page`数组`mem_map`，`ARCH_PFN_OFFSET`在`x86`上为0，所以可以看出，只需要以`pfn`为索引，就能在`page`数组中找到对应的`page`，反之亦然。

+ DISCONTIGMEM

  ```c
  #define __pfn_to_page(pfn)          \
  ({  unsigned long __pfn = (pfn);        \
      unsigned long __nid = arch_pfn_to_nid(__pfn);  \
      NODE_DATA(__nid)->node_mem_map + arch_local_page_offset(__pfn, __nid);\
  })
  
  #define __page_to_pfn(pg)                       \
  ({  const struct page *__pg = (pg);                 \
      struct pglist_data *__pgdat = NODE_DATA(page_to_nid(__pg)); \
      (unsigned long)(__pg - __pgdat->node_mem_map) +         \
       __pgdat->node_start_pfn;                   \
  })
  ```

  + 物理地址空间被分段组织到一个`node`中，`node->node_mem_map`作为`page`的数组。
  + 在将`pfn`转换为`page`时，需要先通过`pfn`得到对应的`node id`，由此`node id`得到`node->node_mem_map`首地址，然后再通过`pfn`得到在该`page`数组中的偏移，从而得到对应的`page`结构。
  + 在将`page`转换成`pfn`时，先通过`page`找到对应的`node`，再根据`node->node_mem_map`得到`page`在该`node`中的偏移，再加`上node->node_start_pfn`就能得到正确的`pfn`了。

+ SPARSEMEM_VMEMMAP

  ```c
  #define __pfn_to_page(pfn)  (vmemmap + (pfn))
  #define __page_to_pfn(page) (unsigned long)((page) - vmemmap)
  ```

  + 所有的`page`被放在了一块`VMEMMAP`区域，这是一段虚拟地址空间。在64位机器上，线性地址空间极大，不在乎浪费这么一些线性地址空间来存放`page`数组。
  + 这样存放`page`结果带来的直接好处就是，`pfn`与`page`的相互转换变得极其简单，只需要算偏移即可。

+ SPARSEMEM

  ```c
  #define __page_to_pfn(pg)                   \
  ({  const struct page *__pg = (pg);             \
      int __sec = page_to_section(__pg);          \
      (unsigned long)(__pg - __section_mem_map_addr(__nr_to_section(__sec))); \
  })
  
  #define __pfn_to_page(pfn)              \
  ({  unsigned long __pfn = (pfn);            \
      struct mem_section *__sec = __pfn_to_section(__pfn);    \
      __section_mem_map_addr(__sec) + __pfn;      \
  })
  ```

  + 在没有使用`vmemmap`机制时，`pfn`和`page`的相互转换工作就变得相对复杂了。

  + 内存热插拔时，没有物理内存的区域不会分配`struct page`。

  + 在`sparse`模式下，所有的`page`通过一个名叫`mem_section`的二级表组织起来。

    ![1562677985644](http://ww1.sinaimg.cn/large/77451733gy1g4twbxq51rj20tn0eu0vi.jpg)

### 3.3、buddy系统

+ 用于管理页框缓冲池，解决频繁申请不同大小的连续页框导致的外部碎片问题。	

+ 把所有的空闲页框组织为11个块链表：每个链表分别含`2^0~2^10`个连续的空闲页框。

+ 假设要找256个连续空闲页框，先定位到256对应的链表，如果没有再定位到512对应的链表，有的话就分割，并把剩余的256加入256对应的链表，以此类推。

+ NUMA的一个节点叫做`node`，linux中表示为`pglist_data_t`

+ linux下的每个node被划分成多种zone，

  + zone分为以下几类，需要注意的是，一个编译好的linux系统中并不一定存在以下所有类型，比如在64位机器上ZONE_HIGHMEM就并不存在。

    ```c
    enum zone_type {
        ZONE_DMA,
        ZONE_DMA32,
        ZONE_NORMAL,
        ZONE_HIGHMEM,
        ZONE_MOVABLE,
        ZONE_DEVICE,
        __MAX_NR_ZONES
    };
    ```

  + ZONE_DMA，在i386的机器上，有些硬件只能寻址到16M以下的地址空间，这些硬件的DMA操作只能从该类型的zone中分配空间

  + ZONE_DMA32，在x86_64的机器上，有些硬件可以寻址到4G以下的空间了，这些硬件的DMA操作就能从ZONE_DMA32的zone中分配空间了

  + ZONE_NORMAL，系统中绝大部分物理内存属于这一类型

  + ZONE_HIGHMEM，有些地址区域需要通过mapping才能进行访问，如i386机器。在32位系统上，4G的虚拟地址空间被划分为3G用户空间和1G内核空间，这1G内核空间被划分为三部分，直接映射区域、通过vmap访问的区域以及通过kmap访问的区域。在32位系统中，超过896MB的物理地址空间需要通过kmap进行访问，这些物理地址空间被称为highmem.

  + ZONE_MOVABLE，与内核的反碎片和可插拔内存技术有关。通过命令行参数kernelcore和movablecore可进行配置。

    + kernelcore=nn[KMGTPE]，这个参数决定了内核可以用的non-movable的内存空间大小，这些空间平摊到每个node上，每个node上剩余的空间就被当做movable。如果一个node的可用物理空间太小了，无法同时拥有movable和kernelcore的空间，那么这些空间优先作为kernelcore的空间。这些movable的空间用于page migration，可以被用作reclaim或者move。如此一来，那些大页就无法从这些movable空间上分配了。

    + movablecore=nn[KMG]，这个参数与kernelcore类似，不过指定的是可以用于page migration的内存大小。如果同时指定了两个参数，则kernelcore的数值表示至少要多少unmovable的内存。

  + ZONE_DEVICE，这个标志与NVM/PM有关。

### 3.4、slab分配器

+ 对若干来自于buddy系统的页框块进行有效管理，以满足不同大小“对象”分配的请求。
+ kmalloc_index函数返回kmalloc_caches数组的索引。
+ kmalloc根据对象大小，到相应的缓冲区中进行分配。
+ 当所分配对象大于8KB时，kmalloc维护的13个缓冲区已经无法使用了。
+ 通过调用buddy系统获取页框。
  | kmalloc_caches数组索引 | 对象大小范围（字节） |
  | ---------------------- | -------------------- |
  | 1                      | (64, 96]             |
  | 2                      | (128,   192]         |
  | 3                      | (0,   8]             |
  | 4                      | (8,   16]            |
  | 5                      | (16,   32]           |
  | 6                      | (32,   64]           |
  | 7                      | (96,   128]          |
  | 8                      | (192, 256]           |
  | 9                      | (256,   512]         |
  | 10                     | (512,   1024]        |
  | 11                     | (1024, 2048]         |
  | 12                     | (2048, 4096]         |
  | 13                     | (4096,   8192]       |

### 3.5、kmalloc和vmalloc

+ kmalloc
  + 根据`flags`找到slab中对应的`kmem_cache`（如`__GFP_DMA`，`__GFP_NORMAL`等），并从中分配内存。
  + 分配的内存地址在物理地址和虚拟地址是上都连续。
  + 内核出于性能的考虑，大多使用kmalloc，因为vmalloc页表映射的开销。
+ vmalloc
  + 分配的内存地址虚拟地址上连续，在物理地址上不要求连续。
  + 需要建立专门的页表项把物理上不连续的页转换为虚拟地址空间上的页。映射时会出现TLB抖动，因为多次查映射表可能在`TLB`（Translation lookaside buffer，是一种硬件缓冲区）中不断切换。

### 3.6、用户态内存管理：mmap和brk

> 简单来说就是先从`per-thread vma`缓存中找再从红黑树中找满足大小要求的`vm_area_struct`。虚拟地址空间映射好后，物理内存的的分配与映射是通过`page fault`完成的。对于文件映射会先预读数据到内核中。如果指定了`populate`选项会直接先建立到物理内存的分配与映射。匿名映射是先创建一个创建一个临时的`shm file`（包括它的`dentry`，`inode`和`file`结构体）再与合适的`vma`映射。

+ `mmap`的那些范围通过`vm_area_struct`结构体管理。这个结构体以起始范围为key，通过红黑树连接起来，每个相邻的`vm_area_struct`之间同时还使用双向链表连接。
+ 用户态与`mmap`有关的流程：
  + 调用`mmap`，映射一个文件的一部分或者一段内存空间
  + 如果`mmap`的时候要求`populate`，则需要先建立相关的映射（对于文件而言，需要预读数据到内核中，并建立映射关系；对于内存而言，需要映射相关的内存空间）
  + 在用户态的运行过程中，可能会在访问某个地址时出现`page fault`。这时，`page fault`经过层层检查，发现是`mmap`的一段区域引发的，就会去建立映射关系（如上文）。
  + 在建立好映射关系后，重新执行引起`page fault`的指令
  + 调用`msync`或者取消映射时，对于文件而言，需要将数据刷到磁盘上
  + 调用`munmap`取消映射
+ `mmap`内部的实现：
  + 通过`sys_mmap`进入内核（函数实现在[/](https://elixir.bootlin.com/linux/v4.16.18/source)[arch](https://elixir.bootlin.com/linux/v4.16.18/source/arch)/[x86](https://elixir.bootlin.com/linux/v4.16.18/source/arch/x86)/[kernel](https://elixir.bootlin.com/linux/v4.16.18/source/arch/x86/kernel)/[sys_x86_64.c](https://elixir.bootlin.com/linux/v4.16.18/source/arch/x86/kernel/sys_x86_64.c)中） 
  + 调用到了`sys_mmap_pgoff`（函数实现在[/](https://elixir.bootlin.com/linux/v4.16.18/source)[mm](https://elixir.bootlin.com/linux/v4.16.18/source/mm)/[mmap.c](https://elixir.bootlin.com/linux/v4.16.18/source/mm/mmap.c)中）
    + 如果是非匿名映射，获取`fd`对应的`file`。如果`file`是大页文件系统的文件，将`length`对齐到`2M`
    + 否则，如果设置了`MAP_HUGETLB`标志，在大页文件系统中创建一个`file`
    + 调用`vm_mmap_pgoff`
      + 安全检查
      + 调用`do_mmap_pgoff`，这个函数转发数据到`do_mmap`。`do_mmap`的流程见下文。
      + 如果用户指定了要`populate`，则调用`mm_populate`创建具体的映射关系
  + 完成`mmap`，将`file`的引用计数减一
+ `do_mmap`
  + 参数检查
  + 获取未映射的内存区域
    + 如果是文件映射，且`file->f_op->get_unmapped_area`不为`NULL`，则调用该函数搜索未映射的内存区域
    + 如果是共享内存映射，则调用`shmem_get_unmapped_area`搜索未映射的内存区域
    + 如果上述两者都为`NULL`，则调用`current->mm->get_unmapped_area`，该函数一般对应到`arch_get_unmapped_area`（位于[/](https://elixir.bootlin.com/linux/v4.16.18/source)[arch](https://elixir.bootlin.com/linux/v4.16.18/source/arch)/[x86](https://elixir.bootlin.com/linux/v4.16.18/source/arch/x86)/[kernel](https://elixir.bootlin.com/linux/v4.16.18/source/arch/x86/kernel)/[sys_x86_64.c](https://elixir.bootlin.com/linux/v4.16.18/source/arch/x86/kernel/sys_x86_64.c)中）
  + 对其他参数进行检查
  + 调用`mmap_region`进行映射。
+ `mmap_region`
  + 将映射范围内原来的那些已经被映射的空间`unmap`掉
  + 判断能否和前后范围内的`vma`合并，如果能合并，则函数返回
  + 否则，新建一个`vma`并初始化
  + 如果是文件映射，则调用`file->f_op->mmap`来进行具体的映射
  + 否则，如果是匿名共享内存映射，则创建一个临时的`shm file`（包括它的`dentry`，`inode`和`file`结构体），并使用`shmem_file_operations/shmem_inode_operations/shmem_aops`等来初始化`file/inode/address_space`，对于`vma`，使用`shmem_vm_ops`来初始化
  + 接下来，将`vma`加入到红黑树中
+ `brk`也会去把已经映射的区域`unmap`掉，然后再做`mmap`。`brk`其实是一个简陋的`mmap`，它不管文件，`cow`之类的问题，只匿名映射内存。在`malloc`的实现中，有了`mmap`，一般不会去调用`brk`，因为它有可能把原来的映射给释放掉。

### 3.7、内核地址空间共享机制

+ 内核地址空间是共享的，每当`fork`一个进程时，内核会拷贝属于内核地址空间的那部分`pgd`表，使得所有的进程的`pgd`表中关于内核地址空间的部分都指向同样的`pud`表。
+ 但是在运行过程中，内核可能又在`vmalloc`区域映射了一些新的页，而其他进程是不知道的，这些进程在陷内核访问这些页时，会触发`page fault`。在`page fault`的处理例程中，会去`init_mm`中查找对应区域的这些页表项，将这些页表项拷贝到触发`page fault`的进程的页表项中。（事实上，只有`pgd`表项为空，才会触发`vmalloc`的`page fault`）
+ 对于内核`module`来说，如果在运行时动态加载了一个`module`，那么它的代码会被加载到一个专门的区域中，但这个区域并不在`vmalloc`区域内，那么内核中的其他进程怎么知道这一新映射的`module`代码的位置呢？事实上，这一区域能够被一个`pgd`表项`cover`住，那么，在`fork`的时候，所有进程的这一个`pgd`表项，已经对应到了同一个对应的`pud`页。内核在映射`module`的时候，是修改的`pud/pmd`等页表项，其他进程自然能够看到这一映射关系，而不会引发`page fault`

### 3.8、共享内存

+ 这一套是在`glibc`里面实现的
  + `shm_open`创建一个`file`，获取对应的`fd`
  + `mmap`映射`fd`
  + `munmap`取消映射
  + `shm_unlink`减少引用计数
+ 这一套机制是通过`open`在`/dev/shm`下面创建文件，用`mmap`来映射的。这套机制会经过`fdtable`，所以更为安全。

### 3.9、 ptmalloc

+ `ptmalloc`通过`mmap`或brk（仅`main_arena`使用）每次批发`64MB`大小的内存（称为`heap`）来管理。
+ `ptmalloc`中有多个`arena`，即`struct malloc_state`。只有一个main_arena，其它的`arena`是动态分配的，但数量有上限，这些`arena`以链表的形式组织。分配内存找`arena`时可能会逐个对空闲的`arena`进行`try_lock`。
+ 每个`arena`管理多个`heap`。每个`heap`通过`prev`指针连接起来，`arena->top`指向最近分配的`heap`中的，未分配给用户的，包含了`heap`尾部的那个`chunk`。通过`arena->top`，能够找到最近的那个`heap`。通过最近的那个`heap`的`prev`指针，能够依次找到以前所有的`heap`。
+ `heap`释放时会出现某一个`heap`未全部空闲则该heap前面的空闲的`heap`无法得到释放的问题。
+ 用户请求的空间都用`chunk`来表示。
+ 所有的空闲的chunk被组织在`fastbin`，`unsorted bin`，`small bin`，`large bin`中。
+ `fastbin`可看作`small bin`的缓存，缓存`16B~64B`（以`8B`递进，共`10`个）的`chunk`。
+ `unsorted bin`，`small bin`，`large bin`都是在bins数组中，只是不同`index`范围区分了他们。
+ `chunk`被分配出去时，其在空闲链表中的前后指针以及前后`chunk size`的字段会被复用给用户作为空闲区域。
+ 每个线程有`tcache`，并且每个线程有一个`thread_arena`指向当前线程正在使用的`arena`。
+ `unsorted bin`可以看作`small bin`和`large bin`的缓冲，链表头是`bins[1]`。`fastbin`中的空闲`chunk`合并时会先放到`unsorted bin`中。分配时检查`unsorted bin`没有合适的chunk就会将`unsorted bin`中的chunk放到`small bin`或`large bin`中。
+ `small bin`中的chunk大小范围为`32B~1088B`（以`16B`递增，共`62`个`bin`链表，`bin[2]~bin[63]`）。
+ `large bin`中的chunk大小范围为`1024B`以上。
+ 另外，每个线程有一个`tcache`，共64项，从最小的32字节开始，以`16`字节为单递增。
+ 链表操作通过`CAS`，`arena`争用需要加锁。

### 3.10、[jemalloc](https://youjiali1995.github.io/allocator/jemalloc/)

+ `jemalloc` 中大量使用了宏生成代码，比较晦涩。
  + 通过避免 `false cache line sharing`，使用内存着色等，提高 `cache line` 效率
  + 使用多个 `arena` 管理、更细粒度的锁、 `tsd`、`tcache`等，最小化锁竞争
  + 使用 `slab` 分配不同大小的对象，精心选择 `size classes`，减少内存碎片
  + 使用多层缓存，内存的释放和分配会经历很多阶段，提升速度
+ 每个线程有一个`thread specific data`即 `struct tsd_s tsd`，其中有两个指针，`iarena`和`arena`分别指向用于元数据分配和用于普通数据分配的`arena`。所以，`tsd`需要找两个`arena`进行绑定（这两个`arena`可能是同一个）。
+ `jemalloc`会创建多个`arena`，每个线程由一个 `arena` 负责。`arena`有引用计数字段`nthreads[2]`，`nthreads[0]`计录普通数据分配`arena`指针绑定这个`arena`的次数，`nthreads[1]`记录元数据分配`iarena`指针绑定这个`arena`的次数。一个`tsd`绑定`arena`后，就不会改变`arena`。
+ 有一个全局变量`narenas_auto`，它在初始化时被计算好，表示能够创建的`arena`的最大数量。
+ 有多个`arena`，以全局数组的形式组织。每个线程有一个`tcache`，其中有指向某个`arena`的指针。
+ 当需要绑定一个`arena`时，遍历所有已创建的`arena`，并保存所有`arena`中`nthreads`值最小的那个（根据是绑定元数据还是普通数据，判断使用`nthreads[0]`还是`nthreads[1]`）。
  + 如果在遍历途中发现数组中有`NULL`（也就是说数组有`slot`，还可以创建新的`arena`），那么就创建一个新的`arena`，将`arena`放到那个`slot`中，并绑定在那个`arena`上
  + 如果遍历途中发现所有的`slot`都被用上了，那么就选择`nthreads`值最小的那个，绑定那个`arena`
+ `jemalloc`通过`mmap`以`chunk`（默认2M）为单位向操作系统申请内存。每个`chunk`的头部会有一个`extent_node_t`记录其元数据信息，如所属的`arena`和起始地址。这些`chunk`会以基数树的形式组织起来，保存 `chunk` 地址到 `extent_node_t` 的映射
+ `jemalloc`内部动态分配的内存通过`base`组织。`base` 使用 `extent_node_t` 组成的红黑树 `base_avail_szad` 管理 `chunk`。每次需要分配时，会从红黑树中查找内存大小相同或略大的、地址最低的 `node`， 然后从 `node` 负责的 `chunk` 中分配内存。
+ `chunk` 使用 `Buddy allocation` 划分为不同大小的 `run`。`run` 使用 `Slab allocation` 划分为固定大小的 `region`，大部分内存分配直接查找对应的 `run`，从中分配空闲的 `region`，释放就是标记 `region` 为空闲。
+ `jemalloc` 将对象按大小分为3类，不同大小类别的分配算法不同:
  - `small`（`8B-14K`）: 从对应 `bin` 管理的 `run` 中返回一个 `region`
  - `large（`16K-1792K`）`: 大小比 `chunk` 小，比 `page` 大，会单独返回一个 `run`
  - `huge（`2M-64M`）`: 大小为 `chunk` 倍数，会分配 `chunk`
+ `mutex` 尽量使用 `spinlock`，减少线程间的上下文切换

### 3.11、内部碎片和外部碎片

+ 内部碎片：已经被分配出去的的内存空间大于请求所需的内存空间。能明确指出属于哪一个进程。
  + 比如请求6字节，可能实际申请8字节。
  + 用户态malloc解决。
+ 外部碎片：指还没有分配出去，但是由于大小而无法分配给申请内存空间的新进程的内存空闲块。不属于任何进程。
  + buddy和slab解决。
+ 磁盘上的文件碎片：把文件分成几个片分别存放（往往因为空间不够），磁盘整理就是整理文件，使之紧凑，留出更多可用空闲空间。

## 4. 进程管理

+ [`0`号进程](https://blog.csdn.net/gatieme/article/details/51484562)是`idle`进程，`1`号是`init`即 `systemd`进程，`2`号是`kthreadd`进程
+ 当没有进程需要调度器调度时，就执行`idle`进程，其`task_struct`通过宏初始化，存放在内核`data`段
+ 内核`init_task`即`idle`进程的栈大小：`x64`为`16k`，`x86_32`为`8k`
+ `task_struct`内部存一个`void *stack`指向栈的基址
+ 进程和线程的区别与联系：
  + 子进程(子线程)的调度一般与父进程(父线程)平等竞争
  + 进程是资源分配的基本单位，线程是调度的基本单位。
  + 同一个进程下的多个子线程共享进程的
  + 进程的ID是系统唯一的，线程的ID（`unsigned long`）仅在它所属的进程上下文中才有意义。
  + 线程共享进程的：
    + 代码段
    + 公有数据(利用这些共享的数据，线程很容易的实现相互之间的通讯) 
    + 文件描述符、信号的处理器、进程的当前目录和进程用户ID与进程组ID。
  + 线程独立的资源：
    + 线程ID
    + 寄存器组的值
    + 线程的堆栈
    + errno
    + 线程的信号屏蔽码：由于每个线程所感兴趣的信号不同，所以线程的信号屏蔽码应该由线程自己管理。但所有的线程都 共享同样的信号处理器。
    + 线程的优先级
  + 进程间通信IPC需要特别的方法，线程间可以直接读写进程数据段（如全局变量）来进行通信。
+ 线程终止：
  + 自身调用`pthread_exit`，退出状态可被同进程下的另一个子线程调用`pthread_join`捕获。此状态可以存一个指向堆上的对象的指针，但不能是栈上的，因为线程退出后栈会被撤销。
  + 被同进程下的另一个子线程`cancel`
  + 从启动例程中返回。
  + 进程中的任一线程调用`exit`，或者执行`main`函数的主线程return了，其余所有的线程都是退出。
+ [协程](https://jiajunhuang.com/articles/2018_04_03-coroutine.md.html)：即用户态线程，分为有栈协程和无栈协程。
  + `python`为无栈协程，每次执行到这个coroutine的时候，是把其帧丢到调用者的栈上的，每一个帧为一个`yield`点分隔的函数段。
  + `go`为有栈协程，切换时会保存整个栈的状态。
+ 一般的`pthread`线程都是内核态线程，每次线程切换都需要陷入到内核，由操作系统来调度。另外内核态实现会占用内核稀有的资源，因为操作系统要维护线程列表，操作系统所占内核空间一旦装载后就无法动态改变，并且线程的数量远远大于进程的数量，随着线程数的增加内核将耗尽；
+ `fork`实现：
  + 页表的拷贝是复制父进程的所有页表，只是最后一级页表指向的物理页相同，但需要把那一项设置为只读，在写时会触发page fault重新分配物理页。
  + 文件的拷贝是拷贝`file`数组，指针指向的`file`为相同的，父子进程对文件偏移的更改互相是看得到的。

### 线程相关

```c++
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void *(*start_routine) (void *), void *arg);
pthread_attr_init(3), 
pthread_atfork(3), 
pthread_cancel(3), 
pthread_cleanup_push(3), 
pthread_cond_signal(3),
pthread_cond_wait(3), 
pthread_create(3), 
pthread_detach(3), 
pthread_equal(3), 
pthread_exit(3), 
pthread_key_create(3), 
pthread_kill(3), 
pthread_mutex_lock(3),
pthread_mutex_unlock(3), 
pthread_once(3), 
pthread_setcancelstate(3), 
pthread_setcanceltype(3), 
pthread_setspecific(3), 
pthread_sigmask(3), 
pthread_sigqueue(3), 
pthread_testcancel(3)
    
pthread_cond_signal
pthread_cond_wait
pthread_cond_broadcast
pthread_mutex_lock
pthread_mutex_init
pthread_mutex_destroy

pthread_spin_init
pthread_spin_lock
pthread_spin_trylock
pthread_spin_destroy
pthread_spin_unlock

pthread_rwlock_init
pthread_rwlock_destroy
pthread_rwlock_rdlock
pthread_rwlock_wrlock
pthread_rwlock_timedrdlock

```

+ [线程底部实现](http://yangxikun.com/linux/2013/11/27/process-and-thread.html)调用`clone()`系统调用，线程同步采用`futex()`系统调用。

#### pthread_once()

> glibc/nptl/pthread_once.c
>
> Native POSIX Thread Library(NPTL)

```c
int pthread_once(pthread_once_t *once_control, void (*init_routine)(void));

pthread_once_t once_control = PTHREAD_ONCE_INIT;
```

+ [内部采用双检查机制](https://blog.csdn.net/abcd1f2/article/details/50151349)
+ 其中的 `lll_lock`添加了内存屏障，保证了不会编译器乱序，`x86`架构保证了不会CPU乱序
+  `x86/x64`平台上双检查机制（`DCLP`）不会出现问题，但这只是解决了`cpu`重排问题，对于编译器重排我们还要进行保证
+ `local static`变量不是多线程安全的，而`c++11`中的`local static`是多线程安全的

#### pthread_create()

> glibc/nptl/pthread_create.c

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
				   void *(*start_routine) (void *), void *arg);
```

#### [pthread_cancel](https://www.cnblogs.com/lijunamneg/archive/2013/01/25/2877211.html)

#### 条件变量

+ 接口

  ```c
  #include <pthread.h>
  
  int pthread_cond_init(pthread_cond_t *restrict cond,
                        const pthread_condattr_t *restrict attr);
  // 不能由多个线程同时初始化一个条件变量。
  pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
  
  int pthread_cond_destroy(pthread_cond_t *cond);
  
  int pthread_cond_timedwait(pthread_cond_t *restrict cond,
                             pthread_mutex_t *restrict mutex,
                             const struct timespec *restrict abstime);
  int pthread_cond_wait(pthread_cond_t *restrict cond,
                        pthread_mutex_t *restrict mutex);
  
  int pthread_cond_broadcast(pthread_cond_t *cond);
  int pthread_cond_signal(pthread_cond_t *cond);
  ```

+ 内部实现

  ```c
  #define PTHREAD_COND_INITIALIZER { { {0}, {0}, {0, 0}, {0, 0}, 0, 0, {0, 0} } }
  
  // glibc/sysdeps/nptl/bits/thread-shared-types.h
  struct __pthread_cond_s {
      // waiter序列计数器，waiters获得相关mutex后seq+1
      unsigned long long int __wseq; 
      // g1组开始的下标，signalers获得内部cond lock的时候修改这个值
      // 只有g1组的所有waiter获得信号，才会将g2组转化为g1组并获取信号。
      unsigned long long int __g1_start; 
      // 分别记录g1组和g2组中的引用数量，即组内使用futex的数量
      // 在开始waitting之前ref加1，在cancel waitting之后获取mutex之前减一，只有0才能交换g1和g2
      unsigned int __g_refs[2] __LOCK_ALIGNMENT;
      // 分别记录g1组和g2组中没有获得信号的waiter数量
      unsigned int __g_size[2];
      // g1组初始化大小，两个最低有效位代表cond内部lock，只有获得lock才能access
      unsigned int __g1_orig_size;
  };
  ```

#### 问题

+ 线程的概念，和进程的区别
+ 可重入与不可重入
+ 线程安全
+ 自旋锁
+ `CAS`
+ 多线程，优缺点
+ 线程状态
+ 线程的实现方式
+ 各类锁
+ 无锁队列
+ 单例模式
+ 线程数据布局
+ 线程同步与通信
+ `c++`多线程：`std::thread`
+ 开源项目中多线程的使用
+ [c++11的六种内存序](https://www.zhihu.com/question/24301047)

#### muduo

+ 析构函数与多线程，最终`shared_ptr`解决
+ 线程池大小的阻抗匹配原则：`T=可用CPU数目(C)/密集计算占比(P)`，`P < 0.2`时可取固定值为`0.2`
+ `Proactor`能提高吞吐，但不能降低延迟（P82）
+ 多线程，避免`CPU Cache`换入换出（P82）

#### 其它

+ `mutable`关键字：用来修饰一个 `const` 部分可变的数据成员，例如修饰类内部`mutex_`，使得`mutex_`可被`const`成员函数访问并修改。
+ `volatile`关键字：编译器将不对其相关代码执行优化，而是生成对应代码直接存取原始内存地址，即不从`寄存器`读
+ [`restrict`关键字](https://www.zhihu.com/question/41653775)：告诉编译器不会有别的指针指向与所修饰的指针指向同一数据
+ `socketpair(2)`
+ 服务器：`netstat -tpna | grep :port`；客户端：`netstat` 或 `lsof` 可查看那些进程与服务器连接，与连接情况
+ `Recv-Q`和`Send-Q`都应当为0
+ `x64`下`ulimt -s`查看进程默认栈大小为`8M`
+ [Amdahl's Law](https://zh.wikipedia.org/wiki/%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B)：用于计算并行计算中的加速比，`muduo`中用此定律说明即便算法的并行度高达`95%`，8核的加速比也只有6
+ [每个程序员都应该了解的 CPU 高速缓存](https://www.oschina.net/translate/what-every-programmer-should-know-about-cpu-cache-part2?print)
+ `fseek`和`fread`的实现
+ `printf`是线程安全的，`std::cout`不是
+ `_exit`与`exit`
+ `read`和`pread`
+ `printf`会对`stdout`加锁
+ `signal handler`将`signalfd`注册到事件循环代替`signal handler`处理
+ `sig_atomic_t`是`int`类型的，保证该变量在使用或赋值时操作是原子的
+  **硬件性能对比，包括cache，读写速度**
+ **系统调用过程中被抢占**？

## 5.  地址空间

> **0000000000000000 - 00007fffffffffff (=47 bits) user space, different per mm**
> hole caused by [47:63] sign extension
> ffff800000000000 - ffff87ffffffffff (=43 bits) guard hole, reserved for hypervisor
> **ffff880000000000 - ffffc7ffffffffff (=64 TB) direct mapping of all phys. memory**
> ffffc80000000000 - ffffc8ffffffffff (=40 bits) hole
> **ffffc90000000000 - ffffe8ffffffffff (=45 bits) vmalloc/ioremap space**
> ffffe90000000000 - ffffe9ffffffffff (=40 bits) hole
> **ffffea0000000000 - ffffeaffffffffff (=40 bits) virtual memory map (1TB)**   //vmemmap，存放所有struct page*

![](http://ww1.sinaimg.cn/large/77451733gy1g4tx1l3olfj20yf0iw40i.jpg)

+ 用户地址空间

  > 从低地址到高地址：
  >
  > - text  代码段 —— 代码段，一般是只读的区域;
  > - static_data 段
  > - stack 栈区 —— 局部变量，函数的参数，返回值等，由编译器自动分配释放;
  > - heap 堆区 —— 动态内存分配，由程序员分配释放;
  
+ 用户空间和内核空间
    + CPU其实并不知道什么用户空间和内核空间，它是通过PTE的U/S位与CPL来判断这个页是否可以被访问。所以，内核空间的那些页面对应的PTE与用户空间对应的PTE中，U/S位实际上是不同的，内核通过这一方式来划分用户空间和内核空间。
    + 用户态显然不可以访问内核空间，但是内核态下也不一定就能访问用户空间。这与CPU的配置有关，规则很复杂，可以参考Intel手册卷三4.6节，这里提一个关键字SMAP，有兴趣可以自行搜索，在较新的内核版本中，这一机制已经被使用。
    + 关于[vdso和vvar](http://readm.tech/2016/09/23/syscall/)：主要将部分安全的内核代码映射到用户空间，这使得程序可以不进入内核态直接调用系统调用。

    ![1560844894694](http://ww1.sinaimg.cn/large/77451733gy1g4fwys48n0j210s0k646v.jpg)

    + 线性地址空间划分如上图所示。内核空间的划分是确定的，写在内核代码中的，而用户态的空间在可执行文件被装载时才知道，由装载器和链接器来决定（可能需要参考elf相关的文档，才知道具体的装载位置）。不过，用户态空间整体的布局如上图所示，内核的current->mm结构体中记录着相关段的位置。
+ [`task_struct`](<https://elixir.bootlin.com/linux/v4.18/source/include/linux/sched.h#L593>)

+ [`thread_info`](https://blog.csdn.net/gatieme/article/details/51577479)

+ [`task_struct->active_mm`](<http://www.wowotech.net/process_management/context-switch-arch.html>)

  ```c++
  typedef unsigned long	pgdval_t;
  typedef struct { pgdval_t pgd; } pgd_t;
  
  struct mm_struct {
      // 管理mmap出来的内存，brk出来的也是在这brk就是一个简化版的 mmap, 只映射匿名页
      struct vm_area_struct *mmap;   
      struct rb_root mm_rb;       	// 通过红黑树组织
      u32 vmacache_seqnum;                   /* per-thread vmacache */
      pgd_t * pgd;                // 页表指针，与 arch 相关, x64 下 pgd_t 为 unsigned long
  };
  
  struct task_struct {
  	struct thread_info    thread_info; // 位于栈顶部，可用来定位 task_struct
  	void *stack;   // 指向栈
      struct mm_struct *mm, *active_mm;   // 内核线程没有 mm, 但有 active_mm
     	// active_mm 用于 context_switch 时的加速
      
      // 每个vma表示一个虚拟内存区域
      // VMACACHE_SIZE 为 4，per-thread vma缓存
      // 4.4 为 struct vm_area_struct *vmacache[VMACACHE_SIZE]; 
      // 通过 #define VMACACHE_HASH(addr) ((addr >> PAGE_SHIFT) & VMACACHE_MASK)
     	// 决定vma的数组存储位置，PAGE_SHIFT 为 12，VMACACHE_MASK 为 VMACACHE_SIZE-1 即 3
      struct vmacache     vmacache; 
  
  };
  
	struct vm_area_struct {
  // [vm_start, vm_end)
  	unsigned long vm_start;		/* Our start address within vm_mm. */
  	unsigned long vm_end;   // The first byte after our end address within vm_mm.
  	unsigned long vm_flags;		/* 设置读写等权限和是否为共享内存等等，见 mm.h */
  	/* linked list of VM areas per task, sorted by address */
  	struct vm_area_struct *vm_next, *vm_prev; // 2.6 只有 next
      struct rb_node vm_rb;
      struct mm_struct *vm_mm;	/* The address space we belong to. */
      /* Function pointers to deal with this struct. */
  	const struct vm_operations_struct *vm_ops;
      struct file * vm_file;		/* File we map to (can be NULL). */
      /* Information about our backing store: */
  	unsigned long vm_pgoff;		/* Offset (within vm_file) in PAGE_SIZE units */
  };
  
  struct vmacache {
  	u32 seqnum; // 4.4放在 task_struct 中, 当其与 mm_struct 中的 seqnum 不一致时缓存失效
  	struct vm_area_struct *vmas[VMACACHE_SIZE];
  };
  ```
  

## 6. 文件系统

### 6.1 文件描述符表(fd)

+ 其实就是每个进程的`file`数组的下标。

+ `fd = get_unused_fd_flags` 获取一个可用的`fd`

  ```bash
  get_unused_fd_flags -> __alloc_fd 
  # 内部通过`current->files`获取到当前进程的`struct files_struct *files`，
  # __alloc_fd对files加锁，从0开始逐个找下一个可用的fd,fd是否被占用以0/1 bit位图标记。
  # 所有系统调用都是通过current来访问当前进程数据结构。
  ```

+ [`current`的实现](https://feilengcui008.github.io/post/linux%E5%86%85%E6%A0%B8%E6%A0%88/)：定义一个`per_cpu`变量的`current_task`指针，返回这个指针。

  ```c
  // linux 4.18
  DECLARE_PER_CPU(struct task_struct *, current_task);
  static __always_inline struct task_struct *get_current(void)
  {
          return this_cpu_read_stable(current_task);
  }
  #define current get_current()
  ```

### 6.2 socketfd

+ `int socket(int domain, int type, int protocol);`返回一个fd。
  + 内部调用`sock_alloc_file`，对应的`f_op`为`socket_file_ops`，`f_op.poll`为`sock_poll`，`sock_poll`根据情况会调用到`tcp_poll`或`udp_poll`等。

## 7. 系统调用

### 系统调用的开销

+ 平时说的系统调用开销大，主要是相对于函数调用来说的。
  + 对于一个函数调用，汇编层面上就是一个CALL或者JMP，这种指令在硬件层面上虽然首次是会打乱流水线的，但如果是十分有规律的情况下，大多数CPU都能很好的处理。
  + 对于系统调用来说，过去Linux采用的是`INT 80H`中断的方式处理系统调用，现在是通过`syscall`指令实现
    + 当系统调用参数小于等于6个时，参数则必须按顺序放到寄存器 `rdi，rsi，rdx，r10，r8，r9`中
    + 当系统调用参数大于6个时，全部参数应该依次放在一块连续的内存区域里，同时在寄存器 `ebx` 中保存指向该内存区域的指针
    + 系统调用号和返回值放在`eax`里。
  + 开销比普通函数调用大的地方：
    + 需要设置各种寄存器、切换权限等级，各种检查。

## 8. 信号

## 9. IO管理

## 10. 中断和异常

### 1、[中断、异常和陷入](https://www.cnblogs.com/johnnyflute/p/3765008.html)

+ 中断：是为了设备与CPU之间的通信。
  + 典型的有如服务请求，任务完成提醒等。比如我们熟知的时钟中断，硬盘读写服务请求中断。
  + 断的发生与系统处在用户态还是在内核态无关，只决定于`EFLAGS`寄存器的一个标志位。我们熟悉的sti, cli两条指令就是用来设置这个标志位，然后决定是否允许中断。
  + 中断是异步的，因为从逻辑上来说，中断的产生与当前正在执行的进程无关。
+ 异常：异常是由当前正在执行的进程产生。异常包括很多方面，有出错（fault），有陷入（trap），也有可编程异常（programmable exception）。
  + 出错（fault）和陷入（trap）最重要的一点区别是他们发生时所保存的EIP值的不同。出错（fault）保存的EIP指向触发异常的那条指令；而陷入（trap）保存的EIP指向触发异常的那条指令的下一条指令。
  + 因此，当从异常返回时，出错（fault）会重新执行那条指令；而陷入（trap）就不会重新执行。
  + `缺页异常`（page fault），由于是fault，所以当缺页异常处理完成之后，还会去尝试重新执行那条触发异常的指令（那时多半情况是不再缺页）。
  + `陷入`的最主要的应用是在调试中，被调试的进程遇到你设置的断点，会停下来等待你的处理，等到你让其重新执行了，它当然不会再去执行已经执行过的断点指令。
  + 关于异常，还有另外一种说法叫[软件中断（software interrupt](https://blog.51cto.com/noican/1361087)），其实是一个意思。

### 2、中断向量表

+ `IDTR`寄存器指向中断向量表。

## 11. 进程间通信

## 12. 网络

## 13.内核同步机制

### 13.1 [RCU](https://www.ibm.com/developerworks/cn/linux/l-rcu/index.html)

+ `RCU(Read-Copy Update)`，是 Linux 内核实现的一种针对“读多写少”的共享数据的同步机制。不同于其他的同步机制，它允许多个读者同时访问共享数据，而且读者的性能不会受影响（“随意读”），读者与写者之间也不需要同步机制（但需要“复制后再写”），但如果存在多个写者时，在写者把更新后的“副本”覆盖到原数据时，写者与写者之间需要利用其他同步机制保证同步。

+ RCU并不是针对单个数据，而是表示当前线程处于rcu_lock状态，其它线程在`grace period`内的写操作都会做复制，在`grace period`后更新原来的值。

+ 读者在访问被RCU保护的共享数据期间不能被阻塞，这是RCU机制得以实现的一个基本前提，也就说当读者在引用被RCU保护的共享数据期间，读者所在的CPU不能发生上下文切换，spinlock和rwlock都需要这样的前提。

+ 写者在访问被RCU保护的共享数据时不需要和读者竞争任何锁，只有在有多于一个写者的情况下需要获得某种锁以与其他写者同步。

  + 写者修改数据前首先拷贝一个被修改元素的副本，然后在副本上进行修改，修改完毕后它向垃圾回收器注册一个回调函数以便在适当的时机执行真正的修改操作。等待适当时机的这一时期称为`grace period`，而CPU发生了上下文切换称为经历一个`quiescent state`，`grace period`就是所有CPU都经历一次`quiescent state`所需要的等待的时间。垃圾收集器就是在grace period之后调用写者注册的回调函数来完成真正的数据修改或数据释放操作的。

+ RCU 的一个典型的应用场景是链表，提供了利用 RCU 机制对链表进行增删查改操作的接口。

  + 写者要从链表中删除元素 B，它首先遍历该链表得到指向元素 B 的指针，然后修改元素 B 的前一个元素的 next 指针指向元素 B 的 next 指针指向的元素C，修改元素 B 的 next 指针指向的元素 C 的 prep 指针指向元素 B 的 prep指针指向的元素 A。
  + 在这期间可能有读者访问该链表，修改指针指向的操作是原子的，所以不需要同步，而元素 B 的指针并没有去修改，因为读者可能正在使用 B 元素来得到下一个或前一个元素。
  + 写者完成这些操作后注册一个回调函数以便在 `grace period` 之后删除元素 B，然后就认为已经完成删除操作。垃圾收集器在检测到所有的CPU不在引用该链表后，即所有的 CPU 已经经历了 `quiescent state`,`grace period` 已经过去后，就调用刚才写者注册的回调函数删除了元素 B。

  ![](http://ww1.sinaimg.cn/large/77451733gy1g4vnfid05lj2060052t8l.jpg)

### 13.2 seq锁

+ 写数据时更新数据的seq号，读数据时记录当前读的seq号版本，再次读时若版本不一样则重新读，适合写多的场景，相当于把重负载都丢给了读操作处理，而RCU相当于把重负载的操作都丢给写操作处理。

### 13.3 [futex](https://yq.aliyun.com/articles/6043)

> Futex按英文翻译过来就是**快速用户空间互斥体**。其设计思想其实 不难理解，在传统的Unix系统中，System V IPC(inter process communication)，如 semaphores, msgqueues, sockets还有文件锁机制(flock())等进程间同步机制都是对一个内核对象操作来完成的，这个内核对象对要同步的进程都是可见的，其提供了共享 的状态信息和原子操作。当进程间要同步的时候必须要通过系统调用(如semop())在内核中完成。可是经研究发现，很多同步是无竞争的，即某个进程进入 互斥区，到再从某个互斥区出来这段时间，常常是没有进程也要进这个互斥区或者请求同一同步变量的。但是在这种情况下，这个进程也要陷入内核去看看有没有人 和它竞争，退出的时侯还要陷入内核去看看有没有进程等待在同一同步变量上。这些不必要的系统调用(或者说内核陷入)造成了大量的性能开销。为了解决这个问 题，Futex就应运而生，Futex是一种用户态和内核态混合的同步机制。首先，同步的进程间通过mmap共享一段内存，futex变量就位于这段共享 的内存中且操作是原子的，当进程尝试进入互斥区或者退出互斥区的时候，先去查看共享内存中的futex变量，如果没有竞争发生，则只修改futex,而不 用再执行系统调用了。当通过访问futex变量告诉进程有竞争发生，则还是得执行系统调用去完成相应的处理(wait 或者 wake up)。简单的说，futex就是通过在用户态的检查，**（motivation）如果了解到没有竞争就不用陷入内核了，大大提高了low-contention时候的效率**。 Linux从2.5.7开始支持Futex。

## 14. 线程相关

> `pthread`各种东西均为`glibc`中的实现。

+ 

## 其它

### [core dump](http://happyseeker.github.io/kernel/2016/03/04/core-dump-mechanism.html)

### [进程的挂起与阻塞](https://iamxpy.github.io/2017/09/20/OS%E4%B8%AD%E9%98%BB%E5%A1%9E%E4%B8%8E%E6%8C%82%E8%B5%B7%E7%9A%84%E5%8C%BA%E5%88%AB-sleep()%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/)

## 问题

+ 内核`RCU`机制
+ `seq_lock`：写友好
+ `rcu`：读友好，更新时先拷贝，设置拷贝项的修改先，再设置自身指向的前后指针，再设置前面的指向它，再设置自己指向后面的。删除时，与更新相同，只是最后释放的时候调用`synchronize_rcu()`或`call_rcu()`在一段间隔后回收，间隔`(grace period)`的确定是等待当前时间点所有未完成的`read`操作执行完
+ `madvise`
+ 平常调用`mmap`实际调的是`glibc`的`mmap`，其对内核的系统调用`mmap`进行了封装，最终调用的是`mmap2 -> do_mmap`
+ `sys_mmap`源码应当搜索`SYSCALL_DEFINE6(mmap`，里面调用`SYSCALL_DEFINE6(mmap_pgoff`，再进入`ksys_mmap_pgoff -> vm_mmap_pgoff -> do_mmap_pgoff -> do_mmap`
+ `mmap`的最初起始位置有随机页偏移
+ 内核配置一般是没有开启`CONFIG_PREEMPT`，只开启了`CONFIG_PREEMPT_VOLUNTARY`
+ 内核的各种[中断和异常](http://guojing.me/linux-kernel-architecture/posts/interrupt/)
+ [缺页异常的处理](https://edsionte.com/techblog/archives/4174)
+ [内存碎片](http://www.wowotech.net/memory_management/memory-fragment.html)
+ `CPU`执行一条指令的过程：取指、译码、执行、访存、写回
+ [CPU指令乱序](https://zhuanlan.zhihu.com/p/45808885)：
  + cpu为了提高流水线的运行效率，会做出比如：1)对无依赖的前后指令做适当的乱序和调度；2)对控制依赖的指令做分支预测；3)对读取内存等的耗时操作，做提前预读；等等。以上总总，都会导致指令乱序的可能。
  + 但是对于x86的cpu来说，在单核视角上，其实它做出了Sequential consistency的一致性保障，指令在cpu核内部确实是乱序执行和调度的，但是它们对外表现却是顺序提交的，cpu只需要把内部真实的物理寄存器按照指令的执行顺序，顺序映射到ISA寄存器上，也就是cpu只要将结果顺序地提交到ISA寄存器，就可以保证Sequential consistency。
+ per-cpu变量和线程局部存储。
+ 信号和信号量
  + 信号，类似于软中断，如果进程接受到了信号，就相当于发生了中断，这时候进程会切换到内核态去执行信号处理函数，注意下，一个进程可能有两个执行序列，一个是main，另一个就是信号处理函数，信号处理函数有自己的栈空间的。当执行完了信号处理函数，才会再回到main函数中被抢占的代码上。不同的信号有不同的功能，比如可以让进程结束。
  + 信号量和锁，补充下。一般说来，信号量可以用于互斥和同步，锁用来互斥。你题目说他们实现的依赖关系，信号不应该是这个范围，那就说下信号量和锁。



