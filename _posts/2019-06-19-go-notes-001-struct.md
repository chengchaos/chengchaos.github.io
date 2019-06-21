---
title: Go 学习笔记 001 结构体
key: 2019-06-19
tags: go
2000 年一月一日的样子我现在还记得.
---

2000 年一月一日的样子我现在还记得.

## 定义



结构体是将零个或者多个任意类型的命令变量组合在一起的聚合数据类型。每个变量都叫做结构体的成员。



简单理解，Go 语言的结构体 `struct` 和其他语言的类 `class` 有相等的地位，但是 Go 语言放弃了包括继承在内的大量面向对象的特性，值保留了组合这个基础的特性。



所有的 Go 语言类型除了指针类型外，都可以有自己的方法。

### 一个例子



```go
package main

import (
    "fmt"
)

type Student struct {
    Name string
    Age int
    Sex string
    Score int
}

func testStruct() {
    var stu Student
    stu.Name = "Tom"
    stu.Age = 28
    stu.Sex = "F"
    stu.Score = 108
    
    fmt.Printf("name:%s, age:%d, score:%d sex:%s\n", stu.Name, stu.Age, stu.Score, stu.Sex)
    fmt.Printf("%+v\n", stu)
    fmt.Printf("%v\n", stu)
    fmt.Printf("%#v\n", stu)
}

func main() {
    testStruct()
}
```



输出:



```

```



关于 Go 中的 struct :



1. 用于定义复杂的数据结构
2. struct 里面可以包含多个字段（属性），字段可以是任意类型。
3. struct 类型可以定义方法（注意和函数的区别）
4. struct 类型时值类型
5. struct 类型可以嵌套
6. Go 语言没有 class 类型，只有 struct 类型

## 语法说明



struct 声明：



```go

type <标识符> struct {
    field1 type1
    field2 type2
}
```



struct 中字段的访问和其他语言一样使用圆点 `.` 这个符号：



```go
var stu Student
stu.Name = "tom"
stu.Age = 28
```



### struct 定义的三种形式



```go
type Student struct {
    Name string
    Age int
}
```



对于上面这个结构体，我们有三种定义方式：



```go
// 1:
var stu Student

// 2:
var stu *Student = new(Student)

// 3:
var stu *Student = &Student
```



方法 2 和 方法 3 的效果是一样的，都是返回指向结构体的指针， 访问方式如下：



```go
stu.Name
stu.Age

(*stu).Name
(*stu).Age
// 而这种方法中可以换成上面的方法直接通过 stu.Name 访问
// 这里是go替我们做了转换了，当我们通过stu.Name访问访问的时候，
// go 会先判断stu是值类型还是指针类型如果是指针类型，会替我们改成(*stu).Name
```



struct 中所有字段的内存是连续的

Go 中的 struct 没有构造函数，一般通过工厂模式来解决,通过下面例子理解：



```go
package main

import (
    "fmt"
)

type Student struct{
    Name string
    Age int
}

func NewStudent(name string,age int) *Student {
    return &Student {
        Name:name,
        Age:age,
    }
}

func main(){
    s := NewStudent("tom",23)
    fmt.Println(s.Name)
}
```



## struct 中的 tag



~~未完待续~~



参考：[Go基础之--结构体和方法](https://www.cnblogs.com/zhaof/p/8244542.html)

<!--more-->



<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
