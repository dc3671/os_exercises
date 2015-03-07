# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- 大部分能读懂，但是对于.cfi_def_cfa_offset 8这种类似的语句就不能理解语法了。
- 但是能猜到是由于源代码在c中嵌入汇编导致的。

>  

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
-  文件系统和进程管理
-  说到文件系统，其实也就只清楚一个页式文件系统，对于系统层面怎么进行文件管理仍旧不了解，而且Linux和Windows还不一样，而哪里不一样又不太说得上来。
-  进程管理的话，互斥锁、信号变量等在高性能导论里了解过，但是分时机制只是在计算机组成原理课上知道有哪些算法，还是更期待实践中了解一下。

>   

请给出你觉得的中断的作用是什么？使用中断有何利弊？
- 中断是来自处理器外部的IO设备的信号所导致的结果，用于停止CPU当前工作，响应外部设备。
- 适用情况是传输速度不高，传输量不大的情况，能较快响应IO设备。
- 但是若传输量很大，会占用CPU大量时间，对CPU的干扰十分巨大。

>   

哪些困难（请分优先级）会阻碍你自主完成lab实验？
- 最高：同时有另一门更热爱的课的作业需要完成，而两者截止时间又是一样的。
- 中：由于实验难度过大，卡在一个bug或类似问题上过长时间，会寻求帮助。
- 低：由于身体不舒服等情况，没有精神。

>   

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- 反编译可执行文件/源码，查看它的汇编代码，在代码前部会有其对应物理/线性地址，。

>   

了解函数调用栈对lab实验有何帮助？
- 函数调用栈是内存中管理程序运行的重要机制，而且汇编代码直接对函数调用栈进行操作，
- 了解函数调用栈最基本的作用就是可以帮助我们写好汇编，进而写好lab实验代码。

>   

你希望从lab中学到什么知识？
- 从代码的级别来对一个操作系统的运作进行深入了解，
- 上学期选数据库系统概论，便用C++写了一个类似MySQL的程序，从底层文件系统，到如何翻译、缓存如何处理都从代码级别有了很细的了解，感觉收获很多，所以希望能在操统课上由类似的收获。

>   

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- 由于课程之前就在使用Fedora Linux，所以对于实验需要的工具基本上都安装完毕并且熟悉使用了
- 除了QEMU这个专门用于硬件模拟的软件，但是在安装上也没有太大问题，直接用yum install qemu即可。

> 

熟悉基本的git命令，从github上（http://www.github.com/chyyuu/ucore_lab）下载ucore lab实验
- 已完成。

> 

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- 按照qemu的文档直接qemu ucore.img似乎无法运行，
- 之后按照文档里说的make qemu就可以出现系统窗口了，但是会不停闪烁。

对于如下的代码段，请说明”：“后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
 ```

- 表明该变量在结构中占几位，同时跟字节对齐有关，该结构按照16位进行对齐。

对于如下的代码段，
```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？

- 0x10002

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- https://github.com/dc3671/ucore_lab/blob/master/related_info/lab0/lab0_ex3.c
- 由于要求不是很清楚，所以只是简单的调用list.h中的一个接口运行了一下。

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

>  

---
