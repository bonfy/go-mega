# 02-Template Basic

学习完 [第一章](01-hello-world.md) 之后，你已经拥有了一个简单，不过可以成功运行Web应用

本章将沿用这个应用，在此之上，加入模版渲染，使得页面更丰富

_本章的GitHub链接为：_ [Source](https://github.com/bonfy/go-mega-code/tree/02-Template) [Diff](https://github.com/bonfy/go-mega-code/compare/01-Hello-World...02-Template)

## 什么是模板

微博应用程序的主页会有一个欢迎用户的标题。虽然目前的应用程序还没有实现用户概念，但这不妨碍我使用一个 Go Stuct 来模拟一个用户，由于Go是静态语言，先定义 User 的结构，然后再初始化 user ，如下所示：

```go
type User struct {
    Username string
}

user := User{Username: "bonfy"}
```

创建模拟对象是一项实用的技术，它可以让你专注于应用程序的一部分，而无需为系统中尚不存在的其他部分分心。 在设计应用程序主页的时候，我可不希望因为没有一个用户系统来分散我的注意力，因此我使用了模拟用户对象，来继续接下来的工作。

原先的视图函数返回简单的字符串，我现在要将其扩展为包含完整HTML页面元素的字符串，如下所示：

```go
package main

import (
    "html/template"
    "net/http"
)

// User struct
type User struct {
    Username string
}

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        user := User{Username: "bonfy"}
        tpl, _ := template.New("").Parse(`<html>
            <head>
                <title>Home Page - Bonfy</title>
            </head>
            <body>
                <h1>Hello, {{.Username}}!</h1>
            </body>
        </html>`)
        tpl.Execute(w, &user)
    })
    http.ListenAndServe(":8888", nil)
}
```

对HTML标记语言不熟悉的话，建议阅读一下Wikipedia上的简介[HTML Markup](https://en.wikipedia.org/wiki/HTML#Markup)

利用上述的代码更新这个视图函数，然后再次运行 (这里并没有flask那样子的debug mode，每次更新都必须重新执行)

```cmd
$ go run main.go
```

在浏览器打开它的URL看看结果

![02-01](02-01.png)

本小节 [Diff](https://github.com/bonfy/go-mega-code/commit/1cf55f62ae8ee34bb9fec64508398480b9168e2c)


我们暂时将 template 以文本方式耦合在代码中，随着我们项目的扩大，这个不利于管理，而且如果公司有前端支持的话，模板的工作很大可能会有前端设计，所以我们有必要对其进行解偶。我们打算将模板文件放入 templates 的文件夹

建立 templates 文件夹

```cmd
$ mkdir templates
```

将模板的内容移到 templates文件夹下

templates/index.html

```html
<html>
    <head>
        <title>Home Page - Bonfy</title>
    </head>
    <body>
        <h1>Hello, {{.Username}}!</h1>
    </body>
</html>
```

然后将 template 文本从代码中移除

main.go

```go
package main

import (
    "html/template"
    "net/http"
)

// User struct
type User struct {
    Username string
}

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        user := User{Username: "bonfy"}
        tpl, _ := template.ParseFiles("templates/index.html")
        tpl.Execute(w, &user)
    })
    http.ListenAndServe(":8888", nil)
}
```

然后运行，在浏览器中打开，结果是与刚才一样的，不过代码的组织结构却更清晰了。

本小节 [Diff](https://github.com/bonfy/go-mega-code/commit/cca162aa6510f6d8bf96cb21ef8aed5ea0a1a1a1)

## 模板的常用操作

### 条件语句

在 index.html 中加入条件语句

templates/index.html

```html
<html>
    <head>
        {{if .Title}}
            <title>{{.Title}} - blog</title>
        {{else}}
            <title>Welcome to blog!</title>
        {{end}}
    </head>
    <body>
        <h1>Hello, {{.User.Username}}!</h1>
    </body>
</html>
```

由于 User 没有 Title字段，所以我们引入新的 IndexViewModel 来具体处理所有与View对应的Model

main.go

```go
package main

import (
    "html/template"
    "net/http"
)

// User struct
type User struct {
    Username string
}

// IndexViewModel struct
type IndexViewModel struct {
    Title string
    User  User
}

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        user := User{Username: "bonfy"}
        v := IndexViewModel{Title: "Homepage", User: user}
        tpl, _ := template.ParseFiles("templates/index.html")
        tpl.Execute(w, &v)
    })
    http.ListenAndServe(":8888", nil)
}
```

本小节 [Diff](https://github.com/bonfy/go-mega-code/commit/9df5fb2f1f7d400513e3877aad0704a5e69366ac)

### 循环

在 index.html 中加入循环

```html
<html>
    <head>
        {{if .Title}}
            <title>{{.Title}} - blog</title>
        {{else}}
            <title>Welcome to blog!</title>
        {{end}}
    </head>
    <body>
        <h1>Hello, {{.User.Username}}!</h1>
        {{range .Posts}}
            <div><p>{{ .User.Username }} says: <b>{{ .Body }}</b></p></div>
        {{end}}
    </body>
</html>
```

在 main.go 的 IndexViewModel 中加入 Post

```go
package main

import (
    "html/template"
    "net/http"
)

// User struct
type User struct {
    Username string
}

// Post struct
type Post struct {
    User User
    Body string
}

// IndexViewModel struct
type IndexViewModel struct {
    Title string
    User  User
    Posts []Post
}

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        u1 := User{Username: "bonfy"}
        u2 := User{Username: "rene"}

        posts := []Post{
            Post{User: u1, Body: "Beautiful day in Portland!"},
            Post{User: u2, Body: "The Avengers movie was so cool!"},
        }

        v := IndexViewModel{Title: "Homepage", User: u1, Posts: posts}
        tpl, _ := template.ParseFiles("templates/index.html")
        tpl.Execute(w, &v)
    })
    http.ListenAndServe(":8888", nil)
}
```
本小节 [Diff](https://github.com/bonfy/go-mega-code/commit/50e5028870d4ab7918d391007162c4387540f165)

运行效果图

![02-02](02-02.png)


## Links

  * [目录](README.md)
  * 上一节: [01-Hello-World](01-hello-world.md)


