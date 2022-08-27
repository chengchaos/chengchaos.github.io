---
title: Go command parameters
key: 2022-06-28
tags: go go-lang
---

`os` 包提供一些函数和变量，以平台无关的方式和操作系统交互，命令行参数以 `os` 包中 `Args` 名字的变量提供程序访问， 在 `os` 包之外，使用 `os.Args` 这个名字。

<!--more-->

变量 `os.Args` 是一个字符串切片（slice）。可以通过 `s[i]` 来访问单个元素，通过 `s[m:n]` 来访问一段连续子区间，长度用 `len(s)` 来获取。

> Go 语言中，所有的索引都使用半开区间，即包含第一个元素，不包含最后一个元素。
>
> 例如： `slice s[m:n]` 其中 0 <= m <= n <= len(s)，包含 m - n 个元素。

`os.Args[0]` 是命令本身的名字； 其余的元素是程序开始执行时的参数。



## 实现一个 echo 命令

```go
// gop1.io/ch1/echo1
// echo1 输出其命令行参数
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	echo3()
}

func echo1() {
	var s, sep string
	for i := 1; i < len(os.Args); i++ {
		s += sep + os.Args[i]
		sep = " "
	}
	fmt.Println(s)
}

func echo2() {
	s, sep := "", ""
	for _, arg := range os.Args[1:] {
		s += sep + arg
		sep = " "
	}
	fmt.Println(s)
}

func echo3() {
	fmt.Println(strings.Join(os.Args[1:], " "))
}

```

## 找出重复行

用于文件复制、打印、检索、排序、统计的程序，通常有一个相似的结构：在输入接口上循环读取，软后对每一个元素进行一些计算，在运行时或者结束时输出结果。

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// 内置函数 make 创建 map
	counts := make(map[string]int)
	input := bufio.NewScanner(os.Stdin)
	for input.Scan() {
		counts[input.Text()]++
	}
	// 注意： 忽略 input。Err() 中可能的错误
	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}
}

```

`bufio` 包可以简便和搞笑地处理输入和输出。

其中一个最有用的特性时被称为扫描器的类型（Scanner），它可以读取输入，以行或者单词为单位断开，这是处理以行为单位的输入内容的最简单方式。

```go
input := bufio.NewScanner(os.Stdin)
```

Scanner 从程序的标准输入进行读取。每次调用 `input.Scan()` 读取下一行，并且将结尾的换行符去掉；

通过调用 `input.Text()` 来获取读到的内容。

`Scan()` 函数在读到新行时返回 `true`，在没有更多内容的时候返回 `false`。



```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	dup2()
}

func dup2() {
	counts := make(map[string]int)
	files := os.Args[1:]
	if len(files) == 0 {
		countLines(os.Stdin, counts)
	} else {
		for _, arg := range files {
			f, err := os.Open(arg)
			if err != nil {
        // %v 可以使用默认格式显示任意类型的值；
				fmt.Fprintf(os.Stderr, "dup2: %v\n", err)
				continue
			}
			countLines(f, counts)
			f.Close()
		}
	}
	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}
}

func countLines(f *os.File, counts map[string]int) {
	input := bufio.NewScanner(f)
	for input.Scan() {
		counts[input.Text()]++
	}
}

```

函数 `os.Open` 返回两个值，第一个时打开的文件 (`*os.File`)，该文件随后被 Scanner 读取。第二个返回值时一个内置的 `error` 类型的值。如果 err 等于 nil ，标准文件成功打开。文件在被读到结尾的时候，`Close()` 函数关闭文件释放资源。

`map` 时一个使用 `make` 创建的数据结构的**引用**。当一个 map 被传递给一个函数时，函数接收到这个**引用的副本**，所以被调用函数中对于 map 数据结构中的改变对函数调用者使用的 map 引用也是可见的。



```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
	"strings"
)

func dup3() {

	counts := make(map[string]int)
	for _, filename := range os.Args[1:] {
		data, err := ioutil.ReadFile(filename)
		if err != nil {
			fmt.Fprintf(os.Stderr, "dup3: %v\n", err)
			continue
		}
		for _, line := range strings.Split(string(data), "\n") {
			counts[line]++
		}
	}
	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}
}

func main() {
	dup3()
}

```

`ReadFile` 函数（`io/ioutil`）读取整个文件的内容，返回一个可以转化成字符串的**字节 Slice**。`string.Split` 函数将一个字符串分割为一个由字符串组成的 Slice。

---

If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/)

