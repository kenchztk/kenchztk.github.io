---
layout: post
title:  Linux 常用指令
date:   2022-11-23 13:11:11
categories: linux
tags: linux
---

* content
{:toc}

### **top** 
可以实时动态地查看系统的整体运行情况，是一个综合了多方信息监测系统性能和运行信息的实用工具。通过top命令所提供的互动式界面，用热键可以管理。

- 表头含义：

PID | USER | PR | VIRT | RES | SHR |
 - | - | - | - | - | - | - | 
进程id | 该进程对应的用户 | 优先级 | 虚拟内存 | 常驻内存 | 共享内存

计算一个进程实际使用的内存 = 常驻内存（RES）- 共享内存（SHR）

S | %CPU | %MEM | TIME+ | COMMAND | 
 - | - | - | - | - | - | - | 
表示进程的状态status（sleeping，其中S表示睡眠，R表示运行） | 表示CPU的占用百分比 | 表示内存的占用百分比 | 执行的时间 | 进程的名称


- 选项
```
-b：以批处理模式操作；
-c：显示完整的治命令；
-d：屏幕刷新间隔时间；
-I：忽略失效过程；
-s：保密模式；
-S：累积模式；
-i<时间>：设置间隔时间；
-u<用户名>：指定用户名；
-p<进程号>：指定进程；
-n<次数>：循环显示的次数。
 ```

- 交互命令
``` 
h：显示帮助画面，给出一些简短的命令总结说明；
k：终止一个进程；
i：忽略闲置和僵死进程，这是一个开关式命令；
q：退出程序；
r：重新安排一个进程的优先级别；
S：切换到累计模式；
s：改变两次刷新之间的延迟时间（单位为s），如果有小数，就换算成ms。输入0值则系统将不断刷新，默认值是5s；
f或者F：从当前显示中添加或者删除项目；
o或者O：改变显示项目的顺序；
l：切换显示平均负载和启动时间信息；
m：切换显示内存信息；
t：切换显示进程和CPU状态信息；
c：切换显示命令名称和完整命令行；
M：根据驻留内存大小进行排序；
P：根据CPU使用百分比大小进行排序；
T：根据时间/累计时间进行排序；
w：将当前设置写入~/.toprc文件中。
 ```

### **ps**
报告当前系统的进程状态

- 选项
``` 
a：显示一个终端的所有进程，除会话引线外；
u：显示进程的归属用户及内存的使用情况；
x：显示没有控制终端的进程；
-l：长格式显示更加详细的信息；
-e：显示所有进程；
-f：显示UID,PPIP,C与STIME栏位。
f：用ASCII字符显示树状结构，表达程序间的相互关系。
 ```

- 实例
```
ps aux | sort -rnk 4 # 按内存资源的使用量对进程进行排序
ps aux | sort -nk 3  # 按 CPU 资源的使用量对进程进行排序
ps -efL # 查看线程数
ps -e -o "%C : %p :%z : %a"|sort -k5 -nr # 查看进程并按内存使用大小排列
ps -ef | grep ssh # ps 与grep 常用组合用法，查找特定进程
ps -C nginx # 通过名字或命令搜索进程
ps aux --sort=-pcpu,+pmem # CPU或者内存进行排序,-降序，+升序
ps -f --forest -C nginx # 用树的风格显示进程的层次关系
ps -o pid,uname,comm -C nginx # 显示一个父进程的子进程
ps -aux | grep named # 查看named进程详细信息
 ```
### **ss**
netstat 替代

