必知必会的 Golang Context 模块

## 前言

在写 Golang 项目时发现 context 模块几乎是无处不在，但在使用和理解的时候还是对其的理解总是模糊不清。context 在维护程序性能起到至关重要的作用。

## 作用

一个顾客进入饭店并进行点单。菜单确认后就会由服务员派单到后厨的厨师进行制作餐食。但当客户突然不想吃决定离开，这该怎么办？毫无疑问，服务员必须阻止厨师制作，防止食材浪费。

Context 模块作用就跟服务员一样，它是传递给你的函数或 Goroutines 的参数，并可以在不需要的时候立即停止它们。

## 是什么

context 常用的场景就是客户端终止与服务端的连接。但是当服务端正在处理一些比较重要的任务时该怎么办？

context 模块允许这些进程在不再需要时立即停止。

context 模块的使用主要有三个部分：

- 监听取消事件

- 发出取消事件

- 传递请求范围数据

看下 Context 接口的源代码：

```go
type Context interface {
    Done() <- chan struct{}
    Err() error
	
    Deadline() (deadline time.Time, ok bool)
    Value(key interface{}) interface{}
}
```

context 类型是实现了四个方法的的接口：

- Done 方法会在取消 context 时返回一个空struct 结构的channel 。

- Err 方法在取消时返回非零错误，否则返回零值。

- Deadline 方法返回当任务完成时 context 应该被取消的时间。 未设置截止日期时，截止日期返回 ok==false。

- Value 方法返回与 key 关联的 Context的值，如果没有返回 nil。使用相同 key 连续调用时 Value 方法返回的是同样的结果。

## 功能

### 监听取消事件

Done 方法和 Err 方法在会在context 取消时触发。

```go
func handler(r *http.Request){
    ctx := r.Context()
    
    select {
        case <-time.After(2*time.second):
          fmt.Println("request processed")
        case <- ctx.Done():
          fmt.Println("request Cancelled")
          return
    }
}
```

在上面的方法中使用 time.After() 来模拟一个需要两秒钟来处理请求的函数。如果context 在2秒内被取消时ctx.Done() 的 channel 就会接收到空的struct 结构，就会退出函数。

```go
func handler(ctx context.Context){
    if ctx.Err() != nil {
        fmt.Println("context is Cancelled")
        return
    }
     fmt.Println("request processed")
}
```

在执行一些关键业务逻辑时可以检查 ctx.Err() 的错误。如果 context被取消了，上面的函数就会暂停并返回。

### 发出取消事件

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)

func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
```

context 模块提供了三个返回 CancelFunc 的函数。

调用 cancelFunc 会向 ctx.Done() 通道发出一个空结构体，并通知正在侦听它的下游函数。

### 传递请求范围数据

```go
func WithValue(parent Context, key, val interface{}) Context
```

 像传递变量一样，请求范围数据可以使用 WithValue 函数标记该变量。WithValue 函数将值添加到 ctx 变量中的键，而 Value 函数检索给定键的值。



参考资料：

- https://betterprogramming.pub/understanding-context-in-golang-7f574d9d94e0
