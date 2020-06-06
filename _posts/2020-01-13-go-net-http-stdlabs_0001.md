---
title: go 语言学习笔记之(net/http标准库)[0001]
key: 2020-01-13
tags: go go-lang net/http
---

As The Titile



本文是我学习《 Go Web编程》的学习笔记.





<!--more-->



首先, net/http 标准库可以分为客户端和服务区段两个部分:



- Client, Response, Header, Request 和Cookie 对客户端进行支持
- Server, ServeMux, Handler/HandleFunc, ResponseWriter, Header, Request 和 Cookie 则对服务器进程支持.



## 使用 Go 构件服务器



### Go Web 服务器



使用 Go 创建一个服务器的步骤非常简单, 只要调用 `ListenAndServe` 并传输网络地址以及负责处理请求的处理器 (`handler`)作为参数就可以.



```go
package main

import (
    "net/http"
)

// 如果网络地址参数为空串, 
// 那么默认服务器使用 80 端口进行网络监听
// 如果处理器参数为 `nil`, 
// 那么服务器将使用默认的多路复用器 `DefaultServeMux`
func main() {
    http.ListenAndServe("", nil)
}
```



我们除了可以通过 `ListenAndServe` 的参数对服务器进的网络地址和处理器进行配置外,  还可以通过 Server 结构体对服务器进行更详细的配置, 其中包括为请求读取操作设置超时时间, 为响应写入操作设置超时时间以及为 Server 结构体设置错误日志记录器等.





```go
package main

import (
    "net/http"
)


func main() {
    server := http.Server {
        Addr : "127.0.0.1:8080",
        Handler: nil,
    }
    server.ListenAndServe()
}

// 这些是 Server 结构体所有可选的配置选项.
type Server struct {
    Addr string
    Handler Handler
    ReadTimeout time.Duration
    WriteTimeout time.Duration
    MaxHeaderBytes int
    TLSConfig *tls.Config
    TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
    ConnState func(net.Conn, ConnState)
    ErrorLog *log.Logger
}
```



### 通过 HTTPS 提供服务



HTTPS 实际上就是将 HTTP 通信放到 SSL 之上进行, 通过使用 `ListenAndServeTLS` 函数, 我们可以提供 HTTPS 服务.



```go
package main

import (
    "net/http"
)

func main() {
    server := http.Server {
        Addr := "172.0.0.1:8080",
        Handler := nil,
    }
    // cert.pem 文件是 SSL 证书
    // key.pem 是服务器的私钥 private key
    server.ListenAndServeTLS("cert.pem", "key.pem")
}
```



使用 go 生成 SSL 证书以及服务器私钥的具体代码, (这里在回去好好看看书上的内容)



## 处理器和处理函数



前面的 Web 服务器未实现任何功能, 因为我们没有未服务器编写任何处理器.



在 Go 语言中, 一个处理器就是一个用于 `ServeHTTP` 方法的接口, 这个 `ServeHTTP` 方法接收两个参数:



- 一个 `ResponseWriter` 接口
- 一个指向 `Request` 结构体的指针



```
ServeHTTP(http.ResponseWriter, *http.Request)
```



> 一个问题: 既然 ListenAndServe 接收的第二个参数是一个处理器, 那么为何他的默认值却是多路复用器呢? (`DefaultServeMux`)



这是因为 DefaultServeMux 多路复用器是 ServeMux 结构体的一个实例, 而后者也拥有上面提到的 ServeHTTP 方法, 并且这个方法的签名与成为处理器所需的签名完全一致.

即 DefaultServeMux 既是 ServerMux 结构体的实例, 也是 Handler 结构体的实例, 因此 DefaultServeMux 不仅是一个多路复用器,  还是一个处理器.


DefaultServeMux 是一个特殊的处理器, 它唯一要做的就是根据请求的 URL 将请求重定向到不同的处理器. 



```go
package main

import (
    "fmt"
    "net/http"
)

type MyHandler struct {}

func (h *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello World")
}

func main() {
    handler := MyHandler{}
    server := http.Server{
        Addr : "localhost:8080",
        Handler: &handler,
    }
    server.ListenAndServe()
}

```



这样, 不管怎样访问, 结构都是 "hello world".



这是因为我们创建了一个处理器并将她与服务器进行了绑定, 因此替换了原来的默认的多路复用器. 就使得服务器不会再通过 URL 匹配来将请求路由至不同的处理器, 