- 查看所有 TCP 连接：``ss -t⁠⁣ ``
- 查看所有 UDP 连接：``ss -u⁠⁣ ⁠⁣``
- 查看所有活动连接（含监听）：``ss -a⁠⁣ ``
- 查看监听端口：``ss -l`` 若需同时查看 TCP 和 UDP 监听端口并以数字形式显示，推荐使用 ``ss -tuln⁠⁣ ⁠``
- 显示进程信息：使用 ``-p`` 选项可显示占用端口的进程 ID 和名称（通常需要 root 权限），例如 `` ss -tulnp⁠ ``
- 查看详细 TCP 状态：使用 ``-i`` 或 ``-e`` 选项可显示扩展信息，如计时器、内存使用和拥塞窗口等，命令为 ``ss -ti`` 或 `` ss -te⁠⁣ ⁠⁣``
- 查看连接统计摘要：使用 ``ss -s`` 可快速输出当前系统的连接统计概况


### **netstat**
用来打印Linux中网络系统的状态信息，可让你得知整个Linux系统的网络情况。

- 语法

```
-n或--numeric：直接使用ip地址，而不通过域名服务器；
-p或--programs：显示发起连接的进程pid和进程名称；
-l或--listening：显示状态为 LISTEN（监听）的连接；
-t或--tcp：显示TCP传输协议的连线状况；
-u或--udp：显示UDP传输协议的连线状况；

-a或--all：显示所有连线中的Socket；
-A<网络类型>或--<网络类型>：列出该网络类型连线中的相关地址；
-c或--continuous：持续列出网络状态；
-C或--cache：显示路由器配置的快取信息；
-e或--extend：显示网络其他相关信息；
-F或--fib：显示FIB；
-g或--groups：显示多重广播功能群组组员名单；
-h或--help：在线帮助；
-i或--interfaces：显示网络界面信息表单；
-M或--masquerade：显示伪装的网络连线；
-N或--netlink或--symbolic：显示网络硬件外围设备的符号连接名称；
-o或--timers：显示计时器；
-r或--route：显示Routing Table；
-s或--statistice：显示网络工作信息统计表；
-v或--verbose：显示指令执行过程；
-V或--version：显示版本信息；
-w或--raw：显示RAW传输协议的连线状况；
-x或--unix：此参数的效果和指定"-A unix"参数相同；
--ip或--inet：此参数的效果和指定"-A inet"参数相同。
 ```

- 实例

列出所有端口 (包括监听和未监听的)
```
netstat -a     #列出所有端口
netstat -at    #列出所有tcp端口
netstat -au    #列出所有udp端口
 ```

在netstat输出中不显示主机，端口和用户名(host, port or user)
当你不想让主机，端口和用户名显示，使用netstat -n。将会使用数字代替那些名称。同样可以加速输出，因为不用进行比对查询。

```netstat -an```

如果只是不想让这三个名称中的一个被显示，使用以下命令:
 ```
netsat -a --numeric-ports
netsat -a --numeric-hosts
netsat -a --numeric-users
 ```

找出程序运行的端口

并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。
```netstat -ap | grep ssh```

找出运行在指定端口的进程：
```netstat -an | grep ':80'```

通过端口找进程ID
```netstat -anp|grep 8081 | grep LISTEN|awk '{printf $7}'|cut -d/ -f1```


### **pmap**
``` shell
[root@pdai-centos ~]# pmap -h

Usage:
 pmap [options] PID [PID ...]

Options:
 -x, --extended              show details
 -X                          show even more details
            WARNING: format changes according to /proc/PID/smaps
 -XX                         show everything the kernel provides
 -c, --read-rc               read the default rc
 -C, --read-rc-from=<file>   read the rc from file
 -n, --create-rc             create new default rc
 -N, --create-rc-to=<file>   create new rc to file
            NOTE: pid arguments are not allowed with -n, -N
 -d, --device                show the device format
 -q, --quiet                 do not display header and footer
 -p, --show-path             show path in the mapping
 -A, --range=<low>[,<high>]  limit results to the given range

 -h, --help     display this help and exit
 -V, --version  output version information and exit

For more details see pmap(1).
```

``` shell
xxxx@xxxx:~$ sudo pmap -p 699453 | grep info
00007f8bb14ea000    372K r-x-- /opt/trdlp/TrDlpEngineSrv/libs_x64/libtrrinfo.so
00007f8bb1547000   2048K ----- /opt/trdlp/TrDlpEngineSrv/libs_x64/libtrrinfo.so
00007f8bb1747000     16K rw--- /opt/trdlp/TrDlpEngineSrv/libs_x64/libtrrinfo.so
```

