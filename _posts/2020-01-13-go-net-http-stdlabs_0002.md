---
title: go 语言学习笔记之(模版)[0002]
key: 2020-01-13
tags: go go-lang text/template html/template
---

As The Titile



本文是我学习《 Go Web编程》的学习笔记.

Web 模版引擎演变自 SSI（服务器端包含）技术，并最终衍生出了注入 PHP、ColdFusion 和 JSP 这样的 Web 编程语言。



<!--more-->



模版中被两个大括号包围的点 （`.`) 是一个动作，它指示模版引擎在执行模版时，使用一个值去替换这个动作本身。

```html
<body>
    {{ . }}
</body>
```



使用 Go 的 Web 模版引擎需要以下两个步骤：



1. 对文本格式的模版源进行语法分析，创建一个经过语法分析的模版结构，其中模版源既可以是一个字符串，也可以是模版文件包中包含的内容；
2. 执行经过语法分析的模版，将 ResponseWriter 和模版所需的动态数据传递给模版引擎，被调用的模版引擎会把经过语法分析的模版和传入的数据结合起来，生成出最终的 HTNML，并将这些 HTML 传递给 ResponseWriter。



```go
import (
    "net/http"
    "html/template"
)

func process(w http.ResponseWriter, r *http.Request) {
    t, _ := template.ParseFiles("tmpl.html")
    t.Execute(w, "Hello World!")
}
```







## 动作



Go 模版的动作就是一些嵌入在模版里面的命令.



这些命令在模版中使用两个大括号`{{}}` 进行包围.



### 条件动作



条件动作会根据参数的值来决定对多条语句中的哪一条语句进行求值.



```go
{{ if arg }}
some content
{{ end }}

{{ if arg }}
  some content
{{ else }}
  other content
{{ end }}
```





### 迭代动作



迭代动作可以对数组、切片、映射或者通道进行迭代， 而在迭代循环的内部， 点则会被设置为当前被迭代的元素。



```
{{ range array }}
  Dot is set to the element {{ . }}
{{ end }}

{{ range . }}
  <li>{{ . }}</li>
{{ else }}
  <li>Nothing to show</li>
{{ end }}
```



### 设置动作



设置动作允许用户在指定的范围内为点设置值.



```
{{ with arg }}
  Dot is set to arg
{{ end }}

<div>
{{ with "world" }}
Now the dot is set to {{ . }}
{{ end }}
```





###    包含动作



包含动作 (include action) 允许用户在一个模版里面包含另一个 模版, 从而构建出嵌套的模版.



包含动作的格式为:

```
{{ template "name" }}
{{ template "name" arg }}

<body>
<hr>
{{ template "t2.html" }}
<hr />
</body>
```





## 参数、变量和管道



一个参数（argument）就是模版中的一个值。它可以是布尔值、整数、字符串等字面量，也可以是结构、结构中的一个字段或者数组中的一个 Key。还可以是一个变量、一个方法（这个方法必须只返回一个值， 或者只返回一个值和一个错误）或者一个函数。



参数也可以是一个点， 用于表示处理器向模版引擎传递的数据。



除了参数， 还可以在动作中设置变量。变量是以美元符号开头：

```
$variable := value
```



**管道**



```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title>Go Web</title>
    </head>
    <body>
        {{ 3.1419526 | printf "%.2f"}}
    </body>
</html>
```



## 函数



用户可以自定义自己想要的函数。



> **注意**  Go 的模版引擎函数都是受限制的：这些函数可以接收任意多个参数作为输入，但是他们只能返回一个值，或者返回一个值和一个错误。



为了创建一个自定义模版函数，需要：



1. 创建一个名为 `FuncMap` 的映射，并将映射的键设置为函数的名字，而映射的值则设置为实际定义的函数；
2. 将 `FuncMap` 与模版进行绑定。



```go
import (
    "net/http"
    "html/template"
    "time"
)

func formatDate(t time.Time) string {
    layout := "2006-01-02"
    return t.Format(layout)
}

func process(w http.ResponseWriter, r *http.Request) {
    funcMap := template.FuncMap{
        "fdate" : formatDate
    }
    t := template.New("tmpl.html").Funcs(funcMap)
    t, _ = t.ParseFiles("tmpl.html")
    t.Execute(w, time.Now())
}
```



html:



```html
<div>
    The date/time is  {{ . | fdate}}
</div>
<div>
    The date/time is {{ fdate . }}
</div>
```



## 嵌套模版



还是再看看书吧...























<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
