---
title: Golang 中执行 cmd 命令
key: 2020-08-10
tags: go Command Cmd
---

such as the title said.

<!--more-->

## Command 方法

go 执行操作系统命令使用 os/exec Command 函数：

```go

func Command(name string, arg ...string) *Cmd
```

其中第一个参数是命令名称，后面的参数是命令的参数。

For example：


```go

func Test_RunCommand001(t *testing.T) {


	cmd := exec.Command("java", "-h")
	output, err := cmd.CombinedOutput()
	if err != nil {
		log.Fatalln(err)
	}

	log.Println("result =>", string(output))

}

```

上面这种使用方式虽然能获取到外部命令的执行结果输出 output，但是必须得命令执行完成后才将获取到的结果一次性返回。很多时候我们是需要实时知道命令执行的输出，比如我们调用一个外部的服务构建命令： mvn build，这种情况下实时输出命令执行的结果对我们来说很重要。再比如我们 ping 远程 IP，需要知道实时输出，如果直接 ping 完在输出，在使用上来说体验不好。

要实现外部命令执行结果的实时输出，需要使用 Cmd 结构的 StdoutPipe() 方法创建一个管道连接到命令执行的输出，然后用 for 循环从管道中实时读取命令执行的输出并打印到终端。具体代码如下：


```go
func RunCommand(name string, arg ...string) error {
	cmd := exec.Command(name, arg...)
    // 命令的错误输出和标准输出都连接到同一个管道
	stdout, err := cmd.StdoutPipe()
	cmd.Stderr = cmd.Stdout

	if err != nil {
		return err
	}

	if err = cmd.Start(); err != nil {
		return err
	}
    // 从管道中实时获取输出并打印到终端
	for {
		tmp := make([]byte, 1024)
		_, err := stdout.Read(tmp)
		fmt.Print(string(tmp))
		if err != nil {
			break
		}
	}

	if err = cmd.Wait(); err != nil {
		return err
	}
	return nil
}
```




## 重定向

```go
func Test_RunCommandRedirect(t *testing.T) {

	//var stdout io.ReadCloser
	//var err error

	stdout, err :=os.OpenFile("stdout.log", os.O_CREATE|os.O_WRONLY, os.FileMode.Perm(0600))
	if err != nil {
		log.Fatalln(err)
	}
	defer stdout.Close()


	// 重定向标准输出到文件
	cmd := exec.Command("java.exe", "-h")
	cmd.Stdout = stdout

	// 执行命令
	if err := cmd.Start(); err != nil {
		log.Fatalln(err)
	}
}
```

## cmd 的 Start 方法和 Run 方法的区别

Start 方法不会等待命令完成。
Run 方法回阻塞等待命令完成。

```go

```


## 运行时隐藏窗口

```go
go build -ldflags -H=windowsgui 


cmd := exec.Command("sth")
if runtime.GOOS == "windows" {
    cmd.SysProcAttr = &syscall.SysProcAttr{HideWindow: true}
}
err := cmd.Run()


```

参考：

- https://blog.csdn.net/youngwhz1/article/details/88662172
- https://blog.csdn.net/qianghaohao/article/details/96614079?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.opensearch_close&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.opensearch_close



EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





