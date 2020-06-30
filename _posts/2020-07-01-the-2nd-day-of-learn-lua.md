---
title: 学习 Lua 的第二天
key: 2020-07-01
tags: lua
---





<!--more-->

## 表

表（table）是 Lua 中的唯一的数据结构，可以近似地理解为其他编程语言里面的字典/关联数组/Map 等，但是 Lua 中的表更灵活，可以模拟出 array， list， dict， set，map 等数据结构。



### 定义表

Lua 表中的 key 可以是任何非 nil 的值，所以当 key 为整数时，表就相当于数组，为字符串时，表就相当于字典或这关联数组。Lua 表中的 value 的类型没有任何限制。

定义一个表使用花括号： `{}`

```lua
local a = {} -- 一个空表
local b = {3, 5, 7} -- 数组形式
local c = {one = 1, ['two'] = 2} -- 字典形式
local d = {red = 9, [3] = 3, ['a'] = {}} -- 混合形式
```



定义表时的逗号，可以用分号，没有不同。



### 操作表



操作表里的元素需要使用方括号：`[]`

```lua
assert(b[1] == 3 and b[2] == 5)
b[1] = 100
assert(c['one'] == 1)
c['three'] = 3
```

注意当表作为数组来使用时，下标索引必须从 1 开始。

如果 key 时字符串，我们也可以直接使用圆点符号 `.` 来操作，这是表就可以模拟其他语言里的类或者命名空间：



```lua
local x = {}
x['name'] = "samus"
x.job = 'hunter'
print(x.name, " : ", x['job'])

x.mission = function(dst) 
  print('fly to ', dst)
end

x.mission('Zebes')

```



运算符 `#` 可以计算数数组形式素表里面元素数量，配合 for 循环可以实现遍历：

```lua
assert(#a == 3) 
for i = 1, #a do
  print(a[i], ', ')
end
```

对于字典形式的表，没有办法能够直接获取元素的数量，使用 `#` 或返回 0.



### 范围循环



for 循环的范围循环形式主要用于便利表里面的元素，但需要两个标准函数库的配合： `ipairs()` 和 `pairs()`  。这两个函数起到迭代器的作用，逐个返回表里面的 key/value。前者只适合数组形式的表，并且便利到 nil 值就结束； 后者可以支持任意形式的表，并且遍历表里面的锁哟u元素。但速度没有 `ipairs() ` 快。



```lua
for i, v in ipairs(a) do
  print(a[i], ', ')
end

for k, v in pairs(x) do
  print(k, ' => ', v)
end

```



### 作为函数的参数

当表作为函数的参数时传递的是表的引用，因此函数体内可直接修改表的元素。



## 模块



Lua 语言基于表实现了与 C++ 的 namespace，Java 的 package 类似的机制，用户可以用模块来管理组织代码结构。

模块就是一个函数的集合，通常表现为一个 Lua 表，里面有模块作者提供的各种功能函数，使用圆点操作符访问。

使用 `require` 函数可以加载模块，参数是模块所在的文件名（省略后缀），通常我们需要用变量来保存 `require` 函数的返回结果。



```lua
local cjson = require "cjson"
local str = cjson.encode({a=1, b=2})
print(str)
```

编写自己的 Lua 模块也很容易，在源码文件里创建一个表，把函数作为表的元素，最后用 `return` 返回这个表就可以了。



```lua
local proto = {
  version = '0.1'
}

-- 模块内部数据，不对外暴露
local data = 0 

-- 定义一个函数，注意不能是 local 的
function proto.run() 
  print("run in mod")
end

-- 返回模块的表
return proto
```

`require` 在加载模块的同时会执行文件里的代码（进执行一次，所以多次 require 不会影响效率），在形式上，require 的工作相当于：

```lua
-- 定义一个临时函数
local _tmp = function()
  -- 里面是模块文件里的所有的源码
end

-- 执行这个临时函数，最后会返回模块的表
local xxx = _tmp()
```