### **lsof**
``` shell
-a 列出打开文件存在的进程
-c<进程名> 列出指定进程所打开的文件
-g 列出GID号进程详情
-d<文件号> 列出占用该文件号的进程
+d<目录> 列出目录下被打开的文件
+D<目录> 递归列出目录下被打开的文件
-n<目录> 列出使用NFS的文件
-i<条件> 列出符合条件的进程。（4、6、协议、:端口、 @ip ）
-p<进程号> 列出指定进程号所打开的文件
-u 列出UID号进程详情
-h 显示帮助信息
-v 显示版本信息
```
- 实例1：无任何参数
``` shell
$lsof| more
COMMAND     PID      USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
init          1      root  cwd       DIR              253,0     4096          2 /
init          1      root  rtd       DIR              253,0     4096          2 /
init          1      root  txt       REG              253,0   150352    1310795 /sbin/init
init          1      root  mem       REG              253,0    65928    5505054 /lib64/libnss_files-2.12.so
init          1      root  mem       REG              253,0  1918016    5521405 /lib64/libc-2.12.so
init          1      root  mem       REG              253,0    93224    5521440 /lib64/libgcc_s-4.4.6-20120305.so.1
init          1      root  mem       REG              253,0    47064    5521407 /lib64/librt-2.12.so
init          1      root  mem       REG              253,0   145720    5521406 /lib64/libpthread-2.12.so
...
```

- 实例2：查找某个文件相关的进程
``` shell
$lsof /bin/bash
COMMAND     PID USER  FD   TYPE DEVICE SIZE/OFF    NODE NAME
mysqld_sa  2169 root txt    REG  253,0   938736 4587562 /bin/bash
ksmtuned   2334 root txt    REG  253,0   938736 4587562 /bin/bash
bash      20121 root txt    REG  253,0   938736 4587562 /bin/bash
```
- 实例3：列出某个用户打开的文件信息
``` shell
$lsof -u username

-u 选项，u是user的缩写
```

- 实例4：列出某个程序进程所打开的文件信息
``` shell
$lsof -c mysql
-c 选项将会列出所有以mysql这个进程开头的程序的文件，其实你也可以写成 lsof | grep mysql, 但是第一种方法明显比第二种方法要少打几个字符；
 ```

- 实例5：列出某个用户以及某个进程所打开的文件信息
``` shell
$lsof  -u test -c mysql
```
- 实例6：通过某个进程号显示该进程打开的文件
``` shell
$lsof -p 11968
```
- 实例7：列出所有的网络连接
``` shell
$lsof -i
```
- 实例8：列出所有tcp 网络连接信息

``` shell
$lsof -i tcp

$lsof -n -i tcp
COMMAND     PID  USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
svnserve  11552 weber    3u  IPv4 3799399      0t0  TCP *:svn (LISTEN)
redis-ser 25501 weber    4u  IPv4  113150      0t0  TCP 127.0.0.1:6379 (LISTEN)
 ```

- 实例9：列出谁在使用某个端口
`` $lsof -i :3306 ``

- 实例10：列出某个用户的所有活跃的网络端口
`` $lsof -a -u test -i ``

- 实例11：根据文件描述列出对应的文件信息

``` shell
$lsof -d description(like 2)
示例:

$lsof -d 3 | grep PARSER1
tail      6499 tde    3r   REG    253,3   4514722     417798 /opt/applog/open/log/HOSTPARSER1_ERROR_141217.log.001
```

- 实例12：列出被进程号为1234的进程所打开的所有IPV4 network files

``` shell
$lsof -i 4 -a -p 1234
```

- 实例13：列出目前连接主机nf5260i5-td上端口为：20，21，80相关的所有文件信息，且每隔3秒重复执行

