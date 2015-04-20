# lab5 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。


## 个人思考题

### 总体介绍

 - 第一个用户进程创建有什么特殊的？
 
 - 系统调用的参数传递过程？
     + 放在用户栈，参数传递起始位置。
 - getpid的返回值放在什么地方了？
     + 返回值存在eax。

### 进程的内存布局

 - 尝试在进程运行过程中获取内核堆栈和用户堆栈的调用栈？
 - 尝试在进程运行过程中获取内核空间中各进程相同的页表项（代码段）和不同的页表项（内核堆栈）？

### 执行ELF格式的二进制代码-do_execve的实现

 - 在do_execve中进程清空父进程时，当前进程是哪一个？在什么时候开始使用新加载进程的地址空间？
 - 新加载进程的第一级页表的建立代码在哪？

### 执行ELF格式的二进制代码-load_icode的实现

 - 第一个内核线程和第一个用户进程的创建有什么不同？
 - 尝试跟踪分析新创建的用户进程的开始执行过程？

### 进程复制

 - 为什么新进程的内核堆栈可以先于进程地址空间复制进行创建？
 - 进程复制的代码在哪？复制了哪些内容？
 - 进程复制过程中有哪些修改？为什么要修改？

### 内存管理的copy-on-write机制
 - 什么是写时复制？
 - 写时复制的页表在什么时候进行复制？共享地址空间和写时复制有什么不同？

## 小组练习与思考题

### (1)(spoc) 在真实机器的u盘上启动并运行ucore lab,

请准备一个空闲u盘，然后请参考如下网址完成练习

https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-boot-with-grub2-in-udisk.md

(报告可课后完成)请理解grub multiboot spec的含义，并分析ucore_lab是如何实现符合grub multiboot spec的，并形成spoc练习报告。

- 首先删除 U 盘上的所有分区然后创建一个新的分区，然后将分区格式化为 FAT32 。
- 将 U 盘挂载到 /media/MULTIBOOT 然后在 U 盘上安装 grub 。然后按照提示创建 grub.cfg 如下

```
menuentry 'ucore-lab1' {
    knetbsd /boot/grub_kernel
}
```

- 最后把编译好的内核拷贝到 ./boot 目录下就可以重启运行了。

### (2)(spoc) 理解用户程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 练习用的[lab5 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab5/lab5-spoc-discuss)

#### 掌握知识点
1. 用户进程的启动、运行、就绪、等待、退出
2. 用户进程的管理与简单调度
3. 用户进程的上下文切换过程
4. 用户进程的特权级切换过程
5. 用户进程的创建过程并完成资源占用
6. 用户进程的退出过程并完成资源回收

> 注意，请关注：内核如何创建用户进程的？用户进程是如何在用户态开始执行的？用户态的堆栈是保存在哪里的？

阅读代码，在现有基础上再增加一个用户进程A，并通过增加cprintf函数到ucore代码中，
能够把个人思考题和上述知识点中的内容展示出来：即在ucore运行过程中通过`cprintf`函数来完整地展现出来进程A相关的动态执行和内部数据/状态变化的细节。(约全面细致约好)

请完成如下练习，完成代码填写，并形成spoc练习报告

