Go 中的 nil 真的nil 了吗？

在 DB 操作时，经常会这样操作：开启事务，进行更新操作，然后提交事务。在进行更新时，可能会出现错误。比如未找到记录，输入未找到元数据，所以容错是必不可少的步骤。

我们可以统一处理 HTTP 响应返回格式，如下所示：

```go
type ResponseError struct {
    HTTPResponseCode int
}

func (resError ResponseError) Error() string {
    return fmt.Sprintf("code: %v",e.HTTPResponseCode)
}
```

发生什么错误