``` shell
lsof -i @nf5260i5-td:20,21,80 -r 3
```


### readelf
这个工具和objdump命令提供的功能类似，但是它显示的信息更为具体，并且它不依赖BFD库(BFD库是一个GNU项目，它的目标就是希望通过一种统一的接口来处理不同的目标文件）；

**ELF文件类型**
>ELF(Executable and Linking Format)是一种对象文件的格式，用于定义不同类型的对象文件(Object files)中都放了什么东西、以及都以什么样的格式去放这些东西。它自最早在 System V 系统上出现后，被 xNIX 世界所广泛接受，作为缺省的二进制文件格式来使用。可以说，ELF是构成众多xNIX系统的基础之一。

ELF文件有三种类型：

- 可重定位的对象文件(Relocatable file)
由汇编器汇编生成的 .o 文件
- 可执行的对象文件(Executable file)
可执行应用程序
- 可被共享的对象文件(Shared object file)
动态库文件，也即 .so 文件

``` shell
.text section 里装载了可执行代码；
.data section 里面装载了被初始化的数据；
.bss section 里面装载了未被初始化的数据；
以 .rec 打头的 sections 里面装载了重定位条目；
.symtab 或者 .dynsym section 里面装载了符号信息；
.strtab 或者 .dynstr section 里面装载了字符串信息；
```
**参数说明**
``` shell
-a –all 全部 Equivalent to: -h -l -S -s -r -d -V -A -I
-h –file-header 文件头 Display the ELF file header
-l –program-headers 程序 Display the program headers
–segments An alias for –program-headers
-S –section-headers 段头 Display the sections’ header
--sections
An alias for –section-headers
-e –headers 全部头 Equivalent to: -h -l -S
-s –syms 符号表 Display the symbol table
--symbols
An alias for –syms
-n –notes 内核注释 Display the core notes (if present)
-r –relocs 重定位 Display the relocations (if present)
-u –unwind Display the unwind info (if present)
-d –dynamic 动态段 Display the dynamic segment (if present)
-V –version-info 版本 Display the version sections (if present)
-A –arch-specific CPU构架 Display architecture specific information (if any).
-D –use-dynamic 动态段 Use the dynamic section info when displaying symbols
-x –hex-dump= 显示 段内内容Dump the contents of section
-w[liaprmfFso] or
-I –histogram Display histogram of bucket list lengths
-W –wide 宽行输出 Allow output width to exceed 80 characters
-H –help Display this information
-v –version Display the version number of readelf
```

**示例**
想知道一个应用程序的可运行的架构平台:
``` shell
$readelf -h main| grep Machine
```

-h选项将显示文件头的概要信息，从里面可以看到，有很多有用的信息：
``` shell
$readelf -h main
ELF Header:
Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
Class:                             ELF64
Data:                              2 s complement, little endian
Version:                           1 (current)
OS/ABI:                            UNIX - System V
ABI Version:                       0
Type:                              EXEC (Executable file)
Machine:                           Advanced Micro Devices X86-64
Version:                           0x1
Entry point address:               0x400790
Start of program headers:          64 (bytes into file)
Start of section headers:          5224 (bytes into file)
Flags:                             0x0
Size of this header:               64 (bytes)
Size of program headers:           56 (bytes)
Number of program headers:         8
Size of section headers:           64 (bytes)
Number of section headers:         29
Section header string table index: 26
```

一个编译好的应用程序，想知道其编译时是否使用了-g选项（加入调试信息）:
```
$readelf -S main| grep debug
```
用-S选项是显示所有段信息；如果编译时使用了-g选项，则会有debug段;

查看.o文件是否编入了调试信息（编译的时候是否加了-g):
`` $readelf -S Shpos.o | grep debug ``

