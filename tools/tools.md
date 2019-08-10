## gdb

### `gcc/g++`调试信息生成

```shell
-g # 在可执行文件中加入源文件信息（但不嵌入源文件）。用操作系统原生格式生成调试信息（gdb和其他调试器都可用）
-ggdb # 为gdb生成专用的更丰富的调试信息（但就不能用其他调试器如ddx）
#级别
-g1 # 不包含局部变量和行号的调试信息（只能看看函数调用历史）
-g2 #（默认）包括扩展的符号表，行号，局部和外部变量信息
-g3 # 包含-g2的所以信息，以及源码的宏
```
### `gdb`的启动

> 对于没有调试信息的程序直接`run`是没有效果的，可以通过`readelf`获取入口地址，在入口地址打断点
>
> 如：`b *0x400440`

```shell
# 以core dump文件进行调试
gdb ./a.out core_file.pid # 完整为gdb -se ./a.out -c core_file.pid
gdb -c core_file.pid # 进入后file a.out
gdb -core=core_file.pid # 进入后file a.out
# 调试一正在运行的进程
gdb ./a.out 12345 # 12345为进程pid号。等价于gdb进入后attatch 12345
```

### `gdb`命令

+ 断点

  ```shell
  break file_name.cpp:123 # 123为行号。简写 b
  info breakpoint # 查看断点信息
  disable 3 # 使3号断点失效
  enable 5 # 使5号断点有效
  delete 5 # 删除5号断点
  delete 1-10 # 删除1-10号断点
  clear FileName.cpp:FuncName # 删除函数的所有断点
  clear 12 # 删除12行的所有断点
  ```

+ 观察点

  ```shell
  watch i==99 
  # 表达式设置断点，需要在运行到变量i被定义之后才可设置，可在定义行后用b设置断点然后watch设置
  ```

+ 行控制

  ```shell
  run #（重新）开始运行程序。简写 r
  next # 单步执行，跳过函数调用。简写 n
  nexti # 下一条汇编指令，遇到call指令不跳进去
  step # 单步执行，进入函数调用。简写 s
  stepi # 下一条汇编指令，遇到call指令则跳进去。简写 si
  finish # 继续运行至函数退出。简写 fini
  until # 退出循环体
  continue # 开始运行到下一个断点。简写 c
  jump filename:linenum # 跳到指定位置执行
  signal SIGINT # 发信号
  call funcname # 强制调用函数
  ```

+ 多线程控制

  ```shell
  thread 3 # 切换到3号线程
  break file.c:123 thread all # 在所有线程file.c:123这行打断点
  thread apply 2 3 command 
  # 在2号和3号线程上执行gdb的命令command（代表命令）。2和3可写为all，代表所有
  set scheduler-locking on # 设置为多线程下只有被调试线程运行。off（默认）所有线程运行；step在单步时候只有当被调试线程运行（除非next过一个函数）
  set non-stop off #默认为off
  # all-stop模式：当你的程序在gdb由于任何原因而停止，所有的线程都会停止
  # non-stop模式（网络编程常用）：只有当前的线程会被停止，而其他线程将会继续运行
  attach pidnum #调试进程/子进程
  deattach
  set follow-fork-mode parent #调试父进程（默认）。还可以是child，默认追踪父进程
  info inferiors 
  inferior processnum #切换进程
  # follow-fork-mode       detach-on-fork   说明
  # parent                       on          只调试主进程（GDB默认）
  # child                        on          只调试子进程
  # parent                       off         同时调试两个进程，gdb跟主进程，子进程block在fork位置
  # child                        off         同时调试两个进程，gdb跟子进程，主进程block在fork位置
  ```

+ 查看调试信息---------

  ```shell
  #栈帧信息
  bt #backtrace查看函数堆栈
  bt 4 #查看函数堆栈顶4个
  frame 5 #切换到5层
  up 4 #上移4层，默认一层
  down #下移默认的一层
  ```

+ 显示窗口设置

  ```shell
  layout # 分割窗口
  layout asm # 显示反汇编窗口
  layout # 用于分割窗口，可以一边查看代码，一边测试。主要有以下几种用法：
  layout src # 显示源代码窗口
  layout asm # 显示汇编窗口
  layout regs # 显示源代码/汇编和寄存器窗口
  layout split # 显示源代码和汇编窗口
  layout next # 显示下一个layout
  layout prev # 显示上一个layout
  Ctrl + L # 刷新窗口
  Ctrl + x + 1 # 单窗口模式，显示一个窗口
  Ctrl + x + 2 # 双窗口模式，显示两个窗口
  Ctrl + x + a # 回到传统模式，即退出layout，回到执行layout之前的调试窗口。
  ```

