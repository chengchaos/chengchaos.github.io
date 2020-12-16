---
title: Go 语言 http 客户端
key: 2020-12-08
tags: go http
---



Go语言标准库内建提供了 net/http 包，涵盖了 HTTP 客户端和服务端的具体实现。使用 net/http 包，我们可以很方便地编写 HTTP 客户端或服务端的程序。

<!--more-->

## 基本方法

net/http 包提供了最简洁的 HTTP 客户端实现，无需借助第三方网络通信库（比如 libcurl）就可以直接使用最常见的 GET 和 POST 方式发起 HTTP 请求。

 具体来说，我们可以通过 net/http 包里面的 Client 类提供的如下方法发起 HTTP 请求：

```go
func (c *Client) Get(url string) (r *Response, err error)
func (c *Client) Post(url string, bodyType string body io.Reader) (r *Response, err error)
func (c *Client) PostForm(url string, data url.Values) (r *Response, err error)
func (c *Client) Head(url string) (r *Response, err error)
func (c *Client) Do(req *Request) (resp *Response, err error)
```



### http.Get()



使用 get 方法请求一个资源

> 等价于 `http.DefaultClient.Get()`

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	resp, err := http.Get("https://pts-tsp.accenture.cn")
	if err != nil {
		fmt.Println(err)
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}

```

通过 `http.Get` 发起请求时，默认调用的是上述 `http.Client` 缺省对象上的 `Get` 方法：

```go
func Get(url string) (resp *Response, err error) {
    return DefaultClient.Get(url)
}
```

而 `DefaultClient` 默认指向的正是 `http.Client` 的实例对象：

```go
var DefaultClient = &Client{}
```

它是 net/http 包公开属性，当我们在 http 上调用 Get、Post、PostForm、Head 方法时，最终调用的都是该对象上的对应方法。



### http.Post()

要以 POST 的方式发送数据，也很简单，只需调用 http.Post() 方法并依次传递下面的 3 个参数即可：

- 请求的目标 URL
- 将要 POST 数据的资源类型（MIMEType）
- 数据的比特流（[]byte 形式）

下面演示如何上传一张图片

```go
resp, err := http.Post("http://c.biancheng.net/upload", "image/jpeg", &buf)
if err != nil {
    fmt.Println(err)
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
if err != nil {
    fmt.Println(err)
}
fmt.Println(string(body))
```



### http.PostForm()

http.PostForm() 方法实现了标准编码格式为“application/x-www-form-urlencoded”的表单提交，下面的示例代码模拟了 HTML 表单向后台提交信息的过程：

> 注意：POST 请求参数需要通过 url.Values 方法进行编码和封装。



```go

func doPostForm() {
	targetUrl := "http://localhost:8080/message-board"
	// contentType := "application/x-www-form-urlencoded"
	// reqBody := "title=hello&message=wokao"
	//body io.Reader
	resp, err := http.PostForm(targetUrl, url.Values{
		"title":   {"一个标题"},
		"message": {"<p>无力吐槽</p>"},
	})

	if err != nil {
		log.Println("err ", err)
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Println(err)
	}
	log.Println(string(body))
}
```



### http.Header()

HTTP 的 Head 请求表示只请求目标 URL 的响应头信息，不返回响应实体。可以通过 net/http 包的 http.Head() 方法发起 Head 请求，该方法和 http.Get() 方法一样，只需要传入目标 URL 参数即可。



```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    resp, err := http.Head("http://c.biancheng.net")
    if err != nil {
        fmt.Println("Request Failed: ", err.Error())
        return
    }
    defer resp.Body.Close() // 打印头信息
    for key, value := range resp.Header {
        fmt.Println(key, ":", value)
    }
}
```



### (*http.Client).Do()

下面我们再来看一下 http.Client 类的 Do 方法。

 在多数情况下，`http.Get()`、`http.Post() `和 `http.PostForm()` 就可以满足需求，但是如果我们发起的 HTTP 请求需要更多的自定义请求信息，比如：

- 设置自定义 User-Agent，而不是默认的 Go http package；
- 传递 Cookie 信息；
- 发起其它方式的 HTTP 请求，比如 PUT、PATCH、DELETE 等。


此时可以通过 `http.Client` 类提供的 `Do()` 方法来实现，使用该方法时，就不再是通过缺省的 DefaultClient 对象调用  http.Client 类中的方法了，而是需要我们手动实例化 Client 对象并传入添加了自定义请求头信息的请求对象来发起 HTTP 请求



```go

