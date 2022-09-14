pre-commit hook 规范 Golang 项目开发

在进行 NodeJs 开发时经常会使用 TsLint 或 EsLint ，进行代码格式化，设置规则标记不规范的代码，或自动修复一些结构或风格问题。但是 Golang 项目目前没有比较合适的工具可以充当这部分的功能。

在使用 Git 进行项目版本管理时 git  hook 的 pre-commit 功能，可以很好应用到项目里。它主要在服务端接收提交对象，推送到服务器之前调用。

## pre-commit  安装

在使用之前，需要先安装 pre-commit ，然后一步一步将 hooks 添加到你的项目里，本文中，主要使用 brew 工具安装。

```shell
brew install pre-commit

# 检查是否安装成功，可以查看其版本号
re-commit --version
```

## pre-commit 配置

 在你项目的根目录上新增一个文件，命名为 `.pre-commit-config.yaml`。下一步，将基本的项目检查器 hook 添加到 pre-commit 文件中。

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0
    hooks:
      - id: trailing-whitespace
        name: Trim Trailing Whitespace
        description: This hook trims trailing whitespace.
        entry: trailing-whitespace-fixer
        language: golang
        types: [ text ]
  - repo: https://github.com/dnephin/pre-commit-golang
    rev: v0.5.0
    hooks:
      - id: go-fmt
      - id: go-imports
      - id: no-go-testing
      - id: golangci-lint
      - id: go-unit-tests

```

- repos  一系列仓库映射

- repo  在 git clone 时的来源仓库地址 repository url

- rev 将要 clone 的版本号或者 tag

- hooks 是一系列 hook 映射

在上面的 hooks 中有 2个repo 仓库地址，一个是 pre-commit hooks， 是项目预提交前的检查 hook；一个是 golang hooks，是 golang 项目中会用到一些检查 hooks

**pre-commit-hooks**

- trailing-whitespace 处理行尾和新行的任何空格。

- end-of-file-fixer 删除整个文件项目的 EOF。

- check-yaml 修复 yaml 格式文件。

- check-added-large-files 这个工作会让你知道哪个文件的文件很大。

**Golang hooks**

- go-fmt trigger 自动格式化整个项目的 Go 源代码。

- go-imports 会自动更新您的 Go 导入行（添加缺失并删除未引用的导入）。

- no-go-testing 发现并警告哪些文件没有测试覆盖。

- golangci-lint 运行 linter。
  
  golangci-lint 是支持创建自定义的 linter 规则，只要把自定义规则放在 `.golanglint.yaml`。详细可查看：[https://golangci-lint.run/](https://golangci-lint.run/)

- go-unit-tests 运行 go test 命令。

hook 脚本配置完成后，需要执行命令 `pre-commit install` 启动 hook 脚本。然后就可以通过 git commit 命令提交代码来触发钩子。

为保证团队内 commit 代码时规范化，可以将 hook 运用到团队项目上。

```shell
npm install -g @commitlint/cli @commitlint/config-conventional
```





参考资料：

- https://pre-commit.com/#plugins

- https://golangci-lint.run/
