# 用 Gin 直接渲染 html 页面



## 跨域问题

Gin 是 Golang 最流行的 Web 框架之一。构建后端使用 gin 可以很简单的搭建出来。但前端调用后端时，如果url 非同源的就会发生跨域的问题。

常见的跨域解决方案：

- 只需要在服务端设置 Access-Control-Allow-Origin即可，前端无须设置

- 使用 JSONP 请求服务端，但该方式只能使用 get 请求

- 使用 nginx 反向代理解决跨域

## Gin 解决跨域

Gin 解决跨域的方式就是使用中间件，对所有的请求之前进行拦截，然后设置 header 允许跨域。

```go
c.Header("Access-Control-Allow-Origin", "*")  
c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE, UPDATE")
c.Header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept, Authorization")
c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Cache-Control, Content-Language, Content-Type")
c.Header("Access-Control-Allow-Credentials", "true")
```

使用时，因对所有请求之前进行拦截，所以必须放在路由前。

```go
func main(){
    r := gin.Default()
    r.use(cors()) // 在所有路由设置之前
    r.GET("/",hello);
}
```

如果不设置跨域，我们可以直接使用 Gin 渲染 HTML 。让它们的域名都在同一地址下。

## Gin 渲染 HTML

直接看下我的 Demo 代码：

```go
func main(){
	r := gin.Default()
	// 静态资源
	r.Static("/assets", "./assets")
	// 前端模块
	r.LoadHTMLGlob("/web/html/*")
	r.GET("/web/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.html", gin.H{})
	})
	r.Run(":8080")
}
```

- 首先创建了一个名为 r 的默认 Gin 路由器。除了基本功能外，默认的 Gin 路由器还可以使用中间件。

- 我在 assets 文件下放置了一些静态资源。所以要让 gin 知道该目录下的静态资源，就需要使用 Static() 函数，就可以访问任何静态资源。

- 所有满足模式 web/html/*.html 的模板都由 LoadHTMLGlob() 函数加载。这种模式意味着模板文件应该具有 .html 扩展名并位于 /web/html 目录中。

- 接下来就是告诉 Gin 路由器在 URL 路径 / 处接受 HTTP GET 方法请求。当收到请求时，Gin 会发送一条 HTTP OK 状态消息，并使用 gin.H{} 括号内提供的数据呈现 index.html 模板。在这种情况下，数据只包含一个键/值对，键称为内容。

- 然后在端口 8080 运行

这样，就可以正常访问：http://localhost:8080/web/index 就可以渲染 index.html 页面的内容。




