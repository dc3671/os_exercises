# lab2 SPOC思考题

NOTICE
- 有"w4l1"标记的题是助教要提交到学堂在线上的。
- 有"w4l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

## 个人思考题
---

x86保护模式中权限管理无处不在，下面哪些时候要检查访问权限()  (w4l1)
- [x] 内存寻址过程中
- [x] 代码跳转过程中
- [x] 中断处理过程中
- [ ] ALU计算过程中
 
> 前三个需要。这里假定ALU完成计算所需数据都已经在CPU内部了。


请描述ucore OS建立页机制的准备工作包括哪些步骤？ (w4l1) 
```
  + 采分点：说明了ucore OS在让页机制正常工作的主要准备工作
  - 答案没有涉及如下3点；（0分）
  - 描述了对GDT的初始化,完成了段机制（1分）
  - 除第二点外进一步描述了对物理内存的探测和空闲物理内存的管理。（2分）
  - 除上述两点外，进一步描述了页表建立初始过程和设置CR0控寄存器某位来使能页（3分）

```
- 在pmm.c中，由pmm_init函数进行初始化页机制，调用了gdt_init函数对GDT进行初始化

```
ts.ts_esp0 = (uint32_t)&stack0 + sizeof(stack0);
ts.ts_ss0 = KERNEL_DS;

gdt[SEG_TSS] = SEG16(STS_T32A, (uint32_t)&ts, sizeof(ts), DPL_KERNEL);
gdt[SEG_TSS].sd_s = 0;

lgdt(&gdt_pd);

ltr(GD_TSS);
```

- 此函数设置了内核栈的指针初始值以及默认的SS0，接着初始化TSS，最后调用lgdt重新写入段寄存器的值，调用ltr重新载入TSS。
- 之后需要调用page_init函数对物理内存空间进行探测，并释放已用空间，调用pmm_manager中的init_memmap以便对空闲内存进行管理。
- 最后执行enable_paging函数使能页机制，设置CR0寄存器的相关位CR0_PE, CR0_PG, CR0_AM, CR0_WP, CR0_NE, CR0_TS, CR0_EM, CR0_MP，CR0_TS，CR0_EM。

---

## 小组思考题
---

（1）（spoc）请用lab1实验的基准代码（即没有修改的需要填空的源代码）来做如下实验： 执行`make qemu`，会得到一个输出结果，请给出合理的解释：为何qemu退出了？【提示】需要对qemu增加一些用于基于执行过的参数，重点是分析其执行的指令和产生的中断或异常。 

- 输出如下：

```
(THU.CST) os is loading ...

Special kernel symbols:
  entry  0x00100000 (phys)
  etext  0x0010316e (phys)
  edata  0x0010da56 (phys)
  end    0x0010ee00 (phys)
Kernel executable memory footprint: 60KB
++ setup timer interrupts
```

- 说明计时器崩溃，再联想lab1初始代码没有完成IDT，无法处理时钟中断，故而得出原因。   

（2）(spoc)假定你已经完成了lab1的实验,接下来是对lab1的中断处理的回顾：请把你的学号对37(十进制)取模，得到一个数x（x的范围是-1<x<37），然后在你的答案的基础上，修init.c中的kern_init函数，在大约36行处，即

```
    intr_enable();              // enable irq interrupt
```

语句之后，加入如下语句(把x替换为你学号 mod 37得的值)：

```
    asm volatile ("int $x");
```    

然后，请回答加入这条语句后，执行`make qemu`的输出结果与你没有加入这条语句后执行`make qemu`的输出结果的差异，并解释为什么有差异或没差异？ 

- 学号为2012012390，mod37为9，重新运行之后的结果如下：

