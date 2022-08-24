# 轻松学会 Golang 单元测试和基准测试

## 前言

多人协作的项目里，要保证代码的质量，自然离不开单元测试。开发完一个功能后肯定要对所写的代码进行测试，测试没有问题之后再合并到代码库供他人使用。如果强行合并到代码库可能会影响其他人开发，被上线的话肯定也会导致线上 Bug ，影响用户使用。

所以，单元测试也是一个很重要的事情。单元测试是指在开发中，对一个函数或模块的测试。其强调的是对单元进行测试。

## Go 单元测试

Go 语言提供了单元测试的框架，只要遵循其规则即可：

- 测试文件命名：
  
  - 单元测试的代码文件都必须以 _test.go 结尾，这样才能被 Go 语言测试工具识别
  
  - 单元测试的文件命名都与被测试函数所在的 go 文件的文件名一样，然后再加 _test.go。比如 main.go 的测试文件 main_test.go

- 测试函数命名：
  
  - 单元测试的函数名必须以 Test 开头，再加上要测试函数名，且必须是公有的。比如main.go中有函数 ``func add(){}``,   其函数名应为 `TestAdd`
  
  - 测试函数的签名必须接收一个指向 testing.T 类型的指针，并且不能返回任何值

```bash
# main.go
func Add(){
    // to do something
}

# main_test.go
func TestAdd(t *testing.T) {
	result := Add()
	if result == 3 {
		println("success")
	} else {
		println("error")
	}
}
```

根据以上规则，就可以进行对某测试文件执行命令，进行单元测试:

```bash
go test -v ./main_test.go
```

如果显示的测试结果有 PASS 标记，说明单元测试通过。

## 单元测试覆盖率

函数是否被全面测试，还需要覆盖率进行检测。单元测试命令增加 --coverprofile 标记，就可以得到一个单元测试覆盖文件，且会在控制台打印出代码覆盖率是多少。

```bash
go test -v --coverprofile=main.cover ./main_test.go
```

Go 框架还可以生成 html 文件的覆盖率报告，这样就可以对单元测试覆盖率的结果更清晰，更明白。

```bash
go tool cover -html=main.cover -o=main.html
```

打开 html 文件就可以看到红色标记是没有被覆盖到，绿色是被测试到的。通过单元测试覆盖率很清晰的可以看到代码被测试覆盖的情况。

![](/Users/jasenyang/Documents/pictures/pro-pic/2022-08-22-03-12-44-image.png)

以上是简单的功能的单元测试，验证功能逻辑的正确。但有时候还有性能的要求，这时就可以使用基准测试来评估代码的性能。

## 基准测试

基准测试是一项用于测试和评估软件性能指标的方法，主要测试代码的性能。基准测试的规则和单元测试的规则是不一样的：

- 基准测试函数必须以 Benchmark 开头，且必须是可导出的

- 函数的签名必须接收一个指向 testing.B 类型的指针，并且不能返回任何值；

- 最后的 for 循环很重要，被测试的代码要放到循环里；

- b.N 是基准测试框架提供的，表示循环的次数，因为需要反复调用测试的代码，才可以评估性能。

```go
func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Add()
	}
}
```

写完基准测试，就可以执行命令进行测试：

```bash
go test -bench=. ./
```

使用 go test 命令，再加上 -bench 这个 Flag，它接受一个表达式作为参数，以匹配基准测试的函数，"."表示运行所有基准测试。

![](/Users/jasenyang/Documents/pictures/pro-pic/2022-08-22-03-23-54-image.png)

BenchmarkAdd-10, 其中的 -10 是运行基准测试时对应的 GOMAXPROCS 的值。基准测试的时间默认是 1 秒，也就是 1 秒调用 1000000000 次、每次调用花费 311 纳秒。如果想让测试运行的时间更长，可以通过 -benchtime 指定，比如 3 秒。

```bash
go test -bench=. -benchtime=3s ./
```

- 重置计时方法

进行基准测试之前，需要进行一些数据准备，如构建测试数据，而这部分准备工作不属于性能测试计算范围内所以需要排除在外。通过使用充值计数器 ResetTimer重新计算。也支持使用 StartTimer 和 StopTimer 方法，控制何时开始计时何时结束。

-  内存统计

内存统计主要是统计每次操作分配内存的次数和分配的字节数。使用 ReportAllocs() 方法

```go
func BenchmarkAdd(b *testing.B) {
	b.ResetTimer() // 重置计时时间
    b.ReportAllocs() // 内存统计
	for i := 0; i < b.N; i++ {
		Add()
	}
}
```

对以上命令执行后可在控制台上得到结果。多了两个指标。

- 第一个表示：每次操作分配多少字节内存

- 第二个表示：每次操作分配内存的次数

![](/Users/jasenyang/Documents/pictures/pro-pic/2022-08-22-03-59-17-image.png)

两个指标没有统一标准区说明越小越好还是越大越好，主要还是需要根据业务场景来判断的。

- 并发基准测试

在并发的情况下，Go 也支持基准测试。Go 语言通过 RunParallel 方法运行并发基准测试。创建多个 goroutine 然后将 b.N 分配给这些 goroutine 执行。

```go
func BenchmarkAdd(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			Add()
		}
	})
}
```

单元测试可以保证代码质量，但其也不是万能的，还需要 code Review 和人工测试才能更好的保证代码的质量。

## CI/CD 的自动化测试

在CI/CD 加入单元测试覆盖率，当覆盖率达到一定值时才允许通过，然后继续编译部署。通过加入单元测试节点







参考资料：

- https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/22%20%E8%AE%B2%E9%80%9A%E5%85%B3%20Go%20%E8%AF%AD%E8%A8%80-%E5%AE%8C/18%20%20%E8%B4%A8%E9%87%8F%E4%BF%9D%E8%AF%81%EF%BC%9AGo%20%E8%AF%AD%E8%A8%80%E5%A6%82%E4%BD%95%E9%80%9A%E8%BF%87%E6%B5%8B%E8%AF%95%E4%BF%9D%E8%AF%81%E8%B4%A8%E9%87%8F%EF%BC%9F.md
