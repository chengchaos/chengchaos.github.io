---
title: Golang context 包
key: 2020-06-09
tags: go context 
---




在 Go http 包的 Server 中，每一个请求在都有一个对应的`goroutine`去处理。请求处理函数通常会启动额外的`goroutine`用来访问后端服务，比如数据库和 RPC 服务。用来处理一个请求的`goroutine`通常需要访问一些与请求特定的数据，比如终端用户的身份认证信息、验证相关的 token、请求的截止时间。当一个请求被取消或超时时，所有用来处理该请求的`goroutine`都应该迅速退出，然后系统才能释放这些`goroutine`占用的资源，[官方博客](https://blog.golang.org/context)。

> 注意：`go1.6`及之前版本请使用`golang.org/x/net/context`。`go1.7`及之后已移到标准库`context`。



<!--more-->



每个 Goroutine 在执行之前，都要先知道程序当前的执行状态，通常将这些执行状态封装在一个`Context`变量中，传递给要执行的 Goroutine 中。上下文则几乎已经成为传递与请求同生存周期变量的标准方法。在网络编程下，当接收到一个网络请求 Request，处理 Request 时，我们可能需要开启不同的 Goroutine 来获取数据与逻辑处理，即一个请求 Request，会在多个 Goroutine 中处理。而这些 Goroutine 可能需要共享 Request 的一些信息；同时当 Request 被取消或者超时的时候，所有从这个 Request 创建的所有 Goroutine 也应该被结束。



Go 的设计者早考虑多个 Goroutine 共享数据，以及多 Goroutine 管理机制。`Context`介绍请参考 [Go Concurrency Patterns: Context](http://blog.golang.org/context)，[golang.org/x/net/context](http://godoc.org/golang.org/x/net/context) 包就是这种机制的实现。

`context` 包不仅实现了在程序单元之间共享状态变量的方法，同时能通过简单的方法，使我们在被调用程序单元的外部，通过设置 ctx 变量值，将过期或撤销这些信号传递给被调用的程序单元。在网络编程中，若存在 A 调用 B的 API, B 再调用 C 的 API，若 A 调用 B 取消，那也要取消 B 调用 C，通过在 A,B,C 的 API 调用之间传递`Context`，以及判断其状态，就能解决此问题，这是为什么gRPC的接口中带上`ctx context.Context`参数的原因之一。



### Context 原理

Context 的调用应该是链式的，通过`WithCancel`，`WithDeadline`，`WithTimeout`或`WithValue`派生出新的 Context。当父 Context 被取消时，其派生的所有 Context 都将取消。

通过`context.WithXXX`都将返回新的 Context 和 CancelFunc。调用 CancelFunc 将取消子代，移除父代对子代的引用，并且停止所有定时器。未能调用 CancelFunc 将泄漏子代，直到父代被取消或定时器触发。`go vet`工具检查所有流程控制路径上使用 CancelFuncs。

### 遵循规则

遵循以下规则，以保持包之间的接口一致，并启用静态分析工具以检查上下文传播。

Programs that use Contexts should follow these rules to keep interfaces  consistent across packages and enable static analysis tools to check  context propagation:

1. Do not store Contexts inside a struct type; instead, pass a Context  explicitly to each function that needs it. The Context should be the  first parameter, typically named ctx；不要将 Contexts 放入结构体，相反`context`应该作为第一个参数传入，命名为`ctx`。 ` func DoSomething（ctx context.Context，arg Arg）error {   // ... use ctx ...  } `
2. Do not pass a nil Context, even if a function permits it. Pass  context.TODO if you are unsure about which Context to  use；即使方法允许，也不要传入一个 nil 的 Context。如果不知道用哪种 Context，可以使用`context.TODO()`。
3. Use context Values only for request-scoped data that transits processes  and APIs, not for passing optional parameters to functions；使用 context 的 Value 相关方法只应该用于在程序和接口中传递的和请求相关的元数据，不要用它来传递一些可选的参数
4. The same Context may be passed to functions running in different  goroutines; Contexts are safe for simultaneous use by multiple  goroutines；相同的 Context 可以传递给在不同的`goroutine`；Context 是并发安全的。

在子Context被传递到的goroutine中，应该对该子Context的Done信道（channel）进行监控，一旦该信道被关闭（即上层运行环境撤销了本goroutine的执行），应主动终止对当前请求信息的处理，释放资源并返回。



### Context 包

Context 结构体。

```
// A Context carries a deadline, cancelation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}
```

- Done()，返回一个channel。当times out或者调用cancel方法时，将会close掉。
- Err()，返回一个错误。该context为什么被取消掉。
- Deadline()，返回截止时间和ok。
- Value()，返回值。

所有方法

```go
func Background() Context
func TODO() Context

func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val interface{}) Context
```

上面可以看到Context是一个接口，想要使用就得实现其方法。在context包内部已经为我们实现好了两个空的Context，可以通过调用Background()和TODO()方法获取。一般的将它们作为Context的根，往下派生。

### WithCancel 例子

WithCancel 以一个新的 Done channel 返回一个父 Context 的拷贝。

```go
   229  func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
   230      c := newCancelCtx(parent)
   231      propagateCancel(parent, &c)
   232      return &c, func() { c.cancel(true, Canceled) }
   233  }
   234  
   235  // newCancelCtx returns an initialized cancelCtx.
   236  func newCancelCtx(parent Context) cancelCtx {
   237      return cancelCtx{
   238          Context: parent,
   239          done:    make(chan struct{}),
   240      }
   241  }
```

此示例演示使用一个可取消的上下文，以防止 goroutine 泄漏。示例函数结束时，defer 调用 cancel 方法，gen goroutine 将返回而不泄漏。

```go
package main

import (
    "context"
    "fmt"
)

func main() {
    // gen generates integers in a separate goroutine and
    // sends them to the returned channel.
    // The callers of gen need to cancel the context once
    // they are done consuming generated integers not to leak
    // the internal goroutine started by gen.
    gen := func(ctx context.Context) <-chan int {
        dst := make(chan int)
        n := 1
        go func() {
            for {
                select {
                case <-ctx.Done():
                    return // returning not to leak the goroutine
                case dst <- n:
                    n++
                }
            }
        }()
        return dst
    }

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel() // cancel when we are finished consuming integers

    for n := range gen(ctx) {
        fmt.Println(n)
        if n == 5 {
            break
        }
    }
}
```

### WithDeadline 例子

```go
   369  func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc) {
   370      if cur, ok := parent.Deadline(); ok && cur.Before(deadline) {
   371          // The current deadline is already sooner than the new one.
   372          return WithCancel(parent)
   373      }
   374      c := &timerCtx{
   375          cancelCtx: newCancelCtx(parent),
   376          deadline:  deadline,
   377      }
   ......
```

可以清晰的看到，当派生出的子 Context 的deadline在父Context之后，直接返回了一个父Context的拷贝。故语义上等效为父。

WithDeadline 的最后期限调整为不晚于 d 返回父上下文的副本。如果父母的截止日期已经早于 d，WithDeadline  （父，d） 是在语义上等效为父。返回的上下文完成的通道关闭的最后期限期满后，返回的取消函数调用时，或当父上下文完成的通道关闭，以先发生者为准。

看看官方例子：

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    d := time.Now().Add(50 * time.Millisecond)
    ctx, cancel := context.WithDeadline(context.Background(), d)

    // Even though ctx will be expired, it is good practice to call its
    // cancelation function in any case. Failure to do so may keep the
    // context and its parent alive longer than necessary.
    defer cancel()

    select {
    case <-time.After(1 * time.Second):
        fmt.Println("overslept")
    case <-ctx.Done():
        fmt.Println(ctx.Err())
    }
}
```

### WithTimeout 例子

WithTimeout 返回 WithDeadline(parent, time.Now().Add(timeout))。

```
   436  func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
   437      return WithDeadline(parent, time.Now().Add(timeout))
   438  }
