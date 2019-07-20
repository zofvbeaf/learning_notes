## Course

+ [CMU15740 Computer Architecture](https://www.cs.cmu.edu/afs/cs/academic/class/15740-f18/www/)
+ [CMU15-418 Parallel Computer Architecture and Programming](http://www.cs.cmu.edu/afs/cs/academic/class/15418-s19/www/index.html)

## 并行度与并行体系结构分类

+ 应用程序并行：
  + 数据级并行（`DLP`）：可以同时操作许多数据项
  + 任务级并行（`TLP`）
+ 硬件并行：
  + 指令级并行，利用流水线之类的思想开发数据级并行
  + 向量体系结构和图形处理器（GPU），将单指令应用于一个数据集，开发数据级并行
  + 线程级并行，开发任务级并行
  + 请求级并行
+ 指令流和数据流并行
  + SISD
  + SIMD：同一指令由多个使用不同数据流的处理器执行
  + MISD：目前未见
  + MIMD：

## 硬件速度

```
L1 cache reference                            0.5 ns
Branch mispredict                             5 ns
L2 cache reference                            7 ns
Mutex lock/unlock                             100 ns
Main memory reference                         100 ns
Compress 1K bytes with Zippy                  10,000 ns
Send 2K bytes over 1 Gbps network             20,000 ns
Read 1 MB sequentially from memory            250,000 ns
Round trip within same datacenter             500,000 ns
Disk seek                                     10,000,000 ns
Read 1 MB sequentially from network           10,000,000 ns
Read 1 MB sequentially from disk              30,000,000 ns
Send packet CA->Netherlands->CA               150,000,000 ns
```

## 服务器配置

+ `cache line`：64B

+ 实验室的服务器有两个`socket`，每个`socket`对应一个`Xeon E5620`，每个`socket`含有`4`个`core`，每个`core`可同时运行两个`Thread`，即服务器总共可运行`16`个线程，服务器的`NUMA`节点数也为`2`，具体`lscpu`结果如下：

```
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                16
On-line CPU(s) list:   0-15
Thread(s) per core:    2
Core(s) per socket:    4
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 44
Model name:            Intel(R) Xeon(R) CPU           E5620  @ 2.40GHz
Stepping:              2
CPU MHz:               2393.941
BogoMIPS:              4787.88
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              12288K
NUMA node0 CPU(s):     0,2,4,6,8,10,12,14
NUMA node1 CPU(s):     1,3,5,7,9,11,13,15
```

## 寄存器

> x86-64下共有16个寄存器：%rax, %rbx, %rcx, %rdx, %rdi, %rsi, %rbp, %rsp, and %r8-r15

+ 参考**《x64 cheatsheet》**，[Guide to x86-64](https://web.stanford.edu/class/archive/cs/cs107/cs107.1196/guide/x86-64.html)

+ `stack pointer`：`sp`（16位），`esp`（32位），`rsp`（64位），指向栈顶

+ `base pointer`：`rbp`（栈基址指针）,指向栈帧的基址

+ `rax`：存储函数返回值，当需要返回对象时，`rax`指向要返回对象的起始地址

+ 函数参数：`%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9`,其他的参数从右往左依次压栈。由调用者恢复栈。

	+ 浮点数通过`%xmm0-7`传递

+ 函数构造时，通过`rcx`给`this`指针赋值

	```asm
	; 一般函数调用过程
	push    rbp        
	mov     rbp, rsp   ; 保存栈基址指针， mov dst, src
	
	;  ....
	
	leave
	ret
	```

+ 其它
	+ `rip`：指令指针寄存器，指向CPU要读取指令的地址。
	+ `eflags`：[标志寄存器](https://en.wikipedia.org/wiki/FLAGS_register)，包括加减运算的仅为借位标志和中断允许标志等。

## 一些概念

+ `CPU`乱序和编译器乱序
+ `dpdk`的使用

## 博客

+ [7个示例科普CPU CACHE](https://coolshell.cn/articles/10249.html)


