#lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
 1. 比较UEFI和BIOS的区别。
- UEFI旨在统一所有平台的系统引导，本身已经相当于一个小型的操作系统，可支持图形界面，将BIOS的一系列操作固件化，UEFI系统可以直接对硬盘里的后缀名为efi的文件进行操作，从而进行引导，所以现在安装系统不再像以前那样繁琐，直接拷贝安装程序，并从硬盘或u盘启动即可。而BIOS要多出长时间的自检，而且只能执行约定位置的一段代码，操作十分受限。

 1. 描述PXE的大致启动流程。
- 客户端个人电脑开机后， 在 TCP/IP Bootrom 获得控制权之前先做自我测试。
- Bootprom 送出 BOOTP/DHCP 要求以取得 IP。
- 如果服务器收到个人电脑所送出的要求， 就会送回 BOOTP/DHCP 回应，内容包括客户端的 IP 地址， 预设网关， 及开机映像文件。否则，服务器会忽略这个要求。
- Bootprom 由 TFTP 通讯协议从服务器下载开机映像文件。
- 个人电脑通过这个开机映像文件开机， 这个开机文件可以只是单纯的开机程式也可以是操作系统。
- 开机映像文件将包含 kernel loader 及压缩过的 kernel，此 kernel 将支持NTFS root系统。
- 远程客户端根据下载的文件启动机器。

## 3.2 系统启动流程
 1. 了解NTLDR的启动流程。
- 对于NTLDR，BIOS从MBR前446字节读入引导程序，并将CPU从实模式转换为平面内存模式
- NTLDR将启动小型的文件系统读取硬盘中的配置文件boot.ini，显示系统选择菜单。

 1. 了解GRUB的启动流程。
- GRUB分三个stage
- stage1将MBR的前446个字节载入内存，判断磁盘处于CHS或者LBR模式并运行相应代码，将stage1.5装载到内存并运行stage1.5
- stage1.5位于MBR与第一个分区之间的空隙，用于访问文件系统，找到stage2。
- stage2用于加载配置文件从而启动系统选择菜单或者grub的mini shell。

 1. 比较NTLDR和GRUB的功能有差异。
- 主要是两者支持的系统不一样，NTLDR主要是用于启动winNT/2k/XP，而GRUB作为第三方引导程序主要支持LINUX。
- 安装的时候，NTLDR由于是微软作靠山，会强制重写MDR，而GRUB是会将NTLDR纳入，当启动windows系统时会调用NTLDR进行启动，所以必须先安Windows再安Linux。

 1. 了解u-boot的功能。
- U-Boot对PowerPC系列处理器支持最为丰富，对Linux的支持最完善，是一款开源的引导程序，有丰富的设备驱动源码，如串口、以太网、SDRAM、FLASH、LCD、NVRAM、EEPROM、RTC、键盘等，较为丰富的开发调试文档与强大的网络技术支持。

## 3.3 中断、异常和系统调用比较
 1. 举例说明Linux中有哪些中断，哪些异常？
- 中断比如说键盘输入，设备故障；
- 异常比如说除零操作，段错误等。


1. Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)
- 总共有上百个，分为进程控制，文件操作，系统管理，内存管理，网络管理，socket控制，用户管理等
 

 1. 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)
- sys_exit、sys_fork、sys_wait、sys_exec等等40多个。
- 与Linux类似，文件操作、进程管理、内存管理等。
 
 ## 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
 - 在汇编代码里，通过对eax\ebx\ecx\edx四个寄存器分别赋系统调用所需参数，再由int进行系统调用，从而由内核态进行操作。
 - objdump用于查看可执行文件的信息，可用于反汇编文件
 - nm可用于查看可执行文件中涉及的符号，可以看出该文件使用了哪些系统调用，这里有：
0000000000000006 a SYS_close
000000000000003f a SYS_dup2
000000000000000b a SYS_execve
0000000000000001 a SYS_exit
0000000000000002 a SYS_fork
0000000000000013 a SYS_lseek
000000000000005a a SYS_mmap
000000000000005b a SYS_munmap
0000000000000005 a SYS_open
0000000000000066 a SYS_socketcall
0000000000000005 a SYS_socketcall_accept
0000000000000002 a SYS_socketcall_bind
0000000000000004 a SYS_socketcall_listen
0000000000000001 a SYS_socketcall_socket
0000000000000004 a SYS_write
- file用于查看文件类型，如下：
lab1-ex0.o: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=cbcf22c8d4172a9bf6b10ffee7c5a378e90fc81e, not stripped


 1. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
- strace用于跟踪查看可执行文件的系统调用，输出如下
hello world
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 26.15    0.000017          17         1           execve
 21.54    0.000014           2         9           mmap
 13.85    0.000009           2         4           mprotect
 10.77    0.000007           7         1           munmap
  7.69    0.000005           3         2           open
  6.15    0.000004           1         3           fstat
  4.62    0.000003           3         1           write
  3.08    0.000002           2         1         1 access
  1.54    0.000001           1         1           read
  1.54    0.000001           1         2           close
  1.54    0.000001           1         1           brk
  1.54    0.000001           1         1           arch_prctl
------ ----------- ----------- --------- --------- ----------------
100.00    0.000065                    27         1 total
 出错是由于main函数里未return

 
## 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
- 通过understand可以看到，比如说open，getstat\opendir等函数都调用了这个函数，接着将open函数的两个参数传给sys_open，接着调用syscall函数，在第一个参数设为SYS_open，如此便明确了是什么类型的调用，之后便是汇编码，内核态了。

 1. ucore的系统调用中返回结果的传递代码分析。
- 与上述过程相反，按照函数调用栈，一层层返回结果。

 1. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以syscall为核心，数十个以"sys_"开头的系统调用函数都通过传给syscall不同的参数进而在内核态进行不同的操作。


 1. 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
- 这个一天写不完吧...会尝试的

 
## 3.6 请分析函数调用和系统调用的区别
 1. 请从代码编写和执行过程来说明。
   1. 说明`int`、`iret`、`call`和`ret`的指令准确功能
- int\iret是针对系统调用的指令，涉及到内核态和用户态之间函数栈的转换；
- call\ret是用户态函数调用使用的指令，不涉及函数栈的转换。 
