---
title: 如何避免 Go 命令行执行产生“孤儿”进程？
key: 2022-09-30
tags: go exec.command
---

在 Go 程序当中，如果我们要执行命令时，通常会使用 `exec.Command` ，也比较好用，通常状况下，可以达到我们的目的，如果我们逻辑当中，需要终止这个进程，则可以快速使用 `cmd.Process.Kill()` 方法来结束进程。但当我们要执行的命令会启动其他子进程来操作的时候，会发生什么情况？

<!--more-->

## 0x01 孤儿进程的产生

测试小程序：

```go
package main

import (
  "fmt"
  "os/exec"
  "time"
)

func kill(cmd *exec.Cmd) func() {
  return func() {
    if cmd != nil {
      cmd.Process.Kill()
    }
  }
}

func main() {
  cmd := exec.Command("/bin/bash", "-c", "watch top > top.log")
  time.AfterFunc(1*time.Second, kill(cmd))
  err := cmd.Run()
  fmt.Printf("pid=%d, err=%s\n", cmd.Process.Pid, err)
}


```

执行小程序：

```bash
go run mycmd2.go
pid=3294605, err=signal: killed
```

查看进程信息：

```bash
ps -jaf
UID          PID    PPID    PGID     SID  C STIME TTY          TIME CMD
chengch+ 3290843 3279830 3290843 3279830  0 20:57 pts/0    00:00:00 bash
chengch+ 3290868 3290843 3290868 3279830  0 20:57 pts/0    00:00:00 bash
chengch+ 3295137       1 3295101 3279830  0 23:52 pts/0    00:00:00 watch top
chengch+ 3295296 3290868 3295296 3279830  0 23:54 pts/0    00:00:00 ps -jaf
```

可以看到这个 "watch top" 的 PPID 为 1，说明这个进程已经变成了 “孤儿” 进程。

那为什么会这样，这并不符合我们预期，那么可以从 Go 的文档中找到答案：

> Kill causes the Process to exit immediately. Kill does not wait until the Process has actually exited.
> **This only kills the Process itself, not any other processes it may have startd**.

## 0x02 通过进程组来解决掉所有子进程

在 linux 当中，是有会话、进程组和进程组的概念，并且 Go 也是使用 linux 的 kill(2) 方法来发送信号的，那么是否可以通过 kill 来将要结束进程的子进程都结束掉？

linux 的 kill(2) 的定义如下：

```c
#include <signal.h>

int kill(pid_t pid, int sig);
```

并在方法的描述中，可以看到如下内容：

> The `kill()` system call can be used to send any signal to any process group or process.
> If `pid` is positive, then signal `sig` is sent to the process with the ID specified by `pid`.
> 
> If `pid` equals 0, then `sig` is send to every process in the process group of the calling process.
> 
> If `pid` equals -1, then `sig` is sent to every process for which the calling process has permission to send signals, except for process 1(init), but see below.
> 
> If `pid` is less then -1, then `sig` is sent to every process in the process group whose ID is `-pid`.
>
> If `sig` is 0, then no signal is sent, but existence and permission checks are still performed; this can be used to check for the existence of a process ID or process group ID that the caller is permitted to signal.

如果 pid 为正数的时候，会给指定的 pid 发送 sig 信号，如果 pid 为负数的时候，会给这个进程组发送 sig 信号，那么我们可以通过进程组来将所有子进程退出掉？改一下 Go 程序中 kill 方法：

```go
package main

import (
  "fmt"
  "os/exec"
  "syscall"
  "time"
)

func kill(cmd *exec.Cmd) func() {
  return func() {
    if cmd != nil {
      // cmd.Process.Kill()
      syscall.Kill(-cmd.Process.Pid, syscall.SIGKILL)
    }
  }
}

func main() {
  cmd := exec.Command("/bin/bash", "-c", "watch top > top.log")
  time.AfterFunc(1*time.Second, kill(cmd))
  err := cmd.Run()
  fmt.Printf("pid=%d, err=%s\n", cmd.Process, err)
}

```

再次执行：

会发现程序卡住了，我们来看一下当前执行的进程：

