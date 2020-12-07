---
title: golang 字符串处理
key: 2020-12-07
tags: go golang string
---

字符串是golang的基本类型之一，也是常用的值类型。go的标准库有两个对字符串相关操作包值得利用。

<!--more-->

## springs

strings包主要实现了利用简便的函数来操作UTF-8编码的字符串。

### 比较和包含

s1 中释放包含 s2

```go
fmt.Println(strings.Contains("Hello の right", "の")) // true

// 判断 s 是否包含 chars 中的任意一个字符？
// 有一个就算 true 
strings.ContainsAny(s, chars string) bool

// 判断在 Unicode 大小写不敏感的情况下，s 和 t 是否相等。
strings.EqualFold(s, t string) bool

// 判断 s 是否又 prefix 的前缀
string.HasPrefix(s, prefix string) bool

// 是否又后缀
string.HasSuffix(s, suffix string) bool
```

### 位置和数量

```go
// 字符串 s 中包含结几个 sep 子串？
func Count(s, sep string) int

fmt.Println(strings.Count("街角魔族是最好看的动漫", "街角魔族"))  // 1
fmt.Println(strings.Count("up sail の right", "君の名"))      // 0

// 子串 sep 在 字符串 s 中的第一次出现的位置，
// 不存在返回 -1， 下标从 0 开始
func Index(s, sep string) int

// 还有几个扩展
func IndexByte(s string, b byte) int
func IndexRune(s string, r rune) int
func IndexAnhy(s , chars string) int
// s 中第一个满足函数 f 的位置
// 该处的 utf-8 码值 r 满足 f(r) == true
func IndexFunc(s string, f func(rune) bool) int 

// 从后面查……
func LastIndex(s, sep string) int

func LastIndexByte(s string, c byte) int

func LastIndexAny(s, char string) int

func LastIndexFunc(s string, f func(rune) bool) int


//这里编码由于是unicode因此一个中文算3位
fmt.Println(strings.LastIndexFunc("街角魔族是世界第1好看动漫",
    //出现字符'好'的位置
    func(r rune) bool {
        if r == '好' {
            return true
        }
        return false
    }))
fmt.Println(strings.LastIndexFunc(" right!", func(r rune) bool {
    return !unicode.IsSpace(r) && !unicode.IsNumber(r) //最后一个不是空格且不是数字的位置
}))
fmt.Println(strings.LastIndexFunc("123 ChopinB", unicode.IsLower)) //最后一个是小写字母的
    
```

### 替换

```go
// 将 s 内的所有单词首字母转换为大写
strings.Title("Hello boy you very cool!")

// 将所有字母转换为小写
strings.ToLower(s string) string

// 使用规定的字符映射c，将s内的所有字母转换为小写
strings.ToLowerSpecial(c unicode.SpecialCase, s string) string
// 使用规定的字符映射c，将s内的所有字母转换为大写
strings.ToTitleSpecial(c unicode.SpecialCase, s string) string
strings.ToUpperSpecial(c unicode.SpecialCase, s string) string
// 将素有字母转大写 和 ToUpper 一样
strings.ToTitle(s string) string
strings.ToUpper(s string) string

// 将 s 内不属于 utf-8 范围的字符串替换为 replacement 子串，
// 如果不存在非法字符串则无事发生。
strings.ToValidUTF8(s, replacement string) string
n
// 使用 new 替换字符串 s 中的 old 子串， n 为替换个数
// 如果 old 为空，则在字符串的开头和每个 UTF-8 序列之后匹配
// 如果 n < 0，则替换数没有限制。
strings.Replace(s, old, new string, n int)

// 等于 Replace(s, old, new, -1)
strings.ReplaceAll(s, old, new string) string

// 对应的结构体 Replacer
strings.NewReplacer(oldnew ...string) *Replacer
r := strings.NewReplacer("好", "最好", "&lt;", "<", "&rt;", ">")    //一新一旧交替更换
fmt.Println(r.Replace("&lt;h1&rt;街角魔族是好看的动漫&lt;/h1&rt;"))

// 根据 mapping 函数修改字符串的所有字符
// 如果 mapping 返回一个负值，则从字符串中删除。
fmt.Println(strings.Map(func(r rune) rune {
    switch {
        case r == '不': 
        return -1
        case r == '特':
        return '最'
        case r == ' ':
        return '看'
    }
    return r
}, "角魔族是特不好 的动漫"))
```

### 分割

```go

```

转自：

- [https://blog.csdn.net/BangBrother/article/details/106833621](https://blog.csdn.net/BangBrother/article/details/106833621)

EOF

---

Power by TeXt.
