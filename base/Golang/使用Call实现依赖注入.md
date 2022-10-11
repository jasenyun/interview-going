调研使用 Call 工具包完成依赖注入功能

## 前言

Golang 没有类的概念，也不是面向对象的编程思想。不像 Java 语言中控制反转时面向对象编程的原则之一。使用依赖注入主要是为实现高内聚低耦合的设计思想。在 Golang 常用库中有一个 DI 的工具—wire。它能够根据你的代码生成相应的依赖注入 Go 代码。

在查阅资料的时候发现了一个另外的库包—Call。基于注册表实现参数和函数的依赖注入库。

## 使用 Call

工具包 Call 是 Golang 的一个简单依赖注入第三方库。它将所有的方法和参数都保存在一个映射中，然后就可以随时调用和使用。

可以写些简单的 demo 实践一下，首先需要先安装

```go
go get github.com/rytsh/call
```

又一个函数有一些不同的和相同类型的参数

```go
func GeneratorToken(ctx context.Context, id string)(string, error){
    if isFinished {
        return "", fmt.Errorf("task %s is canceled",id)
    }
    return “this is token ”, nil
}
```

创建完函数后，就可以将该函数和它参数注册到 Call 中。

您可以按任何顺序添加函数和参数。 它只是添加一个内部映射的关系。

```go
reg :=call.NewReg() // 获取新的注册表

reg.AddFunction("GeneratorToken", GeneratorToken,"context","id") // 向注册表中加入函数和参数
```

现在使用 GeneratorToken 函数时：

```go
// 注册表传入对应的参数然后调用
reg.AddArgument("context",context.Background())
   .AddArgument("id","738293232")
result, err := reg.Call("GeneratorToken")
```

此函数返回称为returns 的 GeneratorToken 函数的所有返回，如果Call 函数不能调用 GeneratorToken 函数，则返回err。

返回变量类型是 []interface{}，所以我们应该转换并得到我们想要的。 在我们的示例中，GeneratorToken 函数返回一种字符串类型。

以上的方式通过注册的方式添加方法和参数，然后在调用真正的函数时需要先传入具体的参数。当然，也支持直接调用的时候传入实参。主要使用 CallWithArgs 的方法：

```go
result, err := reg.CallWithArgs("GeneratorToken",context.Background(),"id_237829372")
```

如果函数的参数长度和类型不匹配，它将返回错误。

参考资料：

- https://github.com/rytsh/call