```bash
ps -jaf
UID          PID    PPID    PGID     SID  C STIME TTY          TIME CMD
chengch+ 3290843 3279830 3290843 3279830  0 Sep30 pts/0    00:00:00 bash
chengch+ 3290868 3290843 3290868 3279830  0 Sep30 pts/0    00:00:00 bash
chengch+ 3297114 3290868 3297114 3279830  1 00:06 pts/0    00:00:00 go run mycmd3.go
chengch+ 3297145 3297114 3297114 3279830  0 00:06 pts/0    00:00:00 /tmp/go-build568977363/b001/exe/mycmd3
chengch+ 3297149 3297145 3297114 3279830  0 00:06 pts/0    00:00:00 /bin/bash -c watch top > top.log
chengch+ 3297150 3297149 3297114 3279830  0 00:06 pts/0    00:00:00 watch top
chengch+ 3297195 3281860 3297195 3281860  0 00:07 pts/1    00:00:00 ps -jaf

```

可以看到我们 go run 产生了一个子进程 3297145 3297145 产生了 3297149 （watch top >top.log）进程，3297149 产生了 3297149 （watch top）进程。那为什么没有将这些子进程都关闭掉呢？

其实之类犯了一个低级错误，从上图中，我们可以看到他们的进程组 ID 为 3297114，但是我们传递的是 cmd 的 id 即 3297145，这个并不是进程组的 ID，因此程序并没有 kill，导致 cmd.Run() 一直在执行。

在 Linux 中，进程组中的第一个进程，被称为进程组 Leader，同时这个进程组的 ID 就是这个进程的 ID，从这个进程中创建的其他进程，都会继承这个进程的进程组和会话信息；从上面可以看出 go run mycmd3.go 程序 PID 和 PGID 同为 3297114，那么这个进程就是进程组 Leader，我们不能 kill 这个进程组。

那么我们给要执行的进程，新建一个进程组，在 Kill 不就可以了嘛。在 linux 当中，通过 `setpgid` 方法来设置进程组 ID，定义如下：

```c
#include <unistd.h>

int setpgid(pid_t pid, pid_t pgid);
```

如果将 pid 和 pgid 同时设置成 0，也就是 setpgid(0,0)，则会使用当前进程为进程组 leader 并创建新的进程组。

那么在 Go 程序中，可以通过 `cmd.SysProcAttr` 来设置创建新的进程组，修改后的代码如下：

```go
package main

import (
  "fmt"
  "os/exec"
  "syscall"
  "time"
)

func kill(cmd *exec.Cmd) func() {
  return func() {
    if cmd != nil {
      // cmd.Process.Kill()
      syscall.Kill(-cmd.Process.Pid, syscall.SIGKILL)
    }
  }
}

func main() {
  cmd := exec.Command("/bin/bash", "-c", "watch top >top.log")
  cmd.SysProcAttr = &syscall.SysProcAttr{
    Setpgid: true,
  }

  time.AfterFunc(1*time.Second, kill(cmd))
  err := cmd.Run()
  fmt.Printf("pid=%d err=%s\n", cmd.Process.Pid, err)
}
```

再次执行发现 watch 的进程都不存在了.

## 0x03 子进程监听父进程是否退出(只能在 linux 下执行)

假设要调用的程序也是我们自己写的其他应用程序，那么可以使用 Linux 的 prctl 方法来处理， prctl 方法的定义如下：

```c
#include <sys/prctl.h>

int prctl(int option, unsigned long arg2, unsigned long args,
          unsigned long arg4, unsigned long arg5);
```

这个方法有一个重要的 `option：PR_SET_PDEATHSIG`，通过这个来接收父进程的退出。

让我们来再次构造一个有问题的程序。

有两个文件，分别为 main.go 和 child.go 文件，main.go 会调用 child.go 文件。

main.go 文件：

```go
package main

import (
        "os/exec"
)

func main() {
        cmd := exec.Command("./child")
        cmd.Run()
}
```

child.go 文件：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    for {
    time.Sleep(200 * time.Millisecond)
    fmt.Println(time.Now())
    }
}
```

在 Linux 环境中分别编译这两个文件：

```bash
# 编译 main.go 生成 main 二进制文件
go build -o main main.go

# 编译 child.go 生成 child 二进制文件
go build -o child child.go
```

执行 main 二进制文件：

```bash
./main &
```

查看他们的进程：

```bash
ps -ef

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 06:05 pts/0    00:00:00 /bin/bash
root     11514     1  0 12:12 pts/0    00:00:00 ./main
root     11520 11514  0 12:12 pts/0    00:00:00 ./child
```

可以看到 main 和 child 的进程，child 是 main 的子进程，我们将 main 进程 kill 掉，再查看进程状态：

```bash
kill -9 11514

