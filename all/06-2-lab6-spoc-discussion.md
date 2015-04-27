# lab6 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。


## 个人思考题

### 总体介绍

 - ucore中的调度点在哪里，完成了啥事？
 - 进程控制块中与调度相关的字段有哪些？
 - ucore的就绪队列数据结构在哪定义？在哪进行修改？
 - ucore的等待队列数据结构在哪定义？在哪进行修改？

### 调度算法支撑框架

 - 调度算法支撑框架中的各个函数指针的功能是啥？会被谁在何种情况下调用？
 - 调度函数schedule()的调用函数分析，可以了解进程调度的原因。请分析ucore中所有可能的调度位置，并说明可能的调用原因。
 
### 时间片轮转调度算法

 - 时间片轮转调度算法是如何基于调度算法支撑框架实现的？
 - 时钟中断如何调用RR_proc_tick()的？

### stride调度算法

 - stride调度算法的思路？ 
 - stride算法的特征是什么？
 - stride调度算法是如何避免stride溢出问题的？
 - 无符号数的有符号比较会产生什么效果？
 - 什么是斜堆(skew heap)?

## 小组练习与思考题

### (1)(spoc) 理解调度算法支撑框架的执行过程

即在ucore运行过程中通过`cprintf`函数来完整地展现出来多个进程在调度算法和框架的支撑下，在相关调度点如何动态调度和执行的细节。(约全面细致约好)

请完成如下练习，完成代码填写，并形成spoc练习报告
> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 练习用的[lab6 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/labcodes_answer/lab6_result)

- 代码见[github-dc3671](https://github.com/dc3671/ucore_lab/tree/master/labcodes_answer/lab6_result)
- 内核抢占点有以下几个（调用schedule函数）

    do_wait : 在父进程等待子进程结束时，将主动放弃CPU。
    do_exit : 在进程退出时，也会放弃CPU。
    init_main : initproc内核线程执行一个while循环不断重复do_wait等待所有用户态进程退出。
    cpu_idle : cpuidle内核线程执行一个while循环不断等待处于就绪状态的进程，如果有则主动放弃CPU。
    trap : 在trap处理的最后如果发现current进程的need_resched为1，则调用schedule函数。

- 在以上的抢占点处都已经加入了cprintf代码并运行正确。
