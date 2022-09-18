# Go Channel 的基本运用和原理分析

Go 在 goroutine 的通信经常会提及的设计思想是：不要通过共享内存的方式进行通信，而应该通过通信的方式共享内存。这和 Java 语言不通，Java 中多个线程传递数据的方式一般都是通过共享内存或者其他共享资源的方式解决线程竞争问题。

## Channel

### 基本操作

```go
ch := make(chan int, 10) // 创建 channel
ch <- 1 // 写入
v := <- ch // 读取
```

- 使用 make 初始化 channel ，并可以设置容量 ，10 就是容量 capacity 表示 Channel 容纳的最多元素的数量，即 Channel 缓存的大小。

- 如果没有设置容量，或者容量是为0 ，那说明没有缓存，即为同步。发送和接收需要同步进行，否则容易发生堵塞

- 可定义只读 channel 或只写 channel

```go
ch2 := make(<-chan int) // 只可以用来接收 int 类型的数据
ch3 := make(chan<- int) // 只可以用来发送 int 类型的数据
```

### 常用操作

#### for rang 读取 channel

缓存 channel 需要不断读取时，可以使用 for range 。当channel 关闭了，for 循环就会自动退出，这样就不需要再进行监控。

```go
ch := make(chan int, 10)
for x := range ch {
    fmt.Println(x)
}
```

#### _, ok  判断 channel 是否关闭

读已关闭的 channel 会得到零值，所以可以使用该方式先判断是否已关闭。若 ok=true 则表示读到数据，chan 未关闭；若 ok=false 则表示chan 已关闭，无数据读到。

```go
if v,ok := <-ch; ok{
     fmt.Println(v)
}
```

#### select 处理多个 channel

当有多个通道，需要对多个 channel 进行监控时，可使用 select 。需要注意 select 只处理伟阻塞的 case ，若通道为 nil 时对应的 case 将会堵塞。

```go
ch := make(chan int, 10)
ch1 := make(chan int, 10)
func doLinsen(){
    select {
        case <-ch:
            fmt.Println(1)
        case <-ch1:
            fmt.Println(2)
    }
}
```

#### channel 控制读写权限

控制 channel 只读或者只写

```go
// 只读 readCh的数据，声明为 <-chan int
func consumer(readCh <-chan int) {
    for x:= range readCh{
        fmt.Println(x)
    }
}
```

#### 使用 time 实现 channel 无阻塞读写

select + channel 方法处理 channel 时，为操作加上超时的处理

```go
func consumer(){
    select {
        case ch <-x:
          fmt.Println(ch)
        case time.After(time.Microsecond): // 超时处理
            return errors.New("read time out")
    }
}
```

#### 使用 chan struct{} 作为信号 channel

协程通过传递退出信号，告知协程退出，而这个退出信号就可以使用 空 struct。

```go
type Handler struct {
    stopCh chan struct{}
    reqCh chan *Request
}
```



### 应用场景

channel 主要运用在数据流动的场景中：

- 消息传递

- 事件消息订阅与广播

- 并发控制

- 同步与异步

- 任务分发

## Channel 底层原理

```go
type hchan struct {
    qcount   uint           // total data in the queue
    dataqsiz uint           // size of the circular queue
    buf      unsafe.Pointer // points to an array of dataqsiz elements
    elemsize uint16
    closed   uint32
    elemtype *_type // element type
    sendx    uint   // send index
    recvx    uint   // receive index
    recvq    waitq  // list of recv waiters
    sendq    waitq  // list of send waiters

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex
}
```

如上，Channel 的结构体是 hchan。含有11个字段：

- buf 是用来存储缓存数据，是个循环链表

- sendx 和 recvx 是用来记录 buf 循环链表中 发送或接收的索引

- lock 是互斥锁

- recvq 和 sendq 是接收和发送的协程队列



参考资料：

- https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/
