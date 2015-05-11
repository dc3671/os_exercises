# 死锁与IPC(lec 20) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 死锁概念 
 - 尝试举一个生活中死锁实例。
 - 可重用资源和消耗资源有什么区别？

### 可重用和不可撤销；
 - 资源分配图中的顶点和有向边代表什么含义？
 - 出现死锁的4个必要条件是什么？

### 死锁处理方法 
 - 死锁处理的方法有哪几种？它们的区别在什么地方？
 - 安全序列的定义是什么？

### 进程的最大资源需要量小于可用资源与前面进程占用资源的总合；
 - 安全、不安全和死锁有什么区别和联系？

### 银行家算法 
 - 什么是银行家算法？
 - 安全状态判断和安全序列是一回事吗？

### 死锁检测 
 - 死锁检测与安全状态判断有什么区别和联系？

> 死锁检测、安全状态判断和安全序列判断的本质就是资源分配图中的循环等待判断。

### 进程通信概念 
 - 直接通信和间接通信的区别是什么？
  本质上来说，间接通信可以理解为两个直接通信，间接通信中假定有一个永远有效的直接通信方。
 - 同步和异步通信有什么区别？
### 信号和管道 
 - 尝试视频中的信号通信例子。
 - 写一个检查本机网络服务工作状态并自动重启相关服务的程序。
 - 什么是管道？

### 消息队列和共享内存 
 - 写测试用例，测试管道、消息队列和共享内存三种通信机制进行不同通信间隔和通信量情况下的通信带宽、通信延时、带宽抖动和延时抖动方面的性能差异。
 
## 小组思考题

 - （spoc） 每人用python实现[银行家算法](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/bankers-homework.py)。大致输出可参考[参考输出](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/example-output.txt)。除了`YOUR CODE`部分需要填写代码外，在算法的具体实现上，效率也不高，可进一步改进执行效率。
- 见[github](https://github.com/dc3671/ucore_lab/blob/master/related_info/lab7/deadlock/bankers-homework.py)

 - (spoc) 以小组为单位，请思考在lab1~lab5的基础上，是否能够实现IPC机制，请写出如何实现信号，管道或共享内存（三选一）的设计方案。

    - 管道： 
    ```
    struct struct_pipe { 
        list //来保存所有的pipe的内存位置。 
        pipe_number //表示一共有多少个pipe_file
        limit //表示限制总的pipe大小 
    } 

    父进程新建pipe 
        pipe_open() { 
        get_pipe_id //分配空间。 
        return pipe_id 
    } 

    子进程fork的时候，pipe_id不变 
    void read(pipe_id,result_buffer) {
        P(rc_mutex); //开始对rc共享变量进行互斥访问 
        rc ++; //来了一个读进程，读进程数加1 
        if (rc==1) P(write)； //如是第一个读进程，判断是否有写进程在临界区。若有，读进程等待，若无，阻塞写进程 
        V(rc_mutex); //结束对rc共享变量的互斥访问 
        //读指定位置，位置根据pip_id寻找的struct_pipe决定；设置引用计数。 
        P(rc_mutex); //开始对rc共享变量的互斥访问 
        rc--; //一个读进程读完，读进程数减1 
        if (rc == 0) V(write)；//最后一个离开临界区的读进程需要判断是否有写进程 //需要进入临界区，若有，唤醒一个写进程进临界区 
        V(rc_mutex); //结束对rc共享变量的互斥访问 
    }

    void write(pipe_id,buffer) { 
        P(write); //无读进程，进入写进程；若有读进程，写进程等待 写指定位置，位置根据pip_id寻找的struct_pipe决定； 设置引用计数。 
        V(write); //写进程完成；判断是否有读进程需要进入临界区， 
        //若有，唤醒一个读进程进临界区 
    }

    父进程关闭管道 
    pipe_close(pipd_id) { 
        判断引用计数，如果引用计数不为0，关闭管道失败。 释放资源。 
    } 
    ```

 - (spoc) 扩展：用C语言实现某daemon程序，可检测某网络服务失效或崩溃，并用信号量机制通知重启网络服务。[信号机制的例子](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/ipc/signal-ex1.c)

 - (spoc) 扩展：用C语言写测试用例，测试管道、消息队列和共享内存三种通信机制进行不同通信间隔和通信量情况下的通信带宽、通信延时、带宽抖动和延时抖动方面的性能差异。[管道的例子](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/ipc/pipe-ex2.c)
