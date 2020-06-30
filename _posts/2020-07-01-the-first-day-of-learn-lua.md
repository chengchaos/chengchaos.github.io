---
title: 学习 Lua 的第一天
key: 2020-07-01
tags: lua
---

我想起了我最开始学习 asp 的时候，看到的 10 天精通 ASP 的一系列文章，我还特意都保存了下来，回去慢看看。

现在我貌似要进行 10 天精通 lua 了。

## 简介


Lua 有 5.1，5.2，5.3 三个主要版本，各版本之间有一些语法上的差异，不完全兼容，由于 OpenResty 使用的是 Lua 5.1 + LuaJIT 。 因此我选择从学习 Lua 5.1 开始。


<!--more-->

## 注释

单行注释使用简单的双横线 `--` 例如：

```lua
-- this is a comment
-- 这是中文注释
print("hello lua") -- 行尾注释。
```

多行祝生意使用 `--[[` 开头 `]]` 结尾：

```lua
--[[
  多行注释，写啥都行。除了两个连续的方括号。
]]
```

多行注释可以在开始和结尾的方括号中间加入若干个等号 `=` 

```lua
--[==[
  [[这样就可以在注释里面写连续的两个方括号了]]
  Yes
]==]
```


## 数据类型

Lua 语言提供了六种基本的数据类型：

- nil : 表示不存在的空对象或无效值，类似 Python 的 None；
- boolean ： 布尔类型，取值为 true 或者 false
- number ： 数字类型，不区分整数和浮点数 （5.3 开始引入了证书，并支持位运算）
- function ： 函数类型
- table ： 表类型。

使用函数 `type()` 可以测试变量的类型，它以字符串的形式返回类型的名称：

```lua
print(type(nil)) -- nil
print(type(true)) -- boolean
print(type(42)) -- number
print(type(2.782)) -- number
print(type("hellowrold")) -- string
print(type(print)) -- function
print(type(table)) -- table (table 是 Lua 标准库里的一个表)
```
Lua 是动态语言，所以生命变量不需要显式地写出类型，变量也可以存储任意类型的值：

```lua
x = 2020 -- number
x = "lua" -- string
x = nil -- nil


```

## string

Lua 可以搞笑地处理字符串，几 kb 或者 mb 的长字符串也不会对效率造成影响，可以放心地使用字符串存储大块的数据。

**定义：**

单引号或者双引号，支持转转义字符，还可以使用两个方括号的形式支持 raw string：


 ```lua
print('openresty')
print("It's work")
print("lua\tnginx")
print([[raw string \r\n]]) -- 这里的 \r\n 不会被转义
print([[^\d+.\d+$]])
print([["", '', ""]]) -- 引号无需转义

x = [=[ -- 这里的换行符不会包含在字符串里面
 [[no \r\n , just one line]] 
]=]

 ```

**特点：**

Lua 里的字符串更准确地说应该是“字符序列”，不仅可以包含可见字符，还能够包含任意的二进制数据。

Lua 字符串的另一个提点是只读的，类似 Java。

Lua 在内部使用一个全局散列表来管理所有的字符串，所以多个相同的字符串不会占用多份内存，而且字符串的相等比较成本很低，不需要逐个检查里面的字符，而是直接比较两者的散列值。


## 变量

lua 里面的变量有作用域的感念，分为局部变量和全局变量，名字区分大小写。

局部变量需要使用 `local` 关键字声明，作用域仅限本代码块（文件内或者语句块内）。

没有使用 `local` 色号干嘛的变量是全局变量不需要 生命就可以直接使用。

变量没有显式赋值，那么他的值就是 `nil`  。


```lua
x = 1
local str = "matrix"

do 
  local pi = 3.14
end

print(type(pi)) -- 局部变量消失，访问的是全局变量 pi

```

一个比较常用的全局变量是 `_` （下划线），通常用作占位符。暂存某些值并忽略。


Lua 中没有常量，实践中我们通常用全大写名字的变量来表示常量：

 ```lua
local MAX_COUNT = 1000 -- 全大侠变量，提醒开发者应当当作常量来使用。

 ```

## 运算符


### 数学运算


**支持基本的四则运算符**。

- 取模 ： `%` 
- 幂运算： `^` : 

**除法会返回小数**。

**不支持  `++` 和 `--` 操作**。


### 关系运算


**支持关系运算符**

注意； **不等使用的是： `~=`**


比较大小时，类型不同会报错。

比较等于或者不等于时，类型不同会返回 false。

常见的情况时比较字符串和数字，为了避免发生意外，必须使用函数 `tonumber()` 或 `tostring()` 显式转换成数字或者字符串后在做比较。


### 逻辑运算


Lua 的逻辑运算符有： `and` `or` `not` 三个。规则有些特殊：

- `nil` 和 `false` 是假，其他都是真，包括数字 0.
- `x and y` ， 如果 x 是真，返回 y， 否则返回 x；
- `x or y` , 与 and 操作相反，如果 x 是真，返回 x， 否则返回 y；
- `not x` , 只返回 true / false ， 对 x 取反。


利用 Lua 逻辑运算符的特性可以实现非常灵活的赋值功能：

```lua
-- 为 x 赋初始值， count 不存在则默认值是 100
local x = count or 100

-- 相当于 a ? b : c
local y = a and b or c

```

### 字符串

连接字符串使用 `..` , 可以自动把数字转成字符串：

```lua
pirnt("hello" .. " " .. "world")

print("room number is " .. 404)


计算字符串长度使用 `#` :

```lua
print(#'openresty') -- 计算字符产长度， 输出 9


```

### 其他

在匀速教案是我们需要注意的是操作数是 `nil` 的情况，很多对 nil 的运损都会导致错误：

```lua
x = nil
print(1 + x)                   
print("msg is " .. x)
```

如果一个变量可能是 nil， 做好使用 `or` 运算符给他一个默认值：

```lua
print(1 + (x or 2))

print("msg is " .. ( x or '-'))
```






---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





