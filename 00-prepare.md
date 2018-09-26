# 00-Prepare

## 语言

安装Go,详情见参照Go的官方网站的[安装文档](https://golang.org/doc/install)

这里介绍下Mac下的安装，适用homebrew：

```command
$ brew install go

# 安装完成后 查看版本
$ go version
go version go1.11 darwin/amd64

# 查看 go env
$ go env
```

## 编辑器

* Goland \(IDE）
* Visual Studio Code （本人推荐，易用，插件易安装）
* VI / Emacs \(都是神级编辑器，不过对使用者的要求比较高\)

这里推荐使用 Visual Studio Code

```cmd
$ brew cask install visual-studio-code
```

当编写Go代码时，即遇到 `.go`结尾的文件时，它会自动提示你安装插件，十分简便。

## 体会

Go 是一门新生的语言，而且有个好的公司推广（google），而且这几年的势头也是很猛，是一门改良型的C语言

就 Web 编程而言，相对于Python

* 原生 net/http 支持，可以不需要依赖于第三方包
* 底层 goroutine，高并发
* 静态语言编译，高效
* 特殊的 error 处理机制，基本上一次编译成功，后面很少出错
* 编译之后二进制，易于部署

## Links

* [目录](<README.md>)
* 下一节: [01-Hello-World](<01-hello-world.md>)