ps -ef

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 06:05 pts/0    00:00:00 /bin/bash
root     11520     1  0 12:12 pts/0    00:00:00 ./child
```

我们可以看到 child 的进程，他的 PPID 已经变成了 1，说明这个进程已经变成了孤儿进程。

那接下来我们可以使用 `PR_SET_PDEATHSIG` 来保证父进程退出，子进程也退出，大致方式有两种：使用 CGO 调用和使用 `syscall.RawSyscall` 来调用。

### 1, 使用 CGO

将 child 修改成如下内容：

```go
import {
  "fmt"
  "time"
}
// #include <stdio.h>
// #include <stdlib.h>
// #include <signal.h>

// static void killTest() {
//   prctl(PR_SET_PDEATHSIG, SIGKILL);
// }

import "C"

func main() {
  C.killTest()
  for {
    time.Sleep(200 * time.Millisecond)
    fmt.Println(time.Now())
  }
}
```

程序中，使用 CGO，为了简单的展示，在 Go 文件中编写了 C 的 killTest 方法，并调用了 prctl 方法，然后在 Go 程序中调用 killTest 方法，让我们重新编译执行一下，再看看进程：

```bash
go build -o child child.go
./main & 
ps -ef 

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 06:05 pts/0    00:00:00 /bin/bash
root     11663     1  0 12:28 pts/0    00:00:00 ./main
root     11669 11663  0 12:28 pts/0    00:00:00 ./child
```

再次 kill 掉 main，并查看进程：

```bash
kill -9 11663
ps -ef

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 06:05 pts/0    00:00:00 /bin/bash
```

可以看到 child 的进程也已经退出了，说明 CGO 调用的 prctl 生效了。

### 2, syscall.RawSyscall 方法

也可以采用 Go 中提供的 `syscall.RawSyscall` 方法来替代调用 CGO，在 Go 的文档中，可以查看到 syscall 包中定义的常量（查看 linux，如果是本地 godoc，需要指定 GOOS=linux），可以看到我们要用的几个常量以及他们对应的数值：

```go
// 其他内容省略掉了
const(
    ....
    PR_SET_PDEATHSIG                 = 0x1
    ....
)

const(     
    .....
    SYS_PRCTL                  = 157
    .....
)
```

其中 PR_SET_PDEATHSIG 操作的值为 1，SYS_PRCTL 的值为 157，那么将 child.go 修改成如下内容：

```go
package main

import (
    "fmt"
    "os"
    "syscall"
    "time"
)

func main() {
    _, _, errno := syscall.RawSyscall(uintptr(syscall.SYS_PRCTL), uintptr(syscall.PR_SET_PDEATHSIG), uintptr(syscall.SIGKILL), 0)
    if errno != 0 {
    os.Exit(int(errno))
    }

    for {
    time.Sleep(200 * time.Millisecond)
    fmt.Println(time.Now())
    }
}
```

再次编译并执行：

```bash
go build -o child child.go
./main & 
ps -ef

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 06:05 pts/0    00:00:00 /bin/bash
root     12208     1  0 12:46 pts/0    00:00:00 ./main
root     12214 12208  0 12:46 pts/0    00:00:00 ./child
```

将 main 进程结束掉：

```bash
kill -9 12208
ps -ef

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 06:05 pts/0    00:00:00 /bin/bash
```

child 进程已经退出了，也达成了最终效果。

## 0x04 总结

当我们使用 Go 程序执行其他程序的时候，如果其他程序也开启了其他进程，那么在 kill 的时候可能会把这些进程变成孤儿进程，一直执行并滞留在内存中。当然，如果我们程序非法退出，或者被 kill 调用，也会导致我们执行的进程变成孤儿进程，那么为了解决这个问题，从两个思路来解决：

- 给要执行的程序创建新的进程组，并调用 syscall.Kill，传递负值 pid 来关闭这个进程组中所有的进程（比较完美的解决方法）。
- 如果要调用的程序也是我们自己编写的，那么可以使用 PR_SET_PDEATHSIG 来感知父进程退出，那么这种方式需要调用 Linxu 的 prctrl，可以使用 CGO 的方式，也可以使用 syscall.RawSyscall 的方式。

但不管使用哪种方式，都只是提供了一种思路，在我们编写服务端服务程序的时候，需要特殊关注，防止孤儿进程消耗服务器资源。

## 0x05 原文链接

作者 | 昕希 <br />
来源 | 阿里技术公众号 <br />
链接 | <https://developer.aliyun.com/article/786841?utm_content=m_1000289494> <br />

EOF