func doDo() {
	targetUrl := "http://localhost:8080/index"
	// 初始化客户端请求对象
	// 第一个是请求方法，第二个是目标 URL，第三个是请求实体
	req, err := http.NewRequest("GET", targetUrl, nil)
	if err != nil {
		log.Println(err)
		return
	}
	// 添加自定义请求头
	req.Header.Add("Customer-Header", "Some-value")
	// 其他请求头配置
	client := &http.Client{
		// 设置客户端属性
	}

	resp, err := client.Do(req)
	if err != nil {
		log.Println(err)
		return
	}
	defer resp.Body.Close()
	io.Copy(os.Stdout, resp.Body)
}
```



## 高级封装



除了之前介绍的基本 HTTP 操作，Go语言标准库也暴露了比较底层的 HTTP 相关库，让开发者可以基于这些库灵活定制 HTTP 服务器和使用 HTTP 服务。



### 1) 自定义 http.Client

前面我们使用的 http.Get()、http.Post()、http.PostForm() 和 http.Head() 方法其实都是在  http.DefaultClient 的基础上进行调用的，比如 http.Get() 等价于  http.Default-Client.Get()，依次类推。

 http.DefaultClient 在字面上就向我们传达了一个信息，既然存在默认的 Client，那么 HTTP Client  大概是可以自定义的。实际上确实如此，在 net/http 包中，的确提供了 Client 类型。让我们来看一看 http.Client  类型的结构：

```go
type Client struct {
    // Transport 用于确定HTTP请求的创建机制。
    // 如果为空，将会使用DefaultTransport
    Transport RoundTripper
    // CheckRedirect定义重定向策略。
    // 如果CheckRedirect不为空，客户端将在跟踪HTTP重定向前调用该函数。
    // 两个参数req和via分别为即将发起的请求和已经发起的所有请求，最早的
    // 已发起请求在最前面。
    // 如果CheckRedirect返回错误，客户端将直接返回错误，不会再发起该请求。
    // 如果CheckRedirect为空，Client将采用一种确认策略，将在10个连续
    // 请求后终止
    CheckRedirect func(req *Request, via []*Request) error
    // 如果Jar为空，Cookie将不会在请求中发送，并会
    // 在响应中被忽略
    Jar CookieJar
}
```

在Go语言标准库中，http.Client 类型包含了 3 个公开数据成员：

```go
Transport RoundTripper
CheckRedirect func(req *Request, via []*Request) error
Jar CookieJar
```



其中 Transport 类型必须实现 http.RoundTripper 接口。Transport 指定了执行一个 HTTP  请求的运行机制，倘若不指定具体的 Transport，默认会使用 http.DefaultTransport，这意味着  http.Transport 也是可以自定义的。net/http 包中的 http.Transport 类型实现了  http.RoundTripper 接口。

 CheckRedirect 函数指定处理重定向的策略。当使用 HTTP Client 的 Get() 或者是 Head() 方法发送 HTTP  请求时，若响应返回的状态码为 30x （比如 301 / 302 / 303 / 307），HTTP Client  会在遵循跳转规则之前先调用这个 CheckRedirect 函数。

 Jar 可用于在 HTTP Client 中设定 Cookie，Jar 的类型必须实现了 http.CookieJar 接口，该接口预定义了 SetCookies() 和 Cookies() 两个方法。

 如果 HTTP Client 中没有设定 Jar，Cookie 将被忽略而不会发送到客户端。实际上，我们一般都用 http.SetCookie() 方法来设定 Cookie。

 使用自定义的 http.Client 及其 Do() 方法，我们可以非常灵活地控制 HTTP 请求，比如发送自定义 HTTP Header 或是改写重定向策略等。创建自定义的 HTTP Client 非常简单，具体代码如下：

```go
client := &http.Client {
    CheckRedirect: redirectPolicyFunc,
}
resp, err := client.Get("http://example.com")
// ...
req, err := http.NewRequest("GET", "http://example.com", nil)
// ...
req.Header.Add("User-Agent", "Our Custom User-Agent")
req.Header.Add("If-None-Match", `W/"TheFileEtag"`)
resp, err := client.Do(req)
// ...
```



### 2) 自定义 http.Transport

在 http.Client 类型的结构定义中，我们看到的第一个数据成员就是一个 http.Transport 对象，该对象指定执行一个 HTTP 请求时的运行规则。下面我们来看看 http.Transport 类型的具体结构：

```go
type Transport struct {
    // Proxy指定用于针对特定请求返回代理的函数。
    // 如果该函数返回一个非空的错误，请求将终止并返回该错误。
    // 如果Proxy为空或者返回一个空的URL指针，将不使用代理
    Proxy func(*Request) (*url.URL, error)
    // Dial指定用于创建TCP连接的dail()函数。
    // 如果Dial为空，将默认使用net.Dial()函数
    Dial func(net, addr string) (c net.Conn, err error)
    // TLSClientConfig指定用于tls.Client的TLS配置。
    // 如果为空则使用默认配置
    TLSClientConfig *tls.Config
    DisableKeepAlives bool
    DisableCompression bool
    // 如果MaxIdleConnsPerHost为非零值，它用于控制每个host所需要
    // 保持的最大空闲连接数。如果该值为空，则使用DefaultMaxIdleConnsPerHost
    MaxIdleConnsPerHost int
    // ...
}
```



在上面的代码中，我们定义了 http.Transport 类型中的公开数据成员，下面详细说明其中的各行代码。



```go
Proxy func(*Request) (*url.URL, error)
```

Proxy 指定了一个代理方法，该方法接受一个 *Request 类型的请求实例作为参数并返回一个最终的 HTTP 代理。如果 Proxy 未指定或者返回的 *URL 为零值，将不会有代理被启用。



```go
Dial func(net, addr string) (c net.Conn, err error)
```

Dial 指定具体的 dial() 方法来创建 TCP 连接。如果不指定，默认将使用 net.Dial() 方法。



```go
TLSClientConfig *tls.Config
```

SSL 连接专用，TLSClientConfig 指定 tls.Client 所用的 TLS 配置信息，如果不指定，也会使用默认的配置。



```go
DisableKeepAlives bool
```

是否取消长连接，默认值为 false，即启用长连接。



```go
DisableCompression bool
```

是否取消压缩（GZip），默认值为 false，即启用压缩。



```go
MaxIdleConnsPerHost int
```

指定与每个请求的目标主机之间的最大非活跃连接（keep-alive）数量。如果不指定，默认使用 DefaultMaxIdleConnsPerHost 的常量值。



 除了 http.Transport 类型中定义的公开数据成员以外，它同时还提供了几个公开的成员方法。

- `func(t *Transport) CloseIdleConnections()`。该方法用于关闭所有非活跃的连接。
- `func(t *Transport) RegisterProtocol(scheme string, rt  RoundTripper)`。该方法可用于注册并启用一个新的传输协议，比如 WebSocket 的传输协议标准（ws），或者 FTP、File  协议等。
- `func(t *Transport) RoundTrip(req *Request) (resp *Response, err error)`。用于实现 http.RoundTripper 接口。


 自定义 http.Transport 也很简单，如下列代码所示：

```go
tr := &http.Transport{
    TLSClientConfig: &tls.Config{RootCAs: pool},
    DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```

Client 和 Transport 在执行多个 goroutine 的并发过程中都是安全的，但出于性能考虑，应当创建一次后反复使用。



### 3) 灵活的 http.RoundTripper 接口

在前面的两小节中，我们知道 HTTP Client 是可以自定义的，而 http.Client 定义的第一个公开成员就是一个 http.Transport 类型的实例，且该成员所对应的类型必须实现 http.RoundTripper 接口。

 下面我们来看看 http.RoundTripper 接口的具体定义：

```go
type RoundTripper interface {
    // RoundTrip执行一个单一的HTTP事务，返回相应的响应信息。
    // RoundTrip函数的实现不应试图去理解响应的内容。如果RoundTrip得到一个响应，
    // 无论该响应的HTTP状态码如何，都应将返回的err设置为nil。非空的err
    // 只意味着没有成功获取到响应。
    // 类似地，RoundTrip也不应试图处理更高级别的协议，比如重定向、认证和
    // Cookie等。
    //
    // RoundTrip不应修改请求内容, 除非了是为了理解Body内容。每一个请求
    // 的URL和Header域都应被正确初始化
    RoundTrip(*Request) (*Response, error)
}
```

从上述代码中可以看到，http.RoundTripper 接口很简单，只定义了一个名为 RoundTrip 的方法。任何实现了  RoundTrip() 方法的类型即可实现 http.RoundTripper 接口。前面我们看到的 http.Transport  类型正是实现了 RoundTrip() 方法继而实现了该接口。

 http.RoundTripper 接口定义的 RoundTrip() 方法用于执行一个独立的 HTTP 事务，接受传入的 \*Request 请求值作为参数并返回对应的 \*Response 响应值，以及一个 error 值。

 在实现具体的 RoundTrip() 方法时，不应该试图在该函数里边解析 HTTP 响应信息。若响应成功，error 的值必须为  nil，而与返回的 HTTP 状态码无关。若不能成功得到服务端的响应，error 必须为非零值。类似地，也不应该试图在 RoundTrip()  中处理协议层面的相关细节，比如重定向、认证或是 cookie 等。

 非必要情况下，不应该在 RoundTrip() 中改写传入的请求体（\*Request），请求体的内容（比如 URL 和 Header 等）必须在传入 RoundTrip() 之前就已组织好并完成初始化。

 通常，我们可以在默认的 http.Transport 之上包一层 Transport 并实现 RoundTrip() 方法，代码如下所示。

```go
package main
import(
    "net/http"
)
type OurCustomTransport struct {
    Transport http.RoundTripper
}
func (t *OurCustomTransport) transport() http.RoundTripper {
    if t.Transport != nil {
        return t.Transport
    }
    return http.DefaultTransport
}
func (t *OurCustomTransport) RoundTrip(req *http.Request) (*http.Response, error) {
    // 处理一些事情 ...
    // 发起HTTP请求
    // 添加一些域到req.Header中
    return t.transport().RoundTrip(req)
}
func (t *OurCustomTransport) Client() *http.Client {
    return &http.Client{Transport: t}
}
func main() {
    t := &OurCustomTransport{
        //...
    }
    c := t.Client()
    resp, err := c.Get("http://example.com")
    // ...
}
```

因为实现了 http.RoundTripper 接口的代码通常需要在多个 goroutine 中并发执行，因此我们必须确保实现代码的线程安全性。



### 4) 设计优雅的 HTTP Client

综上示例讲解可以看到，Go语言标准库提供的 HTTP Client 是相当优雅的。一方面提供了极其简单的使用方式，另一方面又具备极大的灵活性。

 Go语言标准库提供的 HTTP Client 被设计成上下两层结构。一层是上述提到的 http.Client  类及其封装的基础方法，我们不妨将其称为“业务层”。之所以称为业务层，是因为调用方通常只需要关心请求的业务逻辑本身，而无需关心非业务相关的技术细节，这些细节包括：

- HTTP 底层传输细节
- HTTP 代理
- gzip 压缩
- 连接池及其管理
- 认证（SSL 或其他认证方式）


 之所以 HTTP Client 可以做到这么好的封装性，是因为 HTTP Client 在底层抽象了 http.RoundTripper 接口，而 http.Transport 实现了该接口，从而能够处理更多的细节，我们不妨将其称为“传输层”。

 HTTP Client 在业务层初始化 HTTP Method、目标 URL、请求参数、请求内容等重要信息后，经过“传输层”，“传输层”在业务层处理的基础上补充其他细节，然后再发起 HTTP 请求，接收服务端返回的 HTTP 响应。



转自：

- [Go语言HTTP客户端实现简述](http://c.biancheng.net/view/4520.html)

EOF

---

Power by TeXt.
