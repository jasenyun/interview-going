Go 并发模型 GMP

单进程时，CPU一次只能处理一个进程，导致所有的任务都是串行的，出现进程阻塞的话就会使得其他都得等待。后来操作系统开始支持多进程并发，进程阻塞时就会切换其他等待的进程，有效利用CPU 。

进程拥有很多资源，进程的创建，切换，销毁都会占用 CPU 很长时间，因其要对进程进行调度。后来提出线程，但同样引来很多问题：如锁，竞争冲突等

Go 中 协程是 goroutine ，轻量的占用内存小，由 runtime 调度。而线程是内核态，由进程调度。协程跟线程是有区别的，线程由CPU调度是抢占式的，**协程由用户态调度是协作式的**，一个协程让出CPU后，才执行下一个协程。

## GMP 模型

- G Gorouutine 协程

- M Thread 线程

- P Processor 处理器

在 go 中，线程是运行 gotinue 的实体，调度器的功能就是将可运行的 goroutine 分配到工作线程上。

GMP 调度流程：

- 线程 M 想运行任务就必须获取 P ，即与 P 关联

- 从 P 的本地队列获取 G，
  
  - 若本地 P无 G 则 M 从全局队列获取并放置在 P 的本地队列
  
  - 若全局无 G 则从其他 P 获取并放置在自己的本地队列中

- 获取到 G 后，线程 M 运行协程 G ，执行完继续获取下一个不断循环

## GMP 的结构

```go
//协程 G 的内部结构
type g struct {
    stack       stack   // g自己的栈
    m            *m      // 隶属于哪个M
    sched        gobuf   // 保存了g的现场，goroutine切换时通过它来恢复
    atomicstatus uint32  // G的运行状态
    goid         int64
    schedlink    guintptr // 下一个g, g链表
    preempt      bool //抢占标记
    lockedm      muintptr // 锁定的M,g中断恢复指定M执行
    gopc          uintptr  // 创建该goroutine的指令地址
    startpc       uintptr  // goroutine 函数的指令地址
}

//线程 M 的内部结构
type m struct {
    g0      *g     // g0, 每个M都有自己独有的g0

    curg          *g       // 当前正在运行的g
    p             puintptr // 隶属于哪个P
    nextp         puintptr // 当m被唤醒时，首先拥有这个p
    id            int64
    spinning      bool // 是否处于自旋

    park          note
    alllink       *m // on allm
    schedlink     muintptr // 下一个m, m链表
    mcache        *mcache  // 内存分配
    lockedg       guintptr // 和 G 的lockedm对应
    freelink      *m // on sched.freem
}
// 处理器 P 的内部结构
type p struct {
    id          int32
    status      uint32 // P的状态
    link        puintptr // 下一个P, P链表
    m           muintptr // 拥有这个P的M
    mcache      *mcache  

    // P本地runnable状态的G队列，无锁访问
    runqhead uint32
    runqtail uint32
    runq     [256]guintptr

    runnext guintptr // 一个比runq优先级更高的runnable G

    // 状态为dead的G链表，在获取G时会从这里面获取
    gFree struct {
        gList
        n int32
    }

    gcBgMarkWorker       guintptr // (atomic)
    gcw gcWork

}
```

调度过程中的阻塞

- I/O ，select 阻塞

- block on syscall 

- channel 阻塞，

- 等待锁

- runtime.Gosched()

参考资料：

- https://www.yuque.com/aceld/golang/srxd6d
