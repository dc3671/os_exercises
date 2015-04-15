# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍

(1) ucore的线程控制块数据结构是什么？

- 并没有专门的线程控制块数据结构，与进程控制块使用相同的proc_struct

### 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？

- 不同线程的pid也是不同的，但是同一进程的mm是相同的，所以如果mm相同或者cr3相同，可以判断是同一个进程下的不同线程。

(3) context和trapframe分别在什么时候用到？

- 进程切换和中断的时候。

(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？

- 见注释，中间部分的内容会由硬件自动保存。

```
    /* below here defined by x86 hardware */
    uint32_t tf_err;
    uintptr_t tf_eip;
    uint16_t tf_cs;
    uint16_t tf_padding4;
    uint32_t tf_eflags;
```

- 下面三个则是只在用户权限切换的时候保存。

```
    /* below here only when crossing rings, such as from user to kernel */
    uintptr_t tf_esp;
    uint16_t tf_ss;
    uint16_t tf_padding5;
```

### 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
tf.tf_eip = (uint32_t) kernel_thread_entry;
/kern-ucore/arch/i386/init/entry.S
/kern/process/entry.S
```

(6)内核线程的堆栈初始化在哪？
```
tf和context中的esp
```

(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？
- 父进程在`void syscall`函数中`tf->tf_regs.reg_eax = syscalls[num](arg);`，调用`sys_fork()`，再调用`do_fork()`，返回值

```
    ret = proc->pid;
fork_out:
    return ret;。
```

- 子进程在`do_fork()`调用`copy_thread(struct proc_struct *proc, uintptr_t esp, struct trapframe *tf)`函数中`proc->tf->tf_regs.reg_eax = 0;`

(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？

- 在kernel_thread函数里加输出，pid和mm指针

## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 掌握知识点
1. 内核线程的启动、运行、就绪、等待、退出
2. 内核线程的管理与简单调度
3. 内核线程的切换过程

### 练习用的[lab4 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)


请完成如下练习，完成代码填写，并形成spoc练习报告

### 1. 分析并描述创建分配进程的过程

> 注意 state、pid、cr3，context，trapframe的含义

- 在syscall中调用系统调用sys_fork，然后调用do_fork，层层返回，完成进程的创建。
- state表示运行状态，pid表示进程标识符，cr3是页目录项基址，contex是进程运行上下文，trapframe是中断所需的各种寄存器。

### 练习2：分析并描述新创建的内核线程是如何分配资源的

> 注意 理解对kstack, trapframe, context等的初始化

- 在do_fork()函数中完成了主要的创建分配进程工作。
- 首先调用alloc_page()给proc分配空间，如果失败直接返回；
- 然后建立内核堆栈，如果失败要回收空间；
- 然后`copy_mm()`复制或共享内存空间，不过本实验中都是内核线程，所有内核线程公用初始的boot_cr3指向的页表，因此该函数实际并未做任何事情，不过为后续考虑，失败后要回收堆栈和空间；
- 然后设立父进程，调用`copy_thread()`设立context，tf，获取pid，将其加入进程列表，调用`wakeup_proc()`设置其状态为运行，最后返回pid即可。

### 练习3：阅读代码，在现有基础上再增加一个内核线程，并通过增加cprintf函数到ucore代码中
能够把进程的生命周期和调度动态执行过程完整地展现出来

- 见[github-dc3671](https://github.com/dc3671/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss/kern/process/proc.c)

### 练习4 （非必须，有空就做）：增加可以睡眠的内核线程，睡眠的条件和唤醒的条件可自行设计，并给出测试用例，并在spoc练习报告中给出设计实现说明

### 扩展练习1: 进一步裁剪本练习中的代码，比如去掉页表的管理，只保留段机制，中断，内核线程切换，print功能。看看代码规模会小到什么程度。