## 面向对象

如果使用字符串作为 可以， 那么表本身就是对象，可以存储任意变量和函数：

```lua
local x = {}
x.name = 'snake'
x.mission = function() 
  
end
```

Lua 不提供 private， public 这样的修饰词，表里的所有成员都是公开的，如果想要实现私有成员，那么可以在模块文件里用 local 来修饰，这样 local 化的变量就对外界不可见了，而成员函数荏苒可以访问。

**继承** 在 Lua 中不提倡，替代方案是使用 **原型（prototype）模式**，从一个原型对象克隆出一个新对象，然后再动态变更其属性。

###  原型模式

原型模式需要使用 Lua 的高级特性 **元表（metatable）** 和函数 `setmetatable()` 。

元表描述了表的基本行为，有些类似 C++ 或者 Python 里的操作符重载，我们需要用的是 `__index` 方法，他重载了 Lua 里查找 key 的操作，也就是 `table.key`。

函数 `setmetatable(t, meta)` 把表 t 的元表设置为 meta 并返回 t 。如果 meta 里设置了 `__index` 方法，那么对 t 的操作 t.key 也会同样作用到 meta 上，即 `meta.key` 。这样，表 t 就克隆了表 meta 的所有成员，表 meta 就成了表 t 的原型。

```lua
-- 声明一个原型对象，现在是空表：
local proto = {}

-- 为表添加一个方法：
function proto.go()
  print("go pikachu")
end

-- 定义元表，注意重载了 __index
local mt = { __index = proto }

-- 调用 setmetatmable 设置元表，返回新表
local obj = setmetatable({}, mt)

-- 新表对象是原型的克隆，可以执行原型的操作：
obj.go()
```

代码的关键操作是定义元表 mt ，里面只需要设置 `__index` 方法，然后再使用函数 `setmetatable` 从 mt 克隆出一个新对象。

这两个步骤也可以合并为一次操作：

```lua
-- 函数里面直接定义元表：
local obj = setmetatable({}, {_index = proto})
```

在克隆原型的时候还可以在 `setmetatable` 的第一个参数里添加新字段，使之成为不同于原型的新对象，通常这个方法的名字是 new：

```lua
function proto.new()
  return setmetatable({name='pokemon'}, mt)
end
```



### self 参数

Lua 为面向对象的方式使用表内成员函数提供了一个特殊的操作符： `:`，它的功能与 `.` 基本相同，但是在调用函数时会隐含传入一个 `self` 参数。

self 类似 C++ / Java 里面的 this 或者 Python 里面的 Self，实际上就是对象自身，通过 self 就可以很方便地获取到对象内部成员。

`:` 和 `self` 不仅可以在调用函数时使用，也可以在定义函数时使用：

```lua
function proto:hello() 
  print("hello", self.name)
end

local obj = proto:new()
obj:hello()
```

`:` 其实是一种语法糖，是简化的 `.` 写法，相当于：

```lua
function proto.hello(self) 
  print("...")
end

obj.hello(obj)
```

这两种方法可以使用，没有本质区别，但是建议风格一致。

## 标准库

Lua 的库实际上就是包含了函数成员的表。

Lua 内置的标准库很小，只提供了基本的功能，主要有：

- base  ：最核心的函数
- package ：管理 Lua 的模块
- string ： 字符串相关
- table ：表相关
- math ：数学计算相关
- io ： 文件相关
- os  ： 操作系统相关
- debug ： 跳使用函数

### base 库常用的函数

- `assert(s)`  : 如果执行结果为 nil 或者 false 则报错，否则返回执行结果。
- `error(msg)` ： 直接引发一个错误。
- `collectgarbage(opt)` ：Lua 垃圾货收器的操作函数。opt 的值有很多，常用的 collect 要求立即运行垃圾回收，count 检查 Lua 的内存使用情况（KB）。
- `loadstring(string)` ：加载一个字符串形式的 Lua 代码片段，返回包含着短代码的函数：





---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