```
(THU.CST) os is loading ...

Special kernel symbols:
  entry  0x00100000 (phys)
  etext  0x001033ad (phys)
  edata  0x0010ea56 (phys)
  end    0x0010fe00 (phys)
Kernel executable memory footprint: 64KB
ebp:0x00007b18 eip:0x00100a1a args:0x00010094 0x00000000 0x00007b48 0x00100081 
    kern/debug/kdebug.c:306: print_stackframe+21
ebp:0x00007b28 eip:0x00100d18 args:0x00000000 0x00000000 0x00000000 0x00007b98 
    kern/debug/kmonitor.c:125: mon_backtrace+10
ebp:0x00007b48 eip:0x00100081 args:0x00000000 0x00007b70 0xffff0000 0x00007b74 
    kern/init/init.c:49: grade_backtrace2+19
ebp:0x00007b68 eip:0x001000a2 args:0x00000000 0xffff0000 0x00007b94 0x00000029 
    kern/init/init.c:54: grade_backtrace1+27
ebp:0x00007b88 eip:0x001000be args:0x00000000 0x00100000 0xffff0000 0x00100043 
    kern/init/init.c:59: grade_backtrace0+19
ebp:0x00007ba8 eip:0x001000de args:0x00000000 0x00000000 0x00000000 0x001033c0 
    kern/init/init.c:64: grade_backtrace+26
ebp:0x00007bc8 eip:0x00100050 args:0x00000000 0x00000000 0x00010094 0x00000000 
    kern/init/init.c:28: kern_init+79
ebp:0x00007bf8 eip:0x00007d66 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8 
    <unknow>: -- 0x00007d65 --
++ setup timer interrupts
trapframe at 0x7b64
  edi  0x00000001
  esi  0x00000000
  ebp  0x00007bc8
  oesp 0x00007b84
  ebx  0x00010094
  edx  0x000000a1
  ecx  0x00000000
  eax  0x000000ff
  ds   0x----0010
  es   0x----0010
  fs   0x----0023
  gs   0x----0023
  trap 0x0000000d General Protection
  err  0x00000204
  eip  0x001027aa
  cs   0x----0008
  flag 0x00000002 IOPL=0
kernel panic at kern/trap/trap.c:183:
    unexpected trap in kernel.

Welcome to the kernel debug monitor!!
Type 'help' for a list of commands.
K> 
```

- 可见由于数字9号中断不在IDT内，系统无法作出对应处理，所以进入了debug模式。而不会像未输入之前那样正常运行，每隔100ms输出时钟中断的计数器。

（3）对于lab2的输出信息，请说明数字的含义
```
e820map:
  memory: 0009fc00, [00000000, 0009fbff], type = 1.
  memory: 00000400, [0009fc00, 0009ffff], type = 2.
  memory: 00010000, [000f0000, 000fffff], type = 2.
  memory: 07ee0000, [00100000, 07fdffff], type = 1.
  memory: 00020000, [07fe0000, 07ffffff], type = 2.
  memory: 00040000, [fffc0000, ffffffff], type = 2.
```
修改lab2，让其显示` type="some string"` 让人能够读懂，而不是不好理解的数字1,2  (easy) 
- [x]  

> 

（4）(spoc)有一台只有页机制的简化80386的32bit计算机，有地址范围位0~256MB的物理内存空间（physical memory），可表示大小为256MB，范围为0xC0000000~0xD0000000的虚拟地址空间（virtual address space）,页大小（page size）为4KB，采用二级页表，一个页目录项（page directory entry ，PDE）大小为4B,一个页表项（page-table entries PTEs）大小为4B，1个页目录表大小为4KB，1个页表大小为4KB。
```
PTE格式（32 bit） :
  PFN19 ... PFN0|NOUSE9 ... NOUSE0|WRITABLE|VALID
PDE格式（32 bit） :
  PT19 ... PT0|NOUSE9 ... NOUSE0|WRITABLE|VALID
 
其中：
NOUSE9 ... NOUSE0为保留位，要求固定为0
WRITABLE：1表示可写，0表示只读
VLAID：1表示有效，0表示无效
```

假设ucore OS已经为此机器设置好了针对如下虚拟地址<-->物理地址映射的二级页表，设置了页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐），其值为0。已经建立好了从0x1000~41000的二级页表，且页目录表的index为0~63的页目录项的(PT19 ... PT0)的值=(index+1)。
请写出一个translation程序（可基于python, ruby, C, C++，LISP等），输入是一个虚拟地址和一个物理地址，能够自动计算出对应的页目录项的index值,页目录项内容的值，页表项的index值，页表项内容的值。即(pde_idx, pde_ctx, pte_idx, pte_cxt)

