---
title: Java 执行操作系统的命令
key: 2025-05-17
tags: java cmd command
---

在Java中，可以通过两种主要方式来调用外部可执行程序或系统命令：`Runtime.getRuntime().exec` 方法和`ProcessBuilder` 类。

<!--more-->

## 0x01 使用 Runtime.getRuntime().exec

`Runtime.getRuntime().exec` 方法允许 Java 虚拟机创建一个子进程来执行指定的可执行程序或命令，并返回一个 `Process` 对象， 该对象可以用来控制子进程的执行或获取子进程的信息。

例如，

```java
Process process = Runtime.getRuntime().exec("cmd");
process.waitFor();

```

这里的 "cmd" 是要执行的命令。 `waitFor()` 方法是为了等待子进程完成以后再继续执行。如果子进程尚未终止，则调用线程将被阻塞，直到子进程推出。


## 0x02 使用 ProcessBuilder

`ProcessBuilder` 类是 Java 5 中引入的，用于创建操作系统进程。它提供了一种启动和管理进程的方法。`ProcessBuilder` 的构造方法接收一个命令参数的数组形式，第一个元素代表要执行的系统命令， 后面的元素代表要传给该命令的参数。

例如：

```java
List<String> cmd = new ArrayList<>();
cmd.add("cmd");
ProcessBuilder pb = new ProcessBuilder(cmd);
pb.redirectErrorStream(true);
Process process = pb.start();

```
- `redirectErrorStream(true)` 的作用是合并错误流和标准输出流， 这样就可以只使用 `Process.getInputStream()` 来读取所有输出。

### 0x03 注意事项

在使用这些方法时，需要注意处理输出流以避免阻塞。例如，可以使用 `BufferedReader` 来读取 `Process` 的输出流：

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream(), "GBK")));
String line = null;
while ((line = reader.readLine()) != null) {
    System.out.println(line);
}

```

此外，不同的操作系统可能需要不同的命令解释器。例如在 Windows 上可能要调用 cmd.exe ， 在 Linux 上可能要调用 /bin/bash.

```java
String os = System.getProperty("os.name");
ProcessBuilder builder;
String charset;
if (os.toLowerCase().contains("win")) {
    builder = new ProcessBuilder("cmd", "/c", exportCmd.toString());
    charset = "gbk";
} else {
    builder = new ProcessBuilder("sh", "-c" exportCmd.toString());
    charset = "utf-8";
}
```

在调用外部命令时，还需要考虑到命令执行完成后的自动关闭，以及如何处理不同系统执行命令的差异。