这也是我么在 Web 应用中使用多路复用器的原因.



### 使用多个处理器



大多数情况下, 我们希望用多个处理器去处理不同的 URL.  为做到这一点, 我们不在 Server 接头体的 Handler 字段中指定处理器, 而是让服务器使用默认的 DefaultServeMux 作为处理器, 然后通过 `http.Handle` 函数将处理器绑定至 DefaultServeMux. 



需要注意的是, 虽然 Handle 函数来源于 http 包, 但是它实际上是 ServeMux 结构的方法: 这些函数是为了操作便利而创建的, 调用他们等同于调用 DefaultServeMux 的某个方法. 比如说: 调用 `http.Handle` 实际上就是在调用 `DefaultServeMux` 的 `Handle` 方法.





```go
package main

import (
    "fmt"
    "net/http"
)

type HelloHandle struct {}

func (h *HelloHandle) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello!")
}

type WorldHandler struct {}
funcn (h *WorldHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "World!")
}

func main() {
    hello := HelloHandler{}
    world := WorldHandler{}
    server := http.Server {
        Addr: "localhost:8880",
    }
    
    http.Handle("/hello", &hello)
    http.Handle("/world", &world)
    
    server.ListenAndServe()
}
```





### 处理器函数



What's 处理器函数?



处理器函数实际上就是与处理器拥有相同行为的函数:



这些函数与 ServeHTTP 方法拥有相同的签名, 也就是说, 他们接收 ResponseWriter 和指向 Request 结构的指针作为参数.



处理器函数的实现原理是这样的: Go 语言拥有一种 HandlerFunc 函数类型, 它可以把一个带有正确签名的函数 `f`转换成一个导游方法 `f` 的 Handler, 比如:



```go
func hello(w http.ResponseWriiter, r *http.Request) {
    fmt.Fprintf(w, "Hello!")
}

//就可以把 helloHandler 设置成一个 Handler.
helloHandler := HandleFunc(hello)


```





### 串联多个处理器(处理器函数)



一个例子:



```go
package main

import (
    "fmt"
    "net/http"
    "reflect"
    "runtime"
)

// 
func hello(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello!")
}

func log(h http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        name := runtime.FuncForPc(reflect.ValueOf(h).Pointer()).me()
        fmt.Println("Handler function called - ", name)
        h(w, r)
    }
}

func main() {
    server := http.Server {
        Addr: "localhost:8080",
    }
    http.HandleFunc("/hello", log(hello))
    server.ListenAndServe()
}
```



`log` 函数接收一个 `HandlerFunc` 类型的函数作为参数, 然后返回另一个 `HandlerFunc` 类型的函数作为返回值.



**`hello` 函数就是一个 `HandlerFunc` 类型的函数**



因此可以将 hello 函数传递个 log 函数, 于是就串联起了 log 函数和 hello 函数.



这种做法有时候也成为管道处理( pipeline processing).



... 

更多的去书里面查.



### ServeMux 和  DefaultServeMux



ServeMux 是一个 HTTP 请求多路复用器, 他负责接收 HTTP 请求并根据请求中的 URL 将请求重定向到正确的处理器.



DefaultServeMux 实际上是 ServeMux 的一个实例. 并且所有引入了 `net/http` 标准库的程序都可以使用这个实例.



如果被绑定的 URL 不是以 `/` 结尾, 那么它会与完全相同的 URL 匹配: 例如: `/hello`



如果被绑定的 URL 是以 `/` 结尾, 那么及时请求 的 URL 只有前缀部分与被绑定 URL 相同, ServeMux 也会认定这两个 URL 是匹配的, 例如 `/hello/` 



### 使用其他多路复用器



我们来了解一下一个高效的轻量级的第三方多路复用器 -- **HttpRouter**

https://github.com/julienschmidt/httprouter





```go
package main

import (
    "fmt"
    "net/http"
    "github.com/julienschmidt/httprouter"
)

func hello(w http.ResponseWriter, r *http.Request, p httprouter.Params) {
    fmt.Fprintf(w, "hello, %s\n", p.ByName("name"))
}

func main() {
    mux := httprouter.New()
    mux.Get("/hello/:name", hello)
    
    server := http.Server{
        Addr: "Localhost:8080",
        Handler: mux,
    }
    server.ListenAndServe()
}
```





## 使用 HTTP/2



