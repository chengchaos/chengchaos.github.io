---
title: Go 中 io 包的使用方法
key: 2025-03-25
tags: go io.Reader io.Writer
---

原文: [Go 中 io 包的使用方法](https://medium.com/learning-the-go-programming-language/streaming-io-in-go-d93507931185)

在 Go 中，输入和输出操作是使用原语实现的，这些原语将数据模拟成可读的或可写的字节流。
为此，Go 的 io 包提供了 io.Reader 和 io.Writer 接口，分别用于数据的输入和输出，如图：

[!io](https://segmentfault.com/img/bVbdzja?w=1600&h=214)

Go 官方提供了一些 API，支持对内存结构，文件，网络连接等资源进行操作
本文重点介绍如何实现标准库中 io.Reader 和 io.Writer 两个接口，来完成流式传输数据。

<!--more-->

## 0x01 io.Reader

A reader, represented by interfaceio.Reader, reads data from some source into a transfer buffer where it can be streamed and consumed, as illustrated below:

For a type to function as a reader, it must implement method `Read(p []byte)` from interface io.Reader (shown below):

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

Implementation of the `Read()` method should return the number of bytes read or an error if one occurred. If the source has exhausted its content, Read should return `io.EOF`.

## Reading Rules (added)

After a Reddit feedback, I have decided to add this section about reading that may be helpful. The behavior of a reader will depend on its implementation, however there are a few rules, from the `io.Reader` doc that you should be aware of when consuming directly from a reader:

1. `Read()` will read up to `len(p)` into p, when possible.
1. After a `Read()` call, `n` may be less then `len(p)`.
1. Upon error, `Read()` may still return `n` bytes in buffer `p`. For instance, reading from a TCP socket that is abruptly closed. Depending on your use, you may choose to keep the bytes in `p` or retry.
4. When a `Read()` exhausts available data, a reader may return a non-zero `n` and `err=io.EOF`. However, depending on implementation, a reader may choose to return a non-zero `n` and `err = nil` at the end of stream. In that case, any subsequent reads must return `n=0`, `err=io.EOF`.
5. Lastly, a call to `Read()` that returns `n=0` and `err=nil` does not mean `EOF` as the next call to `Read()` may return more data.

As you can see, properly reading a stream directly from a reader can be tricky. Fortunately, readers from the standard library follow sensible approaches that make it easy to stream. Nevertheless, before using a reader, consult its documentation.

### Streaming data from readers

Streaming data directly from a reader is easy. Method `Read` is designed to be called within a loop where, with each iteration, it reads a chunk of data from the source and places it into buffer `p`. This loop will continue until the method returns an `io.EOF` error.

The following is a simple example that uses a string reader, created with `strings.NewReader(string)`, to stream byte values from a string source:

```go
func main() {
    reader := strings.NewReader("Clear is better than clever")
    p := make([]byte, 4)
    for {
        n, err := reader.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Println(string(p[:n]))
    }
}
```

### 自己实现一个 Reader

上一节是使用标准库中的 io.Reader 读取器实现的。
现在，让我们看看如何自己实现一个。它的功能是从流中过滤掉非字母字符。


```go
type alphaReader struct {
    // 资源
    src string
    // 当前读取到的位置 
    cur int
}

// 创建一个实例
func newAlphaReader(src string) *alphaReader {
    return &alphaReader{src: src}
}

// 过滤函数
func alpha(r byte) byte {
    if (r >= 'A' && r <= 'Z') || (r >= 'a' && r <= 'z') {
        return r
    }
    return 0
}

// Read 方法
func (a *alphaReader) Read(p []byte) (int, error) {
    // 当前位置 >= 字符串长度 说明已经读取到结尾 返回 EOF
    if a.cur >= len(a.src) {
        return 0, io.EOF
    }

    // x 是剩余未读取的长度
    x := len(a.src) - a.cur
    n, bound := 0, 0
    if x >= len(p) {
        // 剩余长度超过缓冲区大小，说明本次可完全填满缓冲区
        bound = len(p)
    } else if x < len(p) {
        // 剩余长度小于缓冲区大小，使用剩余长度输出，缓冲区不补满
        bound = x
    }

    buf := make([]byte, bound)
    for n < bound {
        // 每次读取一个字节，执行过滤函数
        if char := alpha(a.src[a.cur]); char != 0 {
            buf[n] = char
        }
        n++
        a.cur++
    }
    // 将处理后得到的 buf 内容复制到 p 中
    copy(p, buf)
    return n, nil
}

func main() {
    reader := newAlphaReader("Hello! It's 9am, where is the sun?")
    p := make([]byte, 4)
    for {
        n, err := reader.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
    fmt.Println()
}
```

### Chaining Readers

The standard library has many readers already implemented. It is a common idiom to use a reader as the source of another reader. This chaining of readers allows one reader to reuse logic from another as is done in the following source snippet which updates the alphaReader to accept an io.Reader as its source. This reduces the complexity of the code by pushing stream housekeeping concerns to the root reader.

```go
type alphaReader struct {
    // alphaReader 里组合了标准库的 io.Reader
    reader io.Reader
}

func newAlphaReader(reader io.Reader) *alphaReader {
    return &alphaReader{reader: reader}
}

func alpha(r byte) byte {
    if (r >= 'A' && r <= 'Z') || (r >= 'a' && r <= 'z') {
        return r
    }
    return 0
}

func (a *alphaReader) Read(p []byte) (int, error) {
    // 这行代码调用的就是 io.Reader
    n, err := a.reader.Read(p)
    if err != nil {
        return n, err
    }
    buf := make([]byte, n)
    for i := 0; i < n; i++ {
        if char := alpha(p[i]); char != 0 {
            buf[i] = char
        }
    }

    copy(p, buf)
    return n, nil
}

func main() {
    //  使用实现了标准库 io.Reader 接口的 strings.Reader 作为实现
    reader := newAlphaReader(strings.NewReader("Hello! It's 9am, where is the sun?"))
    p := make([]byte, 4)
    for {
        n, err := reader.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
    fmt.Println()
}
```

这样做的另一个优点是 alphaReader 能够从任何 Reader 实现中读取。
例如，以下代码展示了 alphaReader 如何与 os.File 结合以过滤掉文件中的非字母字符：

```go
func main() {
    // file 也实现了 io.Reader
    file, err := os.Open("./alpha_reader3.go")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()
    
    // 任何实现了 io.Reader 的类型都可以传入 newAlphaReader
    // 至于具体如何读取文件，那是标准库已经实现了的，我们不用再做一遍，达到了重用的目的
    reader := newAlphaReader(file)
    p := make([]byte, 4)
    for {
        n, err := reader.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
    fmt.Println()
}
```

## The io.Writer

A writer, represented by interface `io.Writer`, streams data from a buffer and writes it to a target resource as illustrated below:


All stream writers must implement method `Write(p []byte)` from interface `io.Writer`(shown below). The method is designed to read data from buffer p and write it to a specified target resource.

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

Implementation of the `Write()` method should return the number of bytes written or an error if any occurred

### Using writers

The standard library comes with many pre-implemented `io.Writer` types. Working with writers directly is simple as shown in the following code snippet which uses type `bytes.Buffer` as an `io.Writer` to write data into a memory buffer.

```go
func main() {
    proverbs := []string{
        "Channels orchestrate mutexes serialize",
        "Cgo is not Go",
        "Errors are values",
        "Don't panic",
    }
    var writer bytes.Buffer

    for _, p := range proverbs {
        n, err := writer.Write([]byte(p))
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        if n != len(p) {
            fmt.Println("failed to write data")
            os.Exit(1)
        }
    }

    fmt.Println(writer.String())
}
```

### Implementing a custom io.Writer
The code in this section shows how to implement a custom `io.Writer` called `chanWriter` which writes its content to a Go channel as a sequence of bytes.

```go
type chanWriter struct {
    // ch 实际上就是目标资源
    ch chan byte
}

func newChanWriter() *chanWriter {
    return &chanWriter{make(chan byte, 1024)}
}

func (w *chanWriter) Chan() <-chan byte {
    return w.ch
}

func (w *chanWriter) Write(p []byte) (int, error) {
    n := 0
    // 遍历输入数据，按字节写入目标资源
    for _, b := range p {
        w.ch <- b
        n++
    }
    return n, nil
}

func (w *chanWriter) Close() error {
    close(w.ch)
    return nil
}

func main() {
    writer := newChanWriter()
    go func() {
        defer writer.Close()
        writer.Write([]byte("Stream "))
        writer.Write([]byte("me!"))
    }()
    for c := range writer.Chan() {
        fmt.Printf("%c", c)
    }
    fmt.Println()
}
```

要使用这个 Writer，只需在函数 `main()` 中调用 `writer.Write()`（在单独的goroutine中）。
因为 `chanWriter` 还实现了接口 `io.Closer` ，所以调用方法 w`riter.Close()` 来正确地关闭channel，以避免发生泄漏和死锁。

## Useful types and packages for IO

As mentioned 如前所述, the Go standard library comes with many useful functions and other types that make it easy to work with streaming IO.

### os.File

Type `os.File` represents a file on the local system. It implements both `io.Reader` and `io.Writer` and, therefore 因此, can be used in any streaming IO contexts. For instance, the following example shows how to write successive string slices directly to a file:

```go
func main() {
    proverbs := []string{
        "Channels orchestrate mutexes serialize\n",
        "Cgo is not Go\n",
        "Errors are values\n",
        "Don't panic\n",
    }
    file, err := os.Create("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    for _, p := range proverbs {
        // file 类型实现了 io.Writer
        n, err := file.Write([]byte(p))
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        if n != len(p) {
            fmt.Println("failed to write data")
            os.Exit(1)
        }
    }
    fmt.Println("file write done")
}
```

Conversely 相反地, type `io.File` can be used as a reader to stream the content of a file from the local file system. For instance, the following source snippet reads a file and prints its content:

```go
func main() {
    file, err := os.Open("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    p := make([]byte, 4)
    for {
        n, err := file.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
}
```

## Standard output, input, and error

The os package exposes three variables, `os.Stdout`, `os.Stdin`, and `os.Stderr`, that are of type `*os.File` to represent file handles for the OS’s standard output, input, and error respectively. For instance, the following source snippet prints directly to standard output:

```go
func main() {
    proverbs := []string{
        "Channels orchestrate mutexes serialize\n",
        "Cgo is not Go\n",
        "Errors are values\n",
        "Don't panic\n",
    }

    for _, p := range proverbs {
        // 因为 os.Stdout 也实现了 io.Writer
        n, err := os.Stdout.Write([]byte(p))
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        if n != len(p) {
            fmt.Println("failed to write data")
            os.Exit(1)
        }
    }
}
```

### io.Copy()

Function `io.Copy()` makes it easy to stream data from a source reader to a target writer. It abstracts out the for-loop pattern (we’ve seen so far) and properly (正确地,适当地,恰当地) handle `io.EOF` and byte counts.

The following shows a simplified version of a previous program which copies the content of in-memory reader proberbs and copies it to writer file:

```go
func main() {
    proverbs := new(bytes.Buffer)
    proverbs.WriteString("Channels orchestrate mutexes serialize\n")
    proverbs.WriteString("Cgo is not Go\n")
    proverbs.WriteString("Errors are values\n")
    proverbs.WriteString("Don't panic\n")

    file, err := os.Create("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    // io.Copy 完成了从 proverbs 读取数据并写入 file 的流程
    if _, err := io.Copy(file, proverbs); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Println("file created")
}
```

Similarly (adv. 相似地; 类似地) 那么，我们也可以使用 io.Copy() 函数重写从文件读取并打印到标准输出的先前程序，如下所示：

```go
func main() {
    file, err := os.Open("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    if _, err := io.Copy(os.Stdout, file); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}
```

### io.WriteString()

This function provides the convenience (n. 方便, 便利) of writing a string value into a specified writer:

```go
func main() {
    file, err := os.Create("./magic_msg.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()
    if _, err := io.WriteString(file, "Go is fun!"); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}
```

### Pipe writers and readers

Types `io.PipeWriter` and `io.PipeReader` model IO operations as in memory pipes. Data is written to the pipe’s writer-end and is read on the pipe’s reader-end using separate (adj. 分开的；单独的) go routines. The following creates the pipe reader/writer pair using the `io.Pipe()` which is then used to copy data from a buffer proverbs to `io.Stdout`:

```go
func main() {
    proverbs := new(bytes.Buffer)
    proverbs.WriteString("Channels orchestrate mutexes serialize\n")
    proverbs.WriteString("Cgo is not Go\n")
    proverbs.WriteString("Errors are values\n")
    proverbs.WriteString("Don't panic\n")

    piper, pipew := io.Pipe()

    // 将 proverbs 写入 pipew 这一端
    go func() {
        defer pipew.Close()
        io.Copy(pipew, proverbs)
    }()

    // 从另一端 piper 中读取数据并拷贝到标准输出
    io.Copy(os.Stdout, piper)
    piper.Close()
}
```

### Buffered IO

Go supports buffered IO via package `bufio` which makes it easy to work with textual content. For instance, the following program reads the content of a file line-by-line delimited with value `'\n'` :


```go
func main() {
    file, err := os.Open("./planets.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()
    reader := bufio.NewReader(file)

    for {
        line, err := reader.ReadString('\n')
        if err != nil {
            if err == io.EOF {
                break
            } else {
                fmt.Println(err)
                os.Exit(1)
            }
        }
        fmt.Print(line)
    }

}
```

### Util package

Package `ioutil`, a sub-package of io, offers several convenience functions for IO. For instance, the following uses function ReadFile to load the content of a file into a `[]byte`.

```go

package main

import (
  "io/ioutil"
   ...
)

func main() {
    bytes, err := ioutil.ReadFile("./planets.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Printf("%s", bytes)
}
```

## Conclusion

This writeup shows how to use the `io.Reader` and `io.Writer` interfaces to implement streaming IO in your program. After reading this write up you should be able to understand how to create programs that use the io package to stream data for IO. There are plenty of examples and the writeup shows you how to create your own `io.Reader` and `io.Writer` types for custom functionalities.

This is an introductory discussion and barely scratches the surface of the breath of and scope of Go packages that support streaming IO. We did not go into file IO, buffered IO, network IO, or formatted IO, for instance (saved for future write ups). I hope this gives you an idea of what is possible with the Go’s streaming IO idiom.

As always, if you find this writeup useful, please let me know by clicking on the clapping hands 👏 icon to recommend this post.

Also, don’t forget to checkout my book on Go, titled Learning Go Programming from Packt Publishing.


## 参考和抄袭:




