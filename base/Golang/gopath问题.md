GOPATH 和 GOROOT



近期在win 系统上重新搭建 Golang 的开发环境，但是在使用 go install 安装依赖包后却没有效果，比如在安装 kratos 后，执行 kratos -v 命令时报错 kratos 命令不存在。

首先需要知道它们之间的区别：

- GOROOT 是 Go 安装的目录。安装完就完全可以不用再去修改。

在 win 系统中，默认是 `c:\GO`,也可以支持自定义。在 Mac 中默认配置地址是`/usr/local/go`。

- GOPATH 是 GO 项目的工作空间和第三方依赖包。就是我们创建 GO 项目放置的位置。因为 Go 的项目其实都是包。GOPATH 是支持设置多个的。

在安装或下载第三方包时，经常会使用的命令是 `go get` 或 `go install`。使用 `go get` 命令下载的包会都下载在 ·`GOPAT` 设置的第一个地址的 src 目录下。使用 `go install` 下载时，在哪个·`GOPATH`中找到了这个包，就会在哪个`GOPATH`下的bin目录生成可执行文件.

进入 `/usr/local/go` 就能看到相关目录文件



![](/Users/jasenyang/Documents/pictures/pro-pic/2022-10-08-23-38-00-image.png)

gopath  的目录下你会看到三个文件目录

- bin ：golang编译可执行文件存放路径

- pkg ：golang编译包时，生成的.a文件存放路径

- src：  源码路径

## 问题排查

大概了解其原理后，可以深入排查下安装了 kratos 后执行还是提示命令不存在的问题。

前提环境是在 window 操作系统下。同时需要知道使用安装命令:`go install github.com/go-kratos/kratos/cmd/kratos/v2@latest`。执行完后按道理会下载到 bin 文件中。

- 首先使用命令 `go env ` 查看当前的 GOPATH 地址。

- 进入 GOPATH 所显示的地址，然后进入其 bin 目录下看是否有 kratos.exe 文件，若存在则说明安装成功。

- 在查看系统和用户的环境变量中有没有配置了 GOPATH和GOROOT ，以及在Path 中是否配置了 GOPATH 目录下bin 文件地址，比如 `%GOPATH%\bin`。

一定要配置环境变量 path = %gopath%\bin。如果有多个 gopath 地址，则需要都配置到 path 地址上，如 `path=d:\\\goproject\\\bin;c:\\\go\\\bin`。

mac 系统中我同样出现了问题。于是我先检查gopath 地址是什么，然后再打开 `～/.bash_profile`文件发现path 地址配置错误了，所以安装第三方包成功了却无法使用。重新修改后执行 `source ~/.bash_profile`后就成功了。


