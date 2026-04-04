---
title: Go 单元测试
key: 2022-01-10
tags: go CGI
---

单元测试很重要.

<!--more-->

## 规范

Go 语言推荐测试文件和源文件放在一起, 这一点和 Java 不一样, Java 使用 Maven 构建时, 测试文件放在单独的 test/java 目录中, 源文件放在单独的 main/java 目录中.

Go 语言中测试文件使用 `_test.go` 结尾. 例如当前包中有个 `calc.go` 的源文件, 测试文件则为 `calc_test.go` .

```sh
example/
  |-- calc.go
  |-- calc_test.go
```

假设 calc.go 的代码如下:

```go
package main

func Add(a int, b int) int {
    return a + b
}
```

测试文件可以这么写

```go
package main

import "testing"

func TestAdd(t *testing.T) {
    if ans := Add(1, 2); ans != 3 {
        t.Errorf("1 + 2 expected be 3, but %d got", ans)
    }
}
```

- 测试方法名称一般规则是 `Test` 加上待测试的方法名.
- 测试用的参数有且只有一个, `t *testing.T`
- 基准测试(benchmark) 的参数是 `b *testing.B`, TestMain 的参数是 `m *testing.M` .

## 运行

执行 `go test`, 改 package 下的所有测试用例都会被执行.

```bash
$ go test
PASS
ok      github.com/chengchaos/other_train/go_test/example       28.356s

```

执行 `go test -v` , 会显示每个用例的测试结果, 另外 `-cover` 参数可以查看覆盖率.

```bash
$ go test -v
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
PASS
ok      github.com/chengchaos/other_train/go_test/example       22.017s

```

如果只想运行其中的一个用例, 例如 `TestAdd` 可以用  `-run` 参数指定, 该参数支持通配符 `*` 和部分正则表达式, 例如 `^`, `$` .

```bash
$ go test -run TestAdd
PASS
ok      github.com/chengchaos/other_train/go_test/example       21.727s

```

## 子测试 (Subtests)

子测试是 Go 语言内置支持的, 可以在某个测试用例中, 根据测试场景使用 `t.Run` 创建不同的子测试用例, 例如.

```go
// calc_test.go

func TestMul(t *testing.T) {
    t.Run("pos", func(t *testing.T) {
        if Mul(2, 3) != 6 {
            t.Fatal("fail")
        }
    })
    t.Run("neg", func(t *tesging.T) {
        if Mul(2, -3) != -6 {
            t.Fatal("fail")
        }
    })
}
```

前面的例子, 测试失败时使用  `t.Error/t.Errorf` , 这里使用了 `t.Fatal/t.Fatalf` , 区别在于前者遇错不停, 还会执行其他的测试用例, 后者遇错即停.

运行子测试:

```bash
go test -run TestMul/pos -v
```

还没有抄完, 待续.

## 参考(照抄)

- [https://geektutu.com/post/quick-go-test.html](https://geektutu.com/post/quick-go-test.html)

---

If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/)