- 见[github-dc3671](https://github.com/dc3671/ucore_lab/tree/master/related_info/lab5/lab5-spoc-discuss/)
- 输出结果如下：

```
schedule : before proc 0 running.
schedule : next proc 1 will run.
proc_run : will run 1
alloc_proc:a new proc struct alloced.
setup_kstack for -1 at 0xc03ac000
copy the memory of 1 to the new process -1
copy_thread : set proc -1 stack at 0x00000000 tf at 0xc03a6f58
        now eip at 0xc010a2a0, esp at 0xc03adfb4
do_fork: fork process 1 to 2
wakeup_proc : wake up 2.
alloc_proc:a new proc struct alloced.
setup_kstack for -1 at 0xc03ae000
copy the memory of 1 to the new process -1
copy_thread : set proc -1 stack at 0x00000000 tf at 0xc03a6f58
        now eip at 0xc010a2a0, esp at 0xc03affb4
do_fork: fork process 1 to 3
wakeup_proc : wake up 3.
schedule : before proc 1 running.
schedule : next proc 3 will run.
proc_run : will run 3
kernel_execve: pid = 3, name = "exit".
load_icode : load ELF program for proc 3
load_icode : 1.create memory for process.
load_icode : 2.create PDT for process.
setup the page directory table at 0xc03b0000
load_icode : 3.build BSS for process.
load_icode : 4.build stack for process.
load_icode : 5.set memory cr3.
load_icode : 6.set trapframe, working in user mode.
set_proc_name : set name of 3 to exit
do_execve : finish execve, 3 add to scheduler list.
I am the parent. Forking the child...
alloc_proc:a new proc struct alloced.
setup_kstack for -1 at 0xc03bf000
copy the memory of 3 to the new process -1
setup the page directory table at 0xc03c1000
copy_thread : set proc -1 stack at 0xafffff40 tf at 0xc03affb4
        now eip at 0xc010a2a0, esp at 0xc03c0fb4
do_fork: fork process 3 to 4
wakeup_proc : wake up 4.
I am parent, fork a child pid 4
I am the parent, waiting now..
schedule : before proc 3 running.
schedule : next proc 2 will run.
proc_run : will run 2
kernel_execve: pid = 2, name = "exit".
load_icode : load ELF program for proc 2
load_icode : 1.create memory for process.
load_icode : 2.create PDT for process.
setup the page directory table at 0xc03d0000
load_icode : 3.build BSS for process.
load_icode : 4.build stack for process.
load_icode : 5.set memory cr3.
load_icode : 6.set trapframe, working in user mode.
set_proc_name : set name of 2 to exit
do_execve : finish execve, 2 add to scheduler list.
I am the parent. Forking the child...
alloc_proc:a new proc struct alloced.
setup_kstack for -1 at 0xc03df000
copy the memory of 2 to the new process -1
setup the page directory table at 0xc03e1000
copy_thread : set proc -1 stack at 0xafffff40 tf at 0xc03adfb4
        now eip at 0xc010a2a0, esp at 0xc03e0fb4
do_fork: fork process 2 to 5
wakeup_proc : wake up 5.
I am parent, fork a child pid 5
I am the parent, waiting now..
schedule : before proc 2 running.
schedule : next proc 5 will run.
proc_run : will run 5
I am the child.
do_yield : resched , proc 0 give up CPU.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
I am the child.
do_yield : resched , proc 0 give up CPU.
schedule : before proc 4 running.
schedule : next proc 5 will run.
proc_run : will run 5
do_yield : resched , proc -1069674760 give up CPU.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
do_yield : resched , proc -1069805788 give up CPU.
schedule : before proc 4 running.
schedule : next proc 5 will run.
proc_run : will run 5
do_yield : resched , proc -1069674716 give up CPU.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
do_yield : resched , proc -1069805788 give up CPU.
schedule : before proc 4 running.
schedule : next proc 5 will run.
proc_run : will run 5
do_yield : resched , proc -1069674716 give up CPU.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
do_yield : resched , proc -1069805788 give up CPU.
schedule : before proc 4 running.
schedule : next proc 5 will run.
proc_run : will run 5
do_yield : resched , proc -1069674716 give up CPU.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
do_yield : resched , proc -1069805832 give up CPU.
schedule : before proc 4 running.
schedule : next proc 5 will run.
proc_run : will run 5
do_yield : resched , proc -1069674716 give up CPU.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
do_yield : resched , proc -1069805788 give up CPU.
schedule : before proc 4 running.
schedule : next proc 5 will run.
proc_run : will run 5
do_yield : resched , proc -1069674716 give up CPU.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
do_yield : resched , proc -1069805788 give up CPU.
schedule : before proc 4 running.
schedule : next proc 5 will run.
proc_run : will run 5
free the page directory table at 0xc03e1000
do_exit : proc 5 change to ZOMBIE. exit code is -66436
do_exit: test proc 2
wakeup_proc : wake up 2.
schedule : before proc 5 running.
schedule : next proc 4 will run.
proc_run : will run 4
free the page directory table at 0xc03c1000
do_exit : proc 4 change to ZOMBIE. exit code is -66436
do_exit: test proc 3
wakeup_proc : wake up 3.
schedule : before proc 4 running.
schedule : next proc 3 will run.
proc_run : will run 3
remove_links : clean the link of 4free the stack of 4 at 0xc03bf000
waitpid 4 ok.
exit pass.
free the page directory table at 0xc03b0000
do_exit : proc 3 change to ZOMBIE. exit code is 0
do_exit: test proc 1
wakeup_proc : wake up 1.
schedule : before proc 3 running.
schedule : next proc 2 will run.
proc_run : will run 2
remove_links : clean the link of 5free the stack of 5 at 0xc03df000
waitpid 5 ok.
exit pass.
free the page directory table at 0xc03d0000
do_exit : proc 2 change to ZOMBIE. exit code is 0
do_exit: test proc 1
schedule : before proc 2 running.
schedule : next proc 1 will run.
proc_run : will run 1
remove_links : clean the link of 3free the stack of 3 at 0xc03ae000
schedule : before proc 1 running.
schedule : next proc 1 will run.
remove_links : clean the link of 2free the stack of 2 at 0xc03ac000
schedule : before proc 1 running.
schedule : next proc 1 will run.
all user-mode processes have quit.
```