请用如下值来验证你写的程序的正确性：
```
va 0xc2265b1f, pa 0x0d8f1b1f
va 0xcc386bbc, pa 0x0414cbbc
va 0xc7ed4d57, pa 0x07311d57
va 0xca6cecc0, pa 0x0c9e9cc0
va 0xc18072e8, pa 0x007412e8
va 0xcd5f4b3a, pa 0x06ec9b3a
va 0xcc324c99, pa 0x0008ac99
va 0xc7204e52, pa 0x0b8b6e52
va 0xc3a90293, pa 0x0f1fd293
va 0xce6c3f32, pa 0x007d4f32
```

参考的输出格式为：
```
va 0xcd82c07c, pa 0x0c20907c, pde_idx 0x00000336, pde_ctx  0x00037003, pte_idx 0x0000002c, pte_ctx  0x0000c20b
```

- 4KB，12位  256MB，28位，虚拟地址32位，10+10+12
- 0xc2265b1f = 1100 0010 0010 0110 0101 1011 0001 1111
- = 1100-0010-00 10-0110-0101 1011-0001-1111
- = 308 265 0xb1f
- 所以0x308为pde_idx，0x265为pte_idx，pte_ctx为物理地址前20位加上10位0加上读写位、有效位两位1。
- 程序算出结果为：

```
va: 0xc2265b1f,  pa: 0x0d8f1b1f,  pde_idx: 0x00000308,  pde_ctx: 0x00009003,  pte_idx: 0x00000265,  pte_ctx: 0x0d8f1003, 
va: 0xcc386bbc,  pa: 0x0414cbbc,  pde_idx: 0x00000330,  pde_ctx: 0x00031003,  pte_idx: 0x00000386,  pte_ctx: 0x0414c003, 
va: 0xc7ed4d57,  pa: 0x07311d57,  pde_idx: 0x0000031f,  pde_ctx: 0x00020003,  pte_idx: 0x000002d4,  pte_ctx: 0x07311003, 
va: 0xca6cecc0,  pa: 0x0c9e9cc0,  pde_idx: 0x00000329,  pde_ctx: 0x0002a003,  pte_idx: 0x000002ce,  pte_ctx: 0x0c9e9003, 
va: 0xc18072e8,  pa: 0x007412e8,  pde_idx: 0x00000306,  pde_ctx: 0x00007003,  pte_idx: 0x00000007,  pte_ctx: 0x00741003, 
va: 0xcd5f4b3a,  pa: 0x06ec9b3a,  pde_idx: 0x00000335,  pde_ctx: 0x00036003,  pte_idx: 0x000001f4,  pte_ctx: 0x06ec9003, 
va: 0xcc324c99,  pa: 0x0008ac99,  pde_idx: 0x00000330,  pde_ctx: 0x00031003,  pte_idx: 0x00000324,  pte_ctx: 0x0008a003, 
va: 0xc7204e52,  pa: 0x0b8b6e52,  pde_idx: 0x0000031c,  pde_ctx: 0x0001d003,  pte_idx: 0x00000204,  pte_ctx: 0x0b8b6003, 
va: 0xc3a90293,  pa: 0x0f1fd293,  pde_idx: 0x0000030e,  pde_ctx: 0x0000f003,  pte_idx: 0x00000290,  pte_ctx: 0x0f1fd003, 
va: 0xce6c3f32,  pa: 0x007d4f32,  pde_idx: 0x00000339,  pde_ctx: 0x0003a003,  pte_idx: 0x000002c3,  pte_ctx: 0x007d4003,
```

---

## 开放思考题

---

（1）请简要分析Intel的x64 64bit体系结构下的分页机制是如何实现的 
```
  + 采分点：说明Intel x64架构的分页机制的大致特点和页表执行过程
  - 答案没有涉及如下3点；（0分）
  - 正确描述了x64支持的物理内存大小限制（1分）
  - 正确描述了x64下的多级页表的级数和多级页表的结构（2分）
  - 除上述两点外，进一步描述了在多级页表下的虚拟地址-->物理地址的映射过程（3分）
 ```
- [x]  

>  

（2）Intel8086不支持页机制，但有hacker设计过包含未做任何改动的8086CPU的分页系统。猜想一下，hacker是如何做到这一点的？提示：想想MMU的逻辑位置

- [x]  

> 

---
