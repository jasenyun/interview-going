Golang 中对 main.go 文件的最佳实践

golang 项目的入口是 main.go 的 main 函数。一个服务只有一个 main 函数。整个 main.go 可以认为是单例模式实现，其中可以启动一堆其他的 goroutine。作为项目的主文件，我们也应该尽量以最佳方式实践：

- 尽量保证 main.go 小点。可以参考该[示例](https://github.com/influxdata/influxdb/blob/master/cmd/influxd/main.go)：

- 尽量不在 main.go 中实现核心功能
  
  - 用 main.go 编写的结构和函数不能导出。其他地方要使用时，则需要重新定义或重构
  
  - 将结构和函数编写在自己的包中时，可以很容易地对其进行测试。
  
  - 使用接口（如果可能）。紧密耦合的 main.go 通常会导致瓶颈。

- main.go 过大容易导致不容易理解，通过保持较小，提高代码的可维护性。
  
  -  main.go 代码量太大的话，随着时间的推移通常变得难以使用
  
  - 业务逻辑太多在 main.go 文件中的话，可读性差，不利于其他开发人员

- 排除复杂性：main() 和 main.go 应该是您尝试运行并打包复杂性的进程的最高抽象级别。

- 测试将变得非常规
  
  - main_test.go 可以用来编写测试。但是为 main.go 编写测试并不是惯例。但是当你想测试你的函数时，可以根据模块进行拆分到鸽子的包中，然后根据包进行测试。

所以，mian.go 尽量只处理一些配置读取和实例化核心实现。不应该有太多的复杂的业务核心实现。

参考资料：

- https://github.com/golang-standards/project-layout/blob/master/README_zh.md

- https://medium.com/@druhin.bala/best-practices-for-main-go-in-golang-6f4ef1888f42