如果使用 HTTPS 模式启动服务器, 那么服务器将默认使用 HTTP/2, 但是, 在默认情况下,  版本低于 1.有的 Go 不会安装 http2 包,

需要手动执行命令来获取:



```
$ go get "golang.org/x/net/http2"
```



后面的看书去吧.

...



### Request 结构



Request 结构体表示一个由客户端发送的 HTTP 请求报文.



Request 结构主要由以下部分组成:



- URL 
- Header
- Body
- Form, PostForm, MultipartForm 



### 请求 URL



Request 结构中的 URL 字段用于表示请求行中包含的 URL 



```go
type URL struct {
    Scheme string
    Opaque string
    User *Userinfo
    Host string
    Path string
    RawQuery string
    Fragment string
}
```





### Header



一个 Header 类型的实例就是一个映射, 这个映射的 Key 为 string, value 则是由任意多个字符串组成的 Slice.

```
func headers(w http.ResponseWriter, r *http.Request) {
    h := r.Header
    fmt.Fprintln(w, h)
    // 返回切片
    // [gzip, deflate]
    ae := r.Header["Accept-Encoding"] 
    // 返回字符串
    // gzip, deflate
    ae2 := r.Header.Get("Accept-Encoding") 
}
```





### Body



请求和响应的主体都由 Request 结构的 Body 字段表示, 这个字段是一个 `io.ReadCloser` 接口, 该接口即包含了 `Reader` 接口, 也包含了 `Closer` 接口. 其中 `Reader` 接口拥有 `Read` 方法, , 这个方法接收一个字节切片作为输入, 并在执行之后返回被读取内容的字节数以及一个可选的错误作为结果.

`Closer` 接口则拥有 `Close` 方法, 这个方法不接受任何参数, 但会在出错时返回一个错误.



同时包含 `Reader` 接口和 `Closer` 接口意味着用户可以对 Body 字段调用 `Read` 方法和 `Close` 方法.





```go
func body(w http.ResponseWriter, r *http.Request) {
    len := r.ContentLength
    body := make([]byte, len)
    r.Body.Read(body)
    fmt.Fprintln(w, string(body))
}
```



## HTML Form



HTML 表单的内容类型(**Content Type**)决定了 POST 请求在发送键值对时使用何种形式, 其中 HTML 表单的内容类型时由表单的 `enctype` 属性指定的:



- enctype="application/x-www-form-urlencoded"
- enctype="multipart/form-data"
- enctype="text/plain"



使用 `Request` 结构的方法获取表单数据的一般步骤是:



1. 调用 `ParseForm` 方法或者 `ParseMultipartForm` 方法对请求进行分析.
2. 根据步骤一调用的方法, 访问相应的 From 字段, PostForm 字段或者 MultipartForm 字段.



```go

func process(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    fmt.Fprintln(w, r.Form)
}
```





...



## ResponseWriter



ResponseWriter 是一个接口, 处理器可以通过这个接口创建 HTTP 响应.



ResponseWriter 在创建应用时会用到 `http.response` 结构, 因为该结构是一个非导出(**nonexported**) 的结构, 所以用户只能通过 ResponseWriter 来使用这个结构, 而不能直接使用它.



> **为什么要以传值的方式将 ResponseWriter 传递给 ServerHTTP**
>
> 接收 Request 结构指针的原因很简单：为了让服务器能够察觉到助力器对 Request 结构的修改， 我们必须以传递引用（pass by reference） 而不是以传递值（pass by value）的方式传递 Request 结构。
>
> 但是，为什么 ServerHTTP 却是以传值的方式接收 ResponseWriter 呢？ 难道服务器不需要知道处处理器对 ResponseWriter 所做的修改吗？
>
> 其实， ResponseWriter 实际上就是 response 这个非导出结构的接口， 而 ResponseWriter 在使用 response 结构是，传递的也是指向 Response 结构的指针， 也就是说，ResponseWRiter 是以传引用而不是传值的方式在使用 response 结构。



ResponseWriter 接口有以下三个方法：

- Write
- WriteHeader
- Header



**对 ResponseWriter 进行写入**



Write 方法接收一个字节数组作为参数，并将数组中的字节写入 HTTP 响应的主题中。如果在适应 Write 方法执行写入操作的时候， 没有为 Header 设置响应的内容类型， 则通过检测被写入的钱 512 字节决定。



