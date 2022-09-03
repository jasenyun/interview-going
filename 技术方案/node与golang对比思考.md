Node 与 Golang 使用的思考

Nodejs 

NodeJS 是一个开源的 JavaScript 运行时环境，用于复杂和简单的可扩展网络应用程序的服务器端开发。

NodeJS 的基础是 Chrome 的 V8 JavaScript 引擎，它在后台在 Chrome 引擎中解析和执行 JavaScript 代码。为了创建和交付可扩展的服务器端应用程序，NodeJS 中有丰富的库。

Golang

Go，它是一种开源、统计类型、多用途、跨平台、可编译和快速的编程语言，通过结合现有编程语言的优点和缺点来解决某些问题。

他们之间的比较

性能

Golang 在性能和原始速度方面比 NodeJS 有一点优势。 Golang 直接组装成机器代码，不需要解释器。 因此，Golang 的性能可以与 C++ 等低级语言相媲美。 Golang 在 IO 操作上与 NodeJS 一样有效。

在性能方面，Golang 略胜过 NodeJS。 优化改进的单线程 NodeJS 提高了效率，V8 JavaScript 引擎确保应用程序在没有任何解释器帮助或要求的情况下运行。

结论 - 在性能方面，Golang 和 NodeJS 不相上下。

扩展性和并发性

在可扩展性方面，谷歌打算创建一种编程语言来创建强大而复杂的企业级生产就绪应用程序。 可扩展性是他们的主要优先事项，他们成功地实现了这一点。

Go 使用 goroutines，它可以实现可靠且简单的线程执行，可以并发且平滑地完成。 由于这些 goroutine，Go 是理想的可扩展编程语言。

Go 使用并发每秒处理超过一千个请求。 正是因为这个特性，Go 比 NodeJS 更具可扩展性和并发性。 同样重要的是要记住 NodeJS 是一个异步单线程 JavaScript 引擎。

CPU 绑定的进程偶尔会停止 NodeJS 单线程设计中的事件循环，并使您的程序运行缓慢。 你最终会得到一个缓慢的应用程序并因此惹恼用户。

结论——Go 在可扩展性和并发性方面明显胜过 NodeJS。

错误处理
NodeJS 使用传统的 throw-catch 错误处理方法，深受程序员喜爱。使用这种传统方法，可以在继续下一步之前迅速显示并修复错误。在比较两种语言的错误处理时，许多 Web 开发人员更喜欢 NodeJS 而不是 Golang，因为他们更习惯于 Node 的 throw-catch 方法。

另一方面，Golang 执行独立的运行时和编译错误检查。虽然运行时问题需要特定的管理，但编译时错误通常与语法相关，并且可以在代码中修复。您必须手动检查此函数的返回值。现在正在进行研究以改进即将推出的 Golang 版本中的处理程序。

结论 - 在这种情况下，NodeJS 和 Golang 是可比的。这是一个平局。

开发工具
在开发工具方面，Go 的库和包比 NodeJS 少。正因为如此，Go 应用程序的开发人员仍然需要执行大量的手动设置工作，需要深入研究。

NodeJS 的大型社区为程序员创建了多个库和包。这个大型社区还在 Stackoverflow 等网站上提供额外的帮助，从而在开发产品时提高生产力。

结论 - 在比较开发人员工具时，NodeJS 轻松获胜。

参考资料：

- https://medium.com/@ongraphtech/nodejs-vs-golang-which-one-to-choose-for-your-next-project-ea2e9710f01a