```

看看官方例子：

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // Pass a context with a timeout to tell a blocking function that it
    // should abandon its work after the timeout elapses.
    ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
    defer cancel()

    select {
    case <-time.After(1 * time.Second):
        fmt.Println("overslept")
    case <-ctx.Done():
        fmt.Println(ctx.Err()) // prints "context deadline exceeded"
    }
}
```

### WithValue 例子

```
   454  func WithValue(parent Context, key, val interface{}) Context {
   454      if key == nil {
   455          panic("nil key")
   456      }
   457      if !reflect.TypeOf(key).Comparable() {
   458          panic("key is not comparable")
   459      }
   460      return &valueCtx{parent, key, val}
   461  }
```

WithValue 返回的父与键关联的值在 val 的副本。

使用上下文值仅为过渡进程和 Api 的请求范围的数据，而不是将可选参数传递给函数。

提供的键必须是可比性和应该不是字符串类型或任何其他内置的类型以避免包使用的上下文之间的碰撞。WithValue  用户应该定义自己的键的类型。为了避免分配分配给接口 {} 时，上下文键经常有具体类型结构  {}。另外，导出的上下文关键变量静态类型应该是一个指针或接口。

看看官方例子：

```go
package main

import (
    "context"
    "fmt"
)

func main() {
    type favContextKey string

    f := func(ctx context.Context, k favContextKey) {
        if v := ctx.Value(k); v != nil {
            fmt.Println("found value:", v)
            return
        }
        fmt.Println("key not found:", k)
    }

    k := favContextKey("language")
    ctx := context.WithValue(context.Background(), k, "Go")

    f(ctx, k)
    f(ctx, favContextKey("color"))
}
```

### 参考连接

[1]  [快速掌握 Golang context 包，简单示例]( https://deepzz.com/post/golang-context-package-notes.html)
[2]  [理解Go Context机制](https://www.cnblogs.com/zhangboyu/p/7456606.html)
[3]  [Go语言并发模型：使用 context](https://segmentfault.com/a/1190000006744213)





EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