+ 查看数据

  ```shell
  # 查看内存
  # examine 简写 x/命令
  x/18xb 0x12345 # 打印18个值、按照16进制、每个值宽度为字节，地址为0x12345
  # 进制：t u o x -> 2 unsigned10 8 16
  # 单元宽度：b h w -> byte halfword(2B) word(4B)
  x/s 0x123456 # 按照ascii打印内存地址
  
  # 查看变量
  # 数字类型
  p/x value_or_number # 以十六进制查看变量或者一个数字
  #Vector
  print *(a_vector_object._M_impl._M_start)@a_vector_object.size() 
  # 查看vector里的元素：size
  print *(a_vector_object._M_impl._M_start)@5 # 看vector的前5个
  print a_vector_object # 同上，gdb 7以上支持
  # shared_ptr<string>
  p ((string*)a_shared_ptr)->c_str()
  ```

+ 其它

  ```shell
  # 调试double free
  # 如果用的glibc可在gdb中设置环境变量为2进行调试，会在double free处停止（不知道效果如何反正编译时-g参数也能用）
  set environment MALLOC_CHECK_ 2
  
  print *(myVector._M_impl._M_start)@myVector.size() # 打印vector
  ```

## git

+ 代理设置

  ```bash
  git config --global http.proxy # 查看是否设置代理
  git config --global --unset http.proxy # 取消设置代理
  ```

+ 撤销与回滚

  ```bash
  git checkout . # 撤销对所有已修改但未提交的文件的修改，但不包括新增的文件
  git checkout [filename]     # 撤销对指定文件的修改，[filename]为文件名
  git reset --hard [commit-hashcode]  # 版本回退
  ```

## [rpm](http://man.linuxde.net/rpm)

```shell
rpm -ivh apache-1.3.6.i386.rpm #安装软件
rpm -Uvh a_rpm_package.rpm #升级软件
rpm -e a_rpm_package.rpm #反安装？？？
rpm -qpi a_rpm_package.rpm #查看软件包详细信息
rpm -qpl a_rpm_package.rpm #查看软件包会向系统写入哪些文件
rpm -qf a_rpm_package.rpm #查询文件所属rpm包
```

## shadowsocks