### 翻屏：	
	空格键：向文件尾部翻一屏；
	b：向文件首部翻一屏；
	Ctrl+d：向文件尾部翻半屏；（DOWN）
	Ctrl+u：向文件首部翻半屏；（UP）
	Ctrl+F：向文件首部翻一屏；（Forward）
	Ctrl+B：向文件尾部翻一屏；（Back）
	回车键：向文件尾部翻一行；
	k：向文件首部翻一行；
	G：跳转至最后一行；
	#G：跳转至指定行，#是一个数字；
	1G：跳转至文件首部；
	:<N> or <N>gg 跳转到第 N 行

	文本搜索：
			/keyword : 从文件首部向文件尾部依次查找；不区分字符大小写；
			?keyword : 从文件尾部向文件首部依次查找；
			n：继续查找下一个；
			N：继续查找上一个；
	
	退出：
		 q：quit


### **在文件夹所有文件中查找相关内容**

``grep -nr "str" path``

**参数说明**

``` shell
grep: 就是grep命令
-nr: n是line number行号，r是recursive，可以理解为遍历文件文件夹
"str": 双引号里面的str就是要查找的内容，用双引号何以使用空格和一些字符进行查找
path: path就是查找路径，默认是./，也就是当前目录开始，也可以手动指定目录
 ```

**限定查找的文件条件**
``find path -name "PATTEN" | xargs grep -n "str" | grep "str"``


### **screen**
用于命令行终端切换

- 实例

``` shell
screen -ls -> 列出当前所有的session

screen -S yourname -> 新建一个叫yourname的session
screen -r yourname -> 回到yourname这个session
screen -d yourname -> 远程detach某个session

screen -X -S yourname quit -> 删除yourname这个session
  ```

在每个screen session 下，所有命令都以 ctrl+a(C-a) 开始，不释放然后再按下。

``` shell
C-a x -> 锁住当前的 window，需用用户密码解锁
C-a d -> detach，暂时离开当前session，将目前的 screen session (可能含有多个 windows) 丢到后台执行，并会回到还没进 screen 时的状态，此时在 screen session 里，每个 window 内运行的 process (无论是前台/后台)都在继续执行，即使 logout 也不影响。 
C-a w -> 显示所有窗口列表
C-a k -> kill window，强行关闭当前的 window
 ```

### lsof
显示Linux系统当前已打开的所有文件列表 lsof -p pid

- 实例

# 列出指定进程号所打开的文件:
``lsof -p $pid``

#### 获取端口对应的进程ID=>pid
``lsof -i:9981 -P -t -sTCP:LISTEN``

#### 列出打开文件的进程:
``lsof $filename``


### **strace**
跟踪系统调用和信号

- 实例

# 追踪系统调用:
``strace ./test``

#### 系统调用统计
``strace -c ./test``
``` shell
oracle@orainst[orcl]:~
$strace -c ./test
execve("./test", ["./test"], [/* 41 vars */]) = 0
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 45.90    0.000140           5        27        25 open
 34.43    0.000105           4        24        21 stat64
  7.54    0.000023           5         5           old_mmap
  2.62    0.000008           8         1           munmap
  1.97    0.000006           6         1           uname
  1.97    0.000006           2         3           fstat64
  1.64    0.000005           3         2         1 read
  1.31    0.000004           2         2           close
  0.98    0.000003           3         1           brk
  0.98    0.000003           3         1           mmap2
  0.66    0.000002           2         1           set_thread_area
------ ----------- ----------- --------- --------- ----------------
100.00    0.000305                    68        47 total
```

- trace一个现有的进程
``strace -p pid``

- 重定向输出
``` shell
# 这两个命令都是将strace结果输出到文件test.txt中
strace -c -o test.txt ./test
strace -c ./test  2>test.txt
```

- 对系统调用进行计时
``` shell
oracle@orainst[orcl]:~
$strace -T ./test
// 这里只摘录部分结果
read(0, 1
"1\n", 1024)                    = 2 <2.673455>
fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0 <0.000014>
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xbf5fe000 <0.000017>
write(1, "000000001\n", 10000000001
)             = 10 <0.000016>
munmap(0xbf5fe000, 4096)                = 0 <0.000020>
exit_group(0)                           = ?
```