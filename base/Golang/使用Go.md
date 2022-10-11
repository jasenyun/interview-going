使用 Go 实现一个共享库

Don't Repeat Yourself 不要重复自己，这是软件开发的一个基本原则，目的就是减少重复。但是在系统中不同的部分，可能会有不同的业务逻辑，若使用相同的功能来解决不同上下文中的问题，那应该使用公共方法来防止代码重复吗？

## 共享库

使用共享库可以协助我们管理代码重用的问题，但是需要考虑共享库依赖和变更控制的问题。

如果有几个服务都使用了一个共享库：

- 共享库发生变更，每个服务都需要使用新版本共享库。旧版本的共享库如果被弃用就会导致服务不可使用，所以在这种情况下，每次发生变更时就需要重新测试和重新部署服务。

- 服务发生变更就需要更改共享库，这样共享库就失去通用性的特征。说明共享库包含服务相关的服务的业务逻辑。但这样是错误的，共享库不应该与服务有任何关联的代码块

如果有几个服务都有使用多个共享库：

不同的服务使用不同的库，依赖多个库。共享库越多，依赖就越多，就会导致依赖管理变得困难。

## 创建库

以Go 操作 RabbitMQ 创建一个共享库为例子，需要初始化 MQ实例，创建连接，重连机制、关闭，消费者通道，创建队列，队列绑定交换机等等功能。

```go
type RabbitMQ struct {
    connection   *amqp.Connection
    channel      *amqp.Channel
    connURL      string
    errCh        <-chan *amqp.Error
    messageChan  <-chan amqp.Delivery
    retryAttempt int
}

// 初始化 MQ 实例
func NewRabbitMQ(options RabbitMQOptions) (*RabbitMQ, error) {
    rabbitMQ := &RabbitMQ{
        connURL:      options.URL,
        retryAttempt: options.RetryAttempt,
    }

    if err := rabbitMQ.connect(); err != nil {
        return nil, err
    }

    return rabbitMQ, nil
}


func (rmq *RabbitMQ) connect() error {}
func (rmq *RabbitMQ) reconnect() {}
func (rmq *RabbitMQ) Close() {}
func (rmq *RabbitMQ) ConsumeMessageChannel() (jsonBytes []byte, err error) {
func (rmq *RabbitMQ) CreateQueue(name string, durable bool, autoDelete bool, exclusive bool, noWait bool, args map[string]interface{}) (amqp.Queue, error) }
```

创建完之后需要再创建一个合适的标签 tag 或版本号来共享该代码。使用过 github 都可以在 release 中看到版本号和 tag 。

## 使用库

首先，我们需要使用 go get 安装这个共享库。 之后，我们必须在我们的项目中导入我们的库，如下所示。

```go
package main

import (
    "fmt"
    rabbitmq_sdk "github.com/nanlv/rabbitmq-sdk"
)

func main() {
    fmt.Println("connect MQ")

    options := rabbitmq_sdk.RabbitMQOptions{
        URL:          "amqp://root:root@localhost:5672/",
        RetryAttempt: 5,
    }

    rabbitMQ, err := rabbitmq_sdk.NewRabbitMQ(options)
    if err != nil {
        return
    }

    queue, err := rabbitMQ.CreateQueue("queue1", true, true, false, false, nil)
    if err != nil {
        fmt.Println(err.Error())
        return
    }
}
```

如果你想测试你的库而不发布它，你可以使用 replace 指令。 将依赖库的地址指向本地的文件的地址。

```go
replace github.com/nanlv/rabbitmq-sdk v1.0.0 => 项目在本地的地址
```

## 总结

若两个部分使用相同的库可能会增加依赖和耦合。因为它们具有不同的业务逻辑，在业务独立演进时，就会增加耦合和维护成本。耦合是衡量两个组件间了解和互相依赖程度的指标。耦合度越高，依赖性越高。

共享库可以解决代码重复的问题。但需要做好版本管理和变更控制。使用 Go 可以创建一个共享库然后上传到 github。就可以将其作为一个共享库使用。
