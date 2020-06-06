---
title: Go 学习笔记 001 结构体
key: 2019-06-19
tags:  go struct
---

我开始学习 Go 语言了。

<!--more-->

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

我们可以为 struct 中的每个字段，写上一个 tag。这个 tag 可以通过反射的机制获取到，最常用的场景就是 json序列化和反序列化。



我们上面声明的 Student ，Name,Age都是首字母大写的，这样你json序列化的时候才能访问到，如果是小写的，json包则无法访问到，但是序列化后的结果是首字母大写的，但是我就是想要小写怎么办？这里就用到了 tag, 如下，序列化的结果就是小写的了：





```go
package main

import (
    "fmt"
    "encoding/json"
)

type Student struct{
    Name string `json:"name"`
    Age int `json:"age"`
}

func main(){
    var stu Student
    stu.Name = "tom"
    stu.Age = 23

    data,err := json.Marshal(stu)
    if err != nil{
        fmt.Printf("json marshal fail fail error:%v",err)
        return
    }
    fmt.Printf("json data:%s\n",data)
}
```



这里多说一个小知识就是如果我们想要把json后的数据反序列化到struct，其实方法也很简单，只需要在上述的代码后面添加如下：



```go
var stu2 Student
err = json.Unmarshal(data,&stu2)
if err != nil{
    fmt.Printf("json unmarshal fail fail error:%v",err)
    return
}
fmt.Printf("%+v\n",stu2)
```



## 结构体的比较

如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，那样的话，两个结构体将可以使用 `==` 或 `!=` 运算符进行比较。相等比较运算符将比较两个机构体的每个成员
如下面例子：



```go
package main

import (
    "fmt"
)

type Point struct{
    x int
    y int
}

func main(){
    p1 := Point{1,2}
    p2 := Point{2,3}
    p3 := Point{1,2}
    fmt.Println(p1 == p2)  //false
    fmt.Println(p1 == p3) //true

}
```



## 匿名字段

结构体中字段可以没有名字
下面是一个简单的例子：



```go
package main

import (
    "fmt"
)


type Student struct{
    Name string
    Age int
    int
}

func main(){
    var s Student
    s.Name = "tom"
    s.Age = 23
    s.int = 100
    fmt.Printf("%+v\n",s)
}
```



可能上面的这里例子看了之后感觉貌似也没啥用，其实，匿名字段的用处可能更多就是另外一个功能(其他语言叫继承)，例子如下



```go
package main

import (
    "fmt"
)

type People struct{
    Name string
    Age int
}

type Student struct{
    People
    Score int
}

func main(){
    var s Student
    /*
    s.People.Name = "tome"
    s.People.Age = 23
    */
    //上面注释的用法可以简写为下面的方法
    s.Name = "tom"
    s.Age = 23

    s.Score = 100
    fmt.Printf("%+v\n",s)
}
```



注意：关于字段冲突的问题，我们在People中定义了一个Name字段，在Student中再次定义Name,这个时候，我们通过s.Name获取的就是Student定义的Name字段



## 方法

首先强调一下：go 中任何自定义类型都可以有方法，不仅仅是struct

> 注意：除了：指针和 interface

通过下面简单例子理解：



```go
package main

import (
    "fmt"
)

//这里是我们普通定义的一个函数add
func add(a,b int) int {
    return a+b
}

type Int int

//这里是对Int这个自定义类型定义了一个方法add
func (i Int) add(a,b int) int{
    return a+b
}
//如果想要把计算的结果赋值给i
func(j *Int) add2(a,b int){
    *j = Int(a+b)
    return
}

func main(){
    c := add(100,200)
    fmt.Println(c)
    var b Int
    res := b.add(10,100)
    fmt.Println(res)

    var sum Int
    sum.add2(20,20)
    fmt.Println(sum)

}
```



方法的定义：

```go
func (receiver type) methodName(参数列表) (返回值列表) {
}
```





下面是给一个结构体struct定义一个方法



```go
package main

import (
    "fmt"
)


type Student struct{
    Name string
    Age int
}

func (stu *Student) Set(name string,age int){
    stu.Name = name
    stu.Age = age
}

func main() {
    var s Student
    s.Set("tome",23)
    fmt.Println(s)
}
```



> 注意：方法的访问控制也是通过大小写控制的

在上面这个例子中需要注意一个地方 `func (stu *Student)Set(name string,age int)` 这里使用的是 `(stu *Student)` 而不是 `(stu Student)` 这里其实是基于指针对象的方法。



## 基于指针对象的方法



当调用一个函数时，会对其每个参数值进行拷贝，如果一个函数需要更新一个变量，或者函数的其中一个参数实在太大，我们希望能够避免进行这种默认的拷贝，这种情况下我们就需要用到指针了，所以在上一个代码例子中那样我们需要 `func  (stu *Student)Set(name string,age int)` 来声明一个方法

这里有一个代码例子：



```go
package main

import (
    "fmt"
)


type Point struct{
    X float64
    Y float64
}

func (p *Point) ScaleBy(factor float64){
    p.X *= factor
    p.Y *= factor
}

func main(){
    //两种方法
    //方法1
    r := &Point{1,2}
    r.ScaleBy(2)
    fmt.Println(*r)

    //方法2
    p := Point{1,2}
    pptr := &p
    pptr.ScaleBy(2)
    fmt.Println(p)

    //方法3
     p2 := Point{1,2}
     (&p2).ScaleBy(2)
     fmt.Println(p2)

     //相对来说方法2和方法3有点笨拙
     //方法4,go语言这里会自己判断p是一个Point类型的变量，
     //并且其方法需要一个Point指针作为指针接收器，直接可以用下面简单的方法
     p3 := Point{1,2}
     p3.ScaleBy(2)
     fmt.Println(p3)

}
```



上面例子中最后一种方法，编译器会隐式的帮我们用 `&p` 的方法去调用 `ScaleBy` 这个方法，当然这种简写方法只适用于变量，包括 `struct` 里面的字段，如：`p.X` 。

 

所有的努力都值得期许，每一份梦想都应该灌溉！



参考（抄袭）：[Go基础之--结构体和方法](https://www.cnblogs.com/zhaof/p/8244542.html)

<!--more-->



<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