+ [client](https://blog.denghaihui.com/2017/10/10/shadowsocks-polipo/)

```bash
pip install shadowsocks
yum install privoxy
vim /etc/polipo/config
###########  write this #######
logSyslog = true
logFile = /var/log/polipo/polipo.log
proxyAddress = "0.0.0.0"
proxyPort = 8123
socksParentProxy = "127.0.0.1:7070"
socksProxyType = socks5

############     end    ########
http_proxy="localhost:8123" curl ip.gs  # for test
```

## vim

+ 对文本内容排序：`:1,10!sort -n`，对第`1-10`行按数字`(-n)`大小升序排序。

## vscode

+ `Ctrl+p`打开快速命令框后输入`ext install cpptools`，安装`C/C++`扩展，再安装`c++ intellisense`。
+ 安装`MinGW`，或者直接安装`code blocks`。
+ 设置环境变量，将`/path/to/mingw/bin`加入其中。
+ `Ctrl+Shift+p`后输入`c++ edit configurations (JSON)`，打开`c_cpp_properties.json`

## [yum](https://www.cnblogs.com/sopost/p/3245477.html)

```shell
yum repolist
yum clean all
yum makecache
# -y 对安装中的所有回答使用yes
```

## 其它命令

```bash
ifconfig
ifup
ifdown
iptables
route
init
service
lsof

useradd your_user_name
passwd your_user_name
userdel -r username #删除用户并删除其工作目录
```

## sshpass和ssh

```shell
# ssh命令
# 脚本中远程执行命令（前提是允许免密登陆）
ssh root@192.168.1.123 "echo your_cmd; echo another_cmd"
ssh root@192.168.1.123 ./shell.sh # 注意此处会执行远程机器上的脚本而不是本机上的

sshpass -p 'your_password' ssh root@192.168.1.123 'echo your_remote_cmd'
#利用shell变量临时存密码
export SSHPASS='your_password'
sshpass -e ssh root@192.168.1.123 'echo your_remote_cmd'
#用文件存密码
sshpass -f password_filename ssh root@192.168.1.123 'echo your_remote_cmd'
```

## ps

```shell
ps -o pid,psr,comm #指定显示列。psr 线程/进程分配到的CPU ID 
```

## nmap

```shell
#安装
yum install nmap 
apt-get install nmap

#主机扫描
nmap 192.168.1.123 #或nmap www.baidu.com
nmap 192.168.1.123 192.168.1.124 #扫描多台。或namp 192.168.1.123,124,125
nmap 192.168.1.1-123 #扫描ip范围
nmap 192.168.1.* --exclude 192.168.1.123 #扫描子网，并排除192.168.1.123
nmap -iL ip_list.txt #扫描ip_list.txt中列出的ip地址

#扫描端口和协议设置
nmap -p 80,8080 192.168.1.123 #-p 80,8080 只扫描80和8080端口（默认TCP）
nmap -p 80-8080 192.168.1.123 #-p 80-8080 扫描80到8080端口（默认TCP）
nmap -p T:80,8080 192.168.1.123 #指定扫描TCP协议的80和8080端口
nmap -sU 53 192.168.1.123 #扫描UPD的53号端口
# -s不是单独选项，应该是set或者scan的意思？？？

#检测主机信息防火墙等服务信息
nmap -sP 192.168.1.* #扫描主机是否在线。-sP 会跳过端口检测和其他检测
nmap -sV 192.168.1.123 #扫描远程主机上运行的服务和版本号
nmap -sA 192.168.1.123 #扫描来检测主机上是否使用了任何包过滤器或者防火墙
nmap -PN 192.168.1.123 #扫描检测一个主机是否受到任何包过滤器软件或者防火墙保护

#使用TCP SYN和TCP ACK扫描。应对禁ICMP ping情况
nmap -PS 192.168.1.123 #使用TCP SYN扫描
nmap -PA 192.168.1.123 #使用TCP ACK扫描

#其他选项
-v #显示远程机器更多信息
-A #打开操作系统检测、版本检测，脚本扫描，traceroute
-O #打开操作系统检测。--osscan-guess 更加激进地预测操作系统
--iflist #列出本机主机接口和路由信息
-F #快速扫描，仅扫描所有列在nmap-services文件中的端口
-sT #用TCP SYN扫描最常用端口
-r #打开连续端口扫描
-sS #执行隐秘扫描
-sN #用TCP空扫描来愚弄防火墙？？？参考（厉害了）：http://www.voidcn.com/article/p-rdnjhxny-ws.html

```

- 端口扫描分类
  - 开放扫描：最具代表性的是TCP连接扫描，即通过三次握手建立TCP连接
  - 半开放扫描：不需要正常连接仅利用正常连接的一部分。有TCP SYN扫描，TCP ACK扫描
  - 隐秘扫描：不包括TCP握手协议任何部分，但效果不好。有TCP FIN扫描，TCP XMAS扫描，NULL扫描，UDP扫描
- 一些扫描方法描述
  - TCP ACK扫描：发送方发送ACK，无论端口开放或关闭都会收到RST，因而无法确定端口开放或关闭，但可用于扫描防火墙配置
  - TCP FIN扫描：发送方发送FIN，如果端口关闭会收到RST，而如果端口开放则报文被丢弃不会收到任何信息
  - NULL扫描：http://www.voidcn.com/article/p-rdnjhxny-ws.html
- Ref
  - 主要参考：<https://linux.cn/article-2561-1.html>

## 查看系统信息

```shell
lscpu #查看cpu和cache信息
lsblk #查看块设备分区情况
smartctl #查看硬盘型号和序列号
lspci #列出所有pci设备。显示系统中所有PCI总线设备或连接到该总线上的所有设备的工具。
lsusb #列出所有usb设备
lsmod #列出加载的内核模块
lsb_release #查看系统详细发行版
hostname #显示主机名
env #查看环境变量
last #查看用户登陆日志
dmesg #获得诸如系统架构、cpu、挂载的硬件，RAM等多个运行级别的大量的系统信息
```

- `/sys/devices/system/cpu/cpu0/cache`目录`index0～index4`四个目录，分别对应`L1d，L1i，L2，L3`信息
- `number_of_sets`：组数
  - `ways_of_associativity`：每组行数
  - `coherency_line_size`：每行字节大小

## 定时

```shell
#crontab
#统定时执行脚本：命令方式或修改配置文件/etc/crontab

#watch
watch -n 9 echo "hello" #每9s运行一次echo “hello”命令
watch -n 9 "echo \"hello\""
```

## 文件操作

#### tar

#### split

```shell
split -b 3G big_file_name prefix_of_new_small_file #切分为每个文件3G。5M或者4K都行
split -1 10 bif_file_name prefix_of_new_small_file #按行数切
#-d 参数使用数字作为后缀
```

#### sort

```shell
sort filename |uniq #排序和去重（按行处理）
```

#### sed

```shell
# 将文本中多个空格合并为一个空格
sed 's/[ ][ ]*/ /g' source_file > result_file
# 空格和tab共存时用：空格和tab一起都换成一个空格
sed -e 's/[[:space:]][[:space:]]*/ /g' source_file > result_file


# -i选项：将更改应用于原文件
sed -i 's///g' source_file
sed -i 's///g' source_file
# 更改原文件并创建source_file.bak的备份
sed -i .bak 's///g' source_file
```

- 正则表达式：<https://www.cnblogs.com/leaftime/p/3270257.html>
- <https://qianngchn.github.io/wiki/4.html>
- <http://wiki.jikexueyuan.com/project/unix/regular-expressions.html>
- <https://blog.csdn.net/fuwencaho/article/details/25536325>

#### dd

```shell
dd bs=512 count=123435 if=/dev/sda/or/file of=/dev/sdb/or/file/or/img

#在新的Linux中（也就是一般情况），dd进程接收到 SIGUSR1 信号后，会打印出已复制的容量和平均速度
kill -USR1 1234 #此处1234应为dd进程的pid。该命令会打印出dd进程的执行进度
```

#### 临时文件查看

```shell
#cat

#less和more

#head和tail

#wc
```

#### du

```shell
du -sh * #查看文件大小
```

#### awk

- 基本框架

  ```shell
  awk 'BEGIN{  } {  } END {  }' filename.txt
  ```

- 变量

  ```shell
  awk '{ print FNR ":" $0 }' filename.txt #基本打印示例。输出为行号:改该行内容
  awk '{ printf("%5d:%s/n", NR, $0) }' filname1.txt filename2.txt #格式化输出。规则类似C语言printf()
  # FNR 当前行在文件中的行号
  # NR 当前行在本次处理过程中的行号
  ```

- 调用shell命令

  ```shell
  #方法1：通过system()函数
  awk 'BEGIN { system("echo hello") }'
  awk 'BEGIN { v1="echo"; v2="hello"; system(v1" "v2) }'
  awk 'BEGIN { v1="echo"; v2="hello"; system(v1 v2) }' #错误：未定义命令
  awk 'BEGIN { v1=echo; v2=hello; system(v1" "v2) }' #错误：无任何输出
  
  #方法2：通过print函数加管道
  awk 'BEGIN { print "echo","hello" | "/bin/bash" }' #另：逗号会被转为一个空格
  ```

- 打印双引号和单引号

  ```shell
  #双引号
  awk 'BEGIN { print " \" " }' #awk中打印常量都需要用" "括起来
  #单引号
  awk 'BEGIN { print " ' \' ' " }' #因为\'先被bash解析成'，还需要' '括起来
  ```

#### 硬盘操作

```shell
fdisk /dev/sda #硬盘分区，交互式命令

mkfs.ext3 /dev/sda1 #格式化成ext3
mkfs.ext2 /dev/sda2 #mkfs.fat

mount /dev/sda1 /a/directory/path/ #挂载影响
unmount /dev/sda1

dd if=ubuntu-16.04-desktop-amd64.iso of=/dev/your_usb_disk #制作启动盘。此时U盘不应该被挂载
#恢复为正常盘
dd count=1 bs=512 if=/dev/zero of=/dev/your_usb_disk #MBR清零
fdisk /dev/your_usb_disk
mkfs.fat /dev/your_usb_disk
```

## 文件传输

#### scp

#### rsync

- 文件同步以及增加量更新

<http://man.linuxde.net/rsync>

```shell
rsync -avuz username@0.0.0.0::modulename /local/dst/path -password-file=/password/file/path
# a：归档模式。递归方式传输文件并保持文件的所有属性
# v：详细模式输出
# u：仅进行更新
# z：传输中压缩文件
```

- 在Linux下，rsync服务器端的secrets file和客户端的passworld file都要求400属性才能成功

#### nc

```shell
#聊天服务器/传文件
nc -l 12345 #服务器端。-l 指定监听端口12345
nc 192.168.1.123 12345 #客户端，TCP连接192.168.1.123:12345
#服务器端和客户端的标准输入会输出到彼此的标准输出。因而可以通过重定向符传送文件。
nc -l 12345 < file_name.txt #或用管道加echo或者cat等方式也能发送
nc 192.168.1.123 12345 > file_name.txt

#远程执行shell（应对没有ssh情况）
nc -l 12345 -e /bin/bash #服务器端。-c 也行
nc 192.168.1.123 12345 #客户端登陆
#反向shell：服务器使用客户端的shell，用于应对防火墙禁止接入的情况
nc -l 12345 #服务器端
nc 192.168.1.123 12345 -e /bin/bash #客户端

#端口扫描
nc -z -v -n 192.168.1.123 1-1024 #扫描192.168.1.12的1~1024端口。也可只指定一个端口号
#-z 端口的扫描模式，即零I/O模式；-v 显示详细信息，-vv 显示更详细信息
#-n 使用纯数字IP地址，即不用DNS解析IP地址

#其他参数
-k #强制服务器端不会因客户端断开而退出
-w 10 #客户端在10秒后断开连接。对服务器端无效
-q 10 #客户端收到EOF后等待10秒再退出
-d #禁止从标准输入中读取数据
-4 #使用IPv4。-6 使用IPv6
-u #使用UDP协议
```

## 磁盘操作

#### smartctl

```shell
smartctl /dev/sda -a #查看硬盘信息，包括产品名、序列号等
```

#### lsblk

- 查看所有块设备及容量和设备号等容量信息

## 编辑

#### tmux

```shell
#安装
#在CentOS 6.10 Final下如下方法可行
#https://blog.csdn.net/dw_java08/article/details/77955502
#实际上在安装时我没有安装libevent，因为系统中有，但make时编译提示找不到libevent的定义因而通过如下方法解决
#https://blog.csdn.net/weixin_39845407/article/details/84532528
#解决办法中下边的内容必须在一行内写完。其中/usr/local/lib是已有的libevent库所在目录：
#/usr/local/lib/libevent-2.0.so.5
CFLAGS="-I/usr/local/include" LDFLAGS="-L//usr/local/lib"  ./configure
make
make install
```

```shell
tmux -2 #强制tmux支持256色并启动（不开此支持可能导致vim上主题颜色异常）
tmux new -s session_name #创建新的session并用-s指定名字（可不指定）
tmux ls #列出所有session。在tmux中是ctrl+b后s
tmux a -t session_name #连接session_name代表的session。可不加-t及之后的参数
tmux detach #断开session。在tmux中可以是ctrl+b后d
tmux kill-session -t session_name #关闭session_name代表的session
tmux kill-server #应该是关闭所有session吧？？？
```

```shell
#tmux内命令
#以下默认为先按前缀组合键之后再按如下命令（默认前缀组合键为ctrl+b）

#关于session
s #列出当前session
$ #重命名当前session
d #断开session

#关于window
c #创建。或者在进入tmux后输入tmux new-window（不在tmux中也会创建新window但应该是在第一个session中创建吧）
& #关闭。或命令中直接输入exit
3 #选择第3号窗口。p 前一个窗口；n 后一个窗口
w #列出所有窗口（包括其他session的）。j和k 向前后选择
, #重命名当前窗口
f #搜索窗口

#关于pane
% #左右切分
" #"(这是没用的双引号)上下切分
x #关闭
o #切换。ctrl+b之按上下左右键也行
ctrl+o #交换两个pane
q #显示窗格编号。然后立即输入数字进行切换
! #将当前窗格切换到新窗口
#一直按着ctrl+b然后按上下左右键可调整pane大小
z #tmux版本>=1.8时，最大化当前pane，再次按可恢复。小于1.8时参考http://superuser.com/a/357799

```

```shell
#.tmux.conf配置文件（在~目录下）
#配置更改后重新加载配置用ctrl+b后按:并输入source ~/.tmux.conf

#更改命令前缀为ctrl+a（我未使用）
unbind C-b
set -g prefix C-a

setw -g mouse-select-window on #打开用鼠标选择切换window
setw -g mouse-select-pane on #打开用鼠标选择切换pane
setw -g mouse-resize-pane on #打开用鼠标拖动更改pane大小
setw -g mode-mouse on #打开鼠标支持（可用鼠标滚轮显示窗口内容，可用鼠标选取文本）。
#上述最后以想设置后，复制时能在tmux中用ctrl+b后p粘贴复制的内。传统的用鼠标选中右键复制粘贴等类似功能，需要按住shift在进行操作，将windows的内容粘贴到tmux中时使用shift+insert

set-option -g allow-rename off #禁止自动rename window名。因为设置上述关于鼠标规则后，重命名后使用鼠标会恢复之前的默认命名
```

- 自定义配置：http://note4code.com/2016/07/03/tmux-%E8%87%AA%E5%AE%9A%E4%B9%89%E9%85%8D%E7%BD%AE/

#### vim

- 命令模式（normal）

```shell
#替换命令
:9,19s/old/new/g #在9到19行内，将old字符串替换为new字符串
# % 整个文件范围； .,.+9 当前行开始的9行内； 9,$ 9行到文件结尾； /FROM/,/;/ SQL语句中from至分号部分中
#查找字符串个数
:%s/your_string/&/gn #查找当前文件中your_string字符串出现的次数。& 在正则表达式中代表已经匹配的串。:%s/your_string//ng 效果相同，但不安全不推荐

#切换到文本输入模式的命令键
i #在光标左侧输入正文
a #在光标右侧输入正文
I #在光标所在行开头输入正文
A #在光标所在行末尾输入正文
o #在光标所在行下边增加新的一行
O #在光标所在行上边增加新的一行
R #替换从光标位置开始的字符（仅限当前行内）

#查找
fa #光标跳到本行第一个字符a处，跳到bcd...为fb，fc，fd...

#替换与修改
r #替换当前光标上的字符，为紧接着输入的字符
x #删除当前光标上的字符

gU3 #当前及往下两行字符转为大写，按Enter应用修改。
#gUi" 将本行中第一对双引号中内容替换为大写，gUi' 为单引号中内容
gu3 #当前及往下两行字符转为小写，按Enter应用修改。
g~3 #当前及往下两行字符大小写互转，按Enter应用修改。

#撤销和恢复
u #撤销最近的修改（不包括u命令本身）
ctrl-r #恢复到最新更改，可用于不小心用u回撤
U #撤销对当前行的所有修改

#高亮单词
gd #高亮当前光标所在单词，下次就可以根据该单词直接查找
shift-* #高亮当前光标所在单词并向下查找。 # 号为向上查找

#窗口
:set mouse=a #所有（all）状态下都可以使用鼠标
crtl-w后H #窗口向左最大贴边（和Windows的移到边缘最大化一样）显示，可实现窗口交换。JKL 下上右

#设置（set）命令
:set all #查看所有vim中环境变量的值
:set invlist #显示不可见字符。 $ 换行符； ^I 制表符。 :%!cat -A 和shell下 cat -A filename 功能相同
:set nolist #关闭显示不可见字符
:set syntax=cpp #设置当前文件语法高亮方式为c++，其他的还有php，error等

:f #显示当前文件名字（包括路径）。或者ctrl-g

q: #展示命令历史，按enter或者输入:q退出

#不知道是否和插件相关的命令
gf #光标指向#include则，在新的tab中跳转到该头文件。库文件也管用
:AT #在新tab中打开该cpp文件的头文件
`` #在同一个文件中通过gg跳转后回到上一个位置，多次跳转后会在这两个位置间来回跳
#其实后一个`是内部标记，代表上一次位置（也可用单引号'）。" 上次编辑该文件的位置；
mt #marks命令。在当前位置定义名为t的标记mark（可跨文件），t可为其他单个字符。可用`t跳转到此处
:marks #显示所有标记

:mksesssion filename.vim #保存当前会话状态。filename.vim不指定时默认为Session.vim。用vim -S filename.vim恢复
#一般包括打开文件位置、tab等。可通过SESSIONOPTION这vim环境变量控制。

#打开多个文件：此处仅仅指用vi filenam1 filename2打开多个文件
#此功能很没用。:ar命令仅仅会显示命令行下打开的文件列表。并且在:n切换下一个文件时候需要保存当前文件（意味着其逻辑是在没有tab，没有vs分屏情况下只会存在一个文件的.swp临时文件，类似于一个个打开文件编辑）（tab算两个独立的vim）
:ar #查看打开文件列表
:n #切换到下一个文件。:n! 不保存当前文件并切换
:N #切换到上一个文件

#文件读写
:r filename #将filename文件的内容读写到光标之后
:r ! ls #执行ls命令并将结果保存到光标之后
:10,110 w filename #10到110行保存到另一个文件，w!表示如果存在就将其覆盖


#文件显示
:set invlist #（恢复 :set nolist） ：显示不可见字符：$换行符、^I制表符。也可 :%!cat -A 或者shell下 cat -A filename
:%!xxd #文本以十六进制显示。返回是:%!xxd -r。详情在shell下执行xxl --help

### 缓冲区
#数字缓冲区1~9：每次删除和复制的内容都会自动放到1号，更旧的就放到2号，依次操作
#26个字母缓冲区a~z：给用户自己用，需要在删除复制时候自己指定
#"wdd**和"w19yy 删除和复制19行并保存到w缓冲区
#"9p和"zP把9号缓冲区内容粘贴到光标下和把z缓冲区内容粘贴到光标上边

#寄存器：缓冲区也是寄存器
#https://harttle.land/2016/07/25/vim-registers.html
```

- 可视（visual）模式

```shell
#进入：在命令模式下
v #进入字符可视模式（从命令模式）
V #进入行可视模式（从命令模式）
ctrl-v #进入块可视模式（从命令模式）
#在复制，删除等操作完之后会自动回到命令模式，修改操作需按Esc键生效后回到命令模式，无修改需按两次Esc回到命令模式

#选择：不会回到命令模式
i" #选中" "中的内容。命令模式下也可用
gv #重新选中之前可视模式选中的区域
#可视模式选中区域由两个端点界定
o #切换到左上角/右下角端点以调整两个端点位置

#操作：可视模式和命令模式都可用的命令，操作完回到命令模式
r #替换，被选中部分的每个字符都被替换，为紧接着输入的字符。可用于批量注释/解除注释代码
d #删除，块模式下由各行尾部字符串左移替换被删除部分。可用于批量解除注释代码
c #修改，将每行选中部分修改为相同内容（为行/块模式时）。先自动删除后进入输入模式，输入完成后需按Esc应用更改效果
I #在选中文本前插入。进入输入模式，输入完成后需按Esc应用更改效果。可用于批量注释代码
A #在选中文本后插入。进入输入模式，输入完成后需按Esc应用更改效果

gU #选中区域转为大写。命令模式下
gu #选中区域转为小写
g~ #选中区域大小写互换
```

- tab相关的命令

```shell
#tab操作
:tabnew filename #在新tab打开文件filename
:tabs #查看所有的tab。> 表示当前页面；+ 表示已更改
:tabp #跳到前一个tab
:tabn #跳到后一个tab，可用:tabnext 3来跳3次
:tabfirst #或者tabr，跳到第一个tab
:tablast #跳到最后一个tab
:tabc #关闭当前tab，全称:tabclose
:tabo #关闭其他所有tab，全称:tabonly
:tabm 3 #将当前tab移动到第3个位置处（位置从0开始标号）

gt #跳到后一个tab
gT #跳到前一个tab

set showtabline=1 #设置为有多个标签页时才在顶部显示标签栏。2 永久显示；0 永不显示
set tabpagemax=10 #设置最大打开标签数为10（默认是10）

:tabdo vim_command #在多个标签页中执行vim_command命令，如替换等
:tab split #在新标签页中打开当前标签页文件（会保留当年标签页）
#tips：在当前标签页分屏时，可用ctrl+w后T来关闭当前窗口并在新的标签页中打开该文件
```

#### git

#### 其他

```shell
jobs
fg
```

## 开发调试

#### g++

```shell
#编译流程
# 生成预处理后的文件：.i
g++ -E test.cpp > test.i
g++ -E test.cpp -o test.i
cpp test.cpp > test.i
# 生成汇编语言：会生成.s文件
g++ -S test.cpp
# 生成目标文件（机器代码）：.o文件
g++ -c test.cpp

#设置代码优化等级
-O0 #关闭所有优化选项（默认等级）
-O1 #基本的优化等级，不花太多编译时间生成更小更快代码
-O2 #推荐的等级，比-O1多些标记，试图提高性能而不增加体积和大量编译时间
-O3 #最高最危险的等级，延长编译时间，更大等级，大大增加编译失败机会和不可预知行为
-Os #优化代码尺寸，不推荐

#其他
-static #静态方式编译可执行文件
-fno-elide-constructors #关闭返回值优化
-x language filename #按照language编译而不管后缀名language：c,objective-c,c-header, c++,cpp-output, assembler,assembler-with-cpp。相反-x none filename

#动态链接库的系统配置和原理=========
#基本方法：/etc/ld.so.conf指定动态库搜索路径，修改完该文件后用ldconfig命令使修改生效
#编译器搜索动态库的先后顺序：
#    编译时指定的动态库搜索路径
#    LD_LIBRARY_PATH环境变量（似乎运行时也会搜索）
#    /etc/ld.so.cache
#    默认/lib，之后/usr/lib
#原理：
#    ld.so.conf文件包含所有动态库目录的清单（/lib和/usr/lib除外，会被自动加载），系统据此查找共享库
#    ldconfig命令将ls.so.conf中的内容转换到ld.so.cache中之后才能被动态装入器（dynamic loader）看到
ldconfig -p #查看所有共享库
export LD_LIBRARY_PATH="/usr/lib/old:/opt/lib"


#g++警告信息设置
-Wunused-variable #打开未使用变量的警告
-Wunused-but-set-variable #打开未使用变量的警告（不知道对不对）
-Wunused-but-set-variable #关闭未使用变量的警告（不知道对不读）

strings /lib64/libc.so.6 |grep GLIBC_ #查看系统glibc支持的版本
strings filename #print the strings of printable characters in files.
```

#### tcpdump

```shell
#tcpdump命令选项
#监听网络接口设置
-D #列出所有可用的网络接口选项
-i any #设置为监听所有网络接口。eth0 监听名为eth0的端口
-n #设置为不解析主机名字。-nn 不解析主机名和端口名
#监听包设置
-c123 #获取到指定数目（123）的数据包后就停止
-s123 #定义snaplength(size)；-s0 表示获取全部。（意思应该是每个包监听大小？？？）
-e #获取 ethernet 头信息
-E #通过提供 key 来解密 IPSEC 流量（不知道干嘛的？？？）
#显示设置
-q #输出较少信息
-S #输出绝对序列号
-t #设置一种更便于阅读的时间戳输出。-tttt 最便于阅读的时间戳输出；可为1到5个t
-X #以HEX和ASCII模式输出数据包的内容。-XX 与-X选项相同，同时还输出ethernet头
-v #输出更多数据包的信息。-vv ；-vvv
#写入和读取文件
-w filename.pcap #写入到文件filename.pcap中（wireshark支持.pcap）
-r filename.pcap #读入文件filename.pcap进行分析

#tcpdump表达式选项
# 三种类型的表达式选项：
# 1.Type：host，net，port
# 2.Direction：src，dst及其组合
# 3.Protocol：tcp，udp，ICMP，ah等

#用法示例
tcpdump -nnvXSs 0 -c1 icmp #指定各种参数的写法和icmp协议
tcpdump net 10.2.64.0/24 #针对子网过滤
tcpdump portrange 21-23 #端口范围过滤
tcpdump less 32 #包大小过滤。tcpdump greater 64 ；tcpdump <= 128

#tcpdump高级功能=========

#逻辑运算符
#与：and，&
#或：or，||
#非（EXCEPT）：not，!
#使用括号要用\转义
tcpdump src 192.168.56.1 and not dst port 22 #指定与和非条件
#tcpdump host 210.27.48.1 and \(210.27.48.2 or 210.27.48.3 \) #使用括号
#tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'
#tcpdump 'gateway snup and (port ftp or ftp-data)'
#tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
#tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'
#tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'

#指定TCP标志位
#特殊指定方式：指定syn包和fin包
tcpdump 'tcp[tcpflags] == tcp-syn'
tcpdump 'tcp[tcpflags] == tcp-fin'
#一般指定方式
# tcpdump 'tcp[13] & 32!=0' 所有 URGENT (URG) 包
# tcpdump 'tcp[13] & 16!=0' 所有 ACKNOWLEDGE (ACK) 包
# tcpdump 'tcp[13] & 8!=0' 所有 PUSH (PSH) 包
# tcpdump 'tcp[13] & 4!=0' 所有 RESET (RST) 包
# tcpdump 'tcp[13] & 2!=0' 所有 SYNCHRONIZE (SYN) 包
# tcpdump 'tcp[13] & 1!=0' 所有 FINISH (FIN) 包
# tcpdump 'tcp[13]=18' 所有 SYNCHRONIZE/ACKNOWLEDGE (SYNACK) 包
#一些特殊的用法
# tcpdump 'tcp[13] = 6' RST 和 SYN 同时启用的数据包（不正常）
# tcpdump 'tcp[32:4] = 0x47455420' 获取 http GET 请求的文本
# tcpdump 'tcp[(tcp[12]>>2):4] = 0x5353482D' 获取任何端口的 ssh 连接（通过 banner 信息）
# tcpdump 'ip[8] < 10' ttl 小于 10 的数据包（出现问题或 traceroute 命令）
# tcpdump 'ip[6] & 128 != 0' 非常有可能是黑客入侵的情况

```



## shell脚本编程

- **sh（全称 Bourne Shell）** 是UNIX最初使用的shell，而且在每种UNIX上都可以使用。 Bourne Shell在shell编程方面相当优秀，但在处理与用户的交互方面做得不如其他几种 shell。
- **bash（全称 Bourne Again Shell）** LinuxOS默认的，它是Bourne Shell的扩展。 与Bourne Shell完全兼容，并且在Bourne Shell的基础上增加了很多特性。可以提供命令补全，命令编辑和命令历史等功能。它还包含了很多C Shell和Korn Shell 中的优点，有灵活和强大的编辑接口，同时又很友好的用户界面。
- **csh（全称 C Shell）** 是一种比Bourne Shell更适合的变种Shell，它的语法与C语言很相似。
- **Tcsh** 是Linux提供的C Shell的一个扩展版本。Tcsh包括命令行编辑，可编程单词补全，拼写校正，历史命令替换，作业控制和类似C语言的语法，他不仅和Bash Shell提示符兼容，而且还提供比Bash Shell更多的提示符参数。
- **ksh（全称 Korn Shell）** 集合了C Shell和Bourne Shell的优点并且和Bourne Shell完全兼容。
- **pdksh** 是LinuxOS提供的ksh的扩展。pdksh支持人物控制，可以在命令行上挂起，后台执行，唤醒或终止程序。



| $$          | $0                 | $n   | $#                   | $*       | $@                           | $?                  |
| ----------- | ------------------ | ---- | -------------------- | -------- | ---------------------------- | ------------------- |
| 脚本进程PID | 脚本名（根据输入） | 参数 | 参数个数（$1开始算） | 所有参数 | 所有参数（引号包含参数合并） | 上个命令退出状态-数 |

```shell
for variable
in list-of-value
do
    #循环体
done

while condition_expr #while true/false 或其他
do
    #循环体
done
while [ condition ] #condition和[]有空格（必须？）
do
    #循环体
done

if [ condition1 ] #condition和[]有空格（必须？）
then
    #代码段1
elif [ condition2 ] #该部分可省略
then
    #代码段2
else #该部分可省略
    #代码段3
fi

#注释方法。也可用于输出到其他shell
:<<SELF_DIFINE_ARBITARY_NAME
Code here is invalid
SELF_DIFINE_ARBITARY_NAME

#算术运算
#常量相加
expr 2 + 1 #运算符左右要空格，常量计算必须是整数
expr 2 \* 1 #*号需要转义
expr 2 \% 1
#变量相加
x=10
x=`expr $x + 1`
#let方法
x=10
let x=x+1 #不能有空格

#test命令
test num1 -eq num2 #num1和num2是否相等
test str1 = str2 #串str1和str2是否相等
test -n str1 #串长度是否非0（\0？）

#数值：-eq、-ne、-gt、-ge、-lt、-le
#字符串：=、!=、-n str1（长度非零）、-z str1（长度为零）
#文件：-r file1（存在且可读）、-w、-s（存在且长度非零）、-d（存在且是否目录）、-f（存在且普通文件）
#shell提供了一种调用test的方法：用方括号调用test命令 注意括号内两边有空格
#与或非：-a -o !
```

## 其他

```shell
#某个进程的网络收发情况
#iftop/iptraf看端口netstat –plan看进程

#某个进程的端口号情况
#netstat

#某个进程的磁盘读写情况
#iotop看进程对磁盘读写
#lsof和stap
lsof #查看进程打开文件
stap inodewatch.stp major minor inode      # 主设备号, 辅设备号, 文件inode节点号
stap inodewatch.stp  0xfd 0x00 523170    # 主设备号, 辅设备号, inode号,可以通过 stat 命令获得

#某个进程打开的文件
#lsof

#某个进程的CPU运行情况
#top
#pidstat

# vim
^@替换：
:%s/\%x00/ /g
^A替换：
:%s/\%x01/ /g
^M替换：
:%s/\%x02/ /g
```

### vim-godef

```bash
# bash
go get -v github.com/rogpeppe/godef
go install -v github.com/rogpeppe/godef

# ~/.vimrc
Plug 'dgryski/vim-godef'

# ~/.vim/bundle/vim-godef/plugin/godef.vim
autocmd FileType go nnoremap <buffer> gd :call GodefUnderCursor()<cr>
autocmd FileType go nnoremap <buffer> <C-]> :call GodefUnderCursor()<cr>
```