```go

func wirteExample(w http.ResponseWriter, r *http.Request) {
    str :=`<html
<head><title>Go Web Programing</title></head>
<body><h1>Hello, It works!</h1></body>
</html>`
    w.Write([]byte(str))
}
```



WriteHeader 方法的名字带有一点误导性质，它并不能用于设置响应的首部（Header 方法才是做这个的）。



WriterHeader 方法接收一个代表 	HTTP  相应状态吗的整数作为参数，并将这个整数用作 HTTP 响应的返回状态码；在调用这个方法之后，用户可以继续对 ResonseWriter 进行写入， 但是不能对响应的首部做任何写入操作。



如果调用 Write 方法之前没有执行过 WriteHeader 方法，默认会使用 200 OK 作为响应的状态码。



```go
import (
    "fmt"
    "encoding/json"
    "net/http"
)

func writeHeaderExample(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(501)
    fmt.Fprintln(w, "No suh service, try next door.")
}
func headerExample(w http.ResonseWriter, r *http.Request) {
    w.Header().Set("Location", "http://google.com")
    w.WriteHeader(302)
}

func jsonExample(w http.ResponseWriter, r *http.Request) {
    w.Header().
    Set("Content-Type", "application/json")
    post := &Post {
        User : "Cheng chao",
        Threads: []string{"first", "second", "third"},
    }
    json, _ := json.Marshal(post)
    w.Write(json)
}
```



## Cookie



cookie 是一种储存在客户端的、体积较小的信息。

cookie 的设计本意是要克服 HTTP 的无状态行， 



gookie 在 Go 语言里面用 `Cookie` 结构表示, 这个结构的定义如下:



```go
type Cookie struct {
    Name string
    Value string
    Path string
    Domain string
    Expires time.Time
    RawExpires string
    MaxAge int
    Secure bool
    HttpOnly bool
    Raw stringh
    Unparsed []string
}
```



没有设置 Expires 字段的 cookie 通常称为会话 cookie 或者临时 cookie ，这种 cookie 在浏览器关闭的时候就会自动被移除。



相对而言，设置了 expires 字段的 cookie 通常称为持久 cookie， 这种 cookie 会一直存在， 知道指定的过期时间来临或者被手动删除为止。



Expires 字段和 MaxAge 字段都可以用于设置 cookie 的过期时间，其中 Expires 字段用于明确地指定 cookie 应该在什么时候过期， 而 MaxAge 字段则指明了 cookie 在被浏览器创建出来斥候能够存活多少秒。



虽然 HTTP1.1 中废弃了 Expires ， 推荐使用 MaxAge 来代替 Expires， 但是几乎手游浏览器都仍然支持 Expires。



为了 让 cookie 在所有的浏览器上都能正常地运作， 一个实际的方法是同时使用 Expires 和 MaxAge。





###将 cookie 发送至浏览器



Cookie 结构的 String 方法可以返回一个经过序列化处理的 cookie， 其中 `Set-Cookie`相应首部的值就是这些序列化之后的 cookie 组成的.



```go
func setCookie(w http.ResponseWrite, r *http.Request) {
    c1 := http.Cookie {
        Name: "first_cookie",
        Value: "Go-web-program",
        HttpOnly: true,
    }
    c2 := http.Cookie {
        Name: "second_cookie",
        Value: "good good study, day day up",
        HttpOnly: true,
    }
    // 第一种设置方法:
    w.Header().Set("Set-Cookie", c1.String())
    w.Header().Set("Set-Cookie", c2.String())
    
    // 第二种设置方法:
    http.SetCookie(w, &c1)
    http.SetCookie(w, &c2)
    
}

func main() {
    server := http.Server{
        Addr: "localhost:8181",
    }
    
    http.HandleFunc("/set_cookie", setCookie)
    
    server.ListenAndServe()
    
    
    
}
```



### 从浏览器获取 cookie



```go
func getCookie(w http.ResonseWrite, r *http.Request) {
    // 返回指定名称的 cookie ,
    // 如果不存在, 则返回一个错误
    c1, err := r.Cookie("first_cookie")
    if err != nil {
        fmt.Fprintln(w, "Cannot get the first cookie")
    }
    // 返回一个包含了所有 cookie 的切片.
    cs := r.Cookies()
    fmt.Fprintln(w, c1)
    fmt.FPrintln(w, cs)
}
```









<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>

