go 并发数量的控制



goroutine 是轻量级线程，调度由 Go 运行时进行管理的。Go 语言的并发控制主要使用关键字 go 开启协程 goroutine。Go 协程（Goroutine）之间通过信道（channel）进行通信，简单的说就是多个协程之间通信的管道。信道可以防止多个协程访问共享内存时发生资源争抢的问题。语法格式：

```go
// 普通函数创建 goroutine
go 函数名(参数列表)

//匿名函数创建 goroutine
go func(参数列表){
    //函数体
}(调用参数列表)
```

协程可以开启多少个？是否有限制呢？ 

```go
func testRoutine() {
	var wg sync.WaitGroup
	for i := 0; i < math.MaxInt32; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			fmt.Printf("并发数量：%d/n", i)
			time.Sleep(time.Second)
		}(i)
	}
	wg.Wait()
}
```

以上代码开启了 math.MaxInt32个协程的并发，执行后可以看到结果直接 panic：“panic: too many concurrent operations on a single file or socket (max 1048575)”。整个并发操作超出了系统最大值。

![](/Users/jasenyang/Documents/pictures/pro-pic/2022-08-21-22-23-31-image.png)

## 控制协程数量

sync 同步机制

使用 sync.WaitGroup 启动指定数量的协程 goroutine。

```go
func testRoutine() {
	var wg = sync.WaitGroup{}

	taskCount := 5 // 指定并发数量
	for i := 0; i < taskCount; i++ {
		wg.Add(1)
		go func(i int) {
			fmt.Println("go func ", i)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

如果 taskcount 设置的很大超出了限制的，则其还是没有控制到并发数量。可以优化下设计，类似池的设计思想，通过允许最大连接数控制量，当超出了数量就需要等待释放，有空闲的连接的时候才可以继续执行。

```go
func testRoutine() {
	task_chan := make(chan bool, 3) //100 为 channel长度
	wg := sync.WaitGroup{}
	defer close(task_chan)
	for i := 0; i < math.MaxInt; i++ {
		wg.Add(1)
        fmt.Println("go func ", i)
		task_chan <- true
		go func() {
				<-task_chan
				defer wg.Done()
		}()
	}
	
	wg.Wait()
}
```

- 创建缓冲区大小为 3 的 channel，在没有被接收的情况下，至多发送 3 个消息则被阻塞。通过 channel 控制每次并发的数量。

- 开启协程前，设置 task_chan <- true，若缓存区满了则阻塞

- 协程任务执行完成后就释放缓冲区

- 等待所有的并发都处理结束后则函数结束。其实可以不使用 sync.WaitGroup。因使用 channel 控制并发处理的任务数量可以不用使用等待并发处理结束。



参考资料：

- https://boilingfrog.github.io/2021/04/14/%E6%8E%A7%E5%88%B6goroutine%E7%9A%84%E6%95%B0%E9%87%8F/
