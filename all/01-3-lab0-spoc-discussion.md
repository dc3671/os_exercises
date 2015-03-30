# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- [x]  


>  * 已读懂


虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- [x]  


>  ucore的接口实现等细节

> 中断寄存器和非通用寄存器等。



哪些困难（请分优先级）会阻碍你自主完成lab实验？
- [x]  

>   尚不清楚

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- [x]  

<<<<<<< HEAD
>   使用list命令
=======
> 1. 在gdb中通过break加行号得到物理地址，list加*物理地址得到行号。
> 2. 用nm, objdump工具可以看到
>>>>>>> 460fe39661c06217deeef5a7cd860eac28ec8ffa

了解函数调用栈对lab实验有何帮助？
- [x]  


>   更好的了解操作系统启动的原理，有利于汇编代码的编写
> 除了错可以调试 
> 对于函数的调用过程和程序的运行过程有更好的理解。
> 便于调试以及检查。 


你希望从lab中学到什么知识？
- [x]  

>   操作系统的具体实现和设计细节

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- [x]  


>  * 使用virtualbox安装已配置环境的Linux 没有遇到困难



熟悉基本的git命令行操作命令，从github上
的 http://www.github.com/chyyuu/ucore_lab 下载
ucore lab实验
- [x]  

<<<<<<< HEAD
>  已完成
=======
> clone 仓库 
> gitclone http://www.github.com/chyyuu/ucore_lab
>>>>>>> 460fe39661c06217deeef5a7cd860eac28ec8ffa

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- [x]   

<<<<<<< HEAD
>  已完成

对于如下的代码段，请说明”：“后面的数字是什么含义
```
/* Gate descriptors for interrupts and traps */
struct gatedesc {
=======
> 清除文件夹：make clean 
> 编译lab1：make 
> 调出debug命令行：make debug

对于如下的代码段，请说明”：“后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
>>>>>>> 460fe39661c06217deeef5a7cd860eac28ec8ffa
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
<<<<<<< HEAD
};
```

- [x]  

> * 冒号后表示位域，即该变量存储时占用的位数,指定了一个struct中每个成员所占的位数

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

=======
 };
 ```

- [x]  

> 每一个filed(域，成员变量)在struct(结构)中所占的位数; 也称“位域”，用于表示这个成员变量占多少位(bit)。

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

- [x]  0x10002

> https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab0/lab0_ex3.c

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
>>>>>>> 460fe39661c06217deeef5a7cd860eac28ec8ffa
- [x]  

>  * windows上：60930
>  * ios上：65538

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- [x]  

> 
```
#include "list.h"
#include <cstdio>
int main() {
	list_entry *a = new list_entry();
	list_entry *b = new list_entry();
	list_entry *c = new list_entry();
	list_entry *d = new list_entry();
	list_entry *e = new list_entry();
	list_entry *f = new list_entry();
	list_init(a);
	list_init(e);
	list_add(a, b);
	list_add_before(a, c);
	list_add_after(a, d);
	list_del(c);
	list_del_init(b);
	printf("%d\n", list_empty(e));
	printf("%d\n", list_next(a));
	printf("%d\n", list_prev(e));
	return 0;
}
```
---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

>  

---

