---
title: Python asyncio 教程
key: 2026-01-10
tags: Python asyncio
---

Python asyncio 教程

`asyncio` 是 Python 3.4 引入的标准库，用于支持异步 I/O 操作。它的编程模型是一个消息循环，通过 `EventLoop` 执行协程，实现异步 I/O。

`asyncio` 模块内部实现了 `EventLoop` ，把需要执行的协程扔到 `EventLoop` 中执行，就实现了异步 IO。

<!--more-->

## 0x01 基本用法

使用 asyncio 实现异步操作的基本步骤如下：

1. 定义协程：使用 `async def` 定义一个协程函数。
2. 运行协程：使用 `asyncio.run()` 运行协程。

例如，以下代码展示了如何使用 asyncio 实现一个简单的 "Hello World" 程序：

```python
import asyncio

async def hello():
    print("Hello world!")
    await asyncio.sleep(1)
    print("Hello again!")

asyncio.run(hello())

```

在这个例子中，hello() 函数首先打印 "Hello world!"，然后异步等待 1 秒，再打印 "Hello again!"。

为了简化并更好地标识异步 IO，从 Python 3.5 开始引入了新的语法 `async` 和 `await` ，可以让 coroutine 的代码更简洁易读。

`async` 把一个函数变成 coroutine 类型，然后，我们就把这个 async 函数 ( coroutine ) 扔到 `asyncio.run()` 中执行。

`await` 语法可以让我们方便地调用另一个 `async` 函数。由于 `asyncio.sleep()` 也是一个 `async` 函数，所以线程不会等待`asyncio.sleep()`，而是直接中断并执行下一个消息循环。当 `asyncio.sleep()` 返回时，就接着执行下一行语句。

把 `asyncio.sleep(1)` 看成是一个耗时 1 秒的 IO 操作，在此期间，主线程并未等待，而是去执行 `EventLoop` 中其他可以执行的 `async` 函数了，因此可以实现并发执行。


## 0x02 并发执行

`asyncio` 可以并发执行多个协程。使用 `asyncio.gather()` 可以同时调度多个协程。例如：

```python
import asyncio
import threading

# 传入name参数:
async def hello(name):
    # 打印name和当前线程:
    print("Hello %s! (%s)" % (name, threading.current_thread))
    # 异步调用asyncio.sleep(1):
    await asyncio.sleep(1)
    print("Hello %s again! (%s)" % (name, threading.current_thread))
    return name

# 用asyncio.gather()同时调度多个async函数：
async def main():
    L = await asyncio.gather(hello("Bob"), hello("Alice"))
    print(L)

asyncio.run(main())

```

执行结果如下：

```sh
(.venv) PS D:\works\VW\OCSP\mypy> & D:/works/VW/OCSP/mypy/.venv/Scripts/python.exe d:/works/VW/OCSP/mypy/a2.py
Hello Bob! (<function current_thread at 0x000001E5BA62F8A0>)
Hello Alice! (<function current_thread at 0x000001E5BA62F8A0>)
Hello Bob again! (<function current_thread at 0x000001E5BA62F8A0>)
Hello Alice again! (<function current_thread at 0x000001E5BA62F8A0>)
['Bob', 'Alice']
(.venv) PS D:\works\VW\OCSP\mypy> 
```

从结果可知，用 `asyncio.run()` 执行 `async` 函数，所有函数均由同一个线程执行。两个 `hello()` 是并发执行的，并且可以拿到 `async` 函数执行的结果（即 `return` 的返回值）。

如果把 `asyncio.sleep()` 换成真正的 IO 操作，则多个并发的 IO 操作实际上可以由一个线程并发执行。

## 0x03 异步网络请求

`asyncio` 还可以用于异步网络请求。以下代码展示了如何使用 asyncio 获取多个网站的首页：

```python
import asyncio

async def wget(host):
    print(f"wget {host}...")
    # 连接80端口:
    reader, writer = await asyncio.open_connection(host, 80)
    # 发送HTTP请求:
    header = f"GET / HTTP/1.0\r\nHost: {host}\r\n\r\n"
    writer.write(header.encode("utf-8"))
    await writer.drain()

    # 读取HTTP响应:
    while True:
        line = await reader.readline()
        if line == b"\r\n":
            break
        print("%s header > %s" % (host, line.decode("utf-8").rstrip()))
    # Ignore the body, close the socket
    writer.close()
    await writer.wait_closed()
    print(f"Done {host}.")

async def main():
    await asyncio.gather(wget("www.sina.com.cn"), wget("www.sohu.com"), wget("www.163.com"))

asyncio.run(main())

```

在这个例子中，三个 wget 函数是并发执行的，分别获取新浪、搜狐和网易的首页。

## 0x04 总结

`asyncio` 提供了完善的异步 I/O 支持，通过 `asyncio.run()` 调度协程。在协程内部，可以使用 `await` 调用其他协程，实现并发执行。通过 `asyncio.gather()` 可以并发执行多个协程。



## Appendix

[使用asyncio](https://liaoxuefeng.com/books/python/async-io/asyncio/index.html)