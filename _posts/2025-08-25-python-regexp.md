---
title: Python 正则表达式
key: 2025-08-25
tags: python regexp re
---

> Search suggest: Python re regexp

Python 自1.5版本起增加了re 模块，它提供 Perl 风格的正则表达式模式。

re 模块使 Python 语言拥有全部的正则表达式功能。

compile 函数根据一个模式字符串和可选的标志参数生成一个正则表达式对象。该对象拥有一系列方法用于正则表达式匹配和替换。

re 模块也提供了与这些方法功能完全一致的函数，这些函数使用一个模式字符串做为它们的第一个参数。


<!--more-->

## 0x01 `re.match` 函数

`re.match` 尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match() 就返回 none。

函数语法：

```python
re.match(pattern, string, flags=0)
# pattern: 匹配的正则表达式
# string: 要匹配的字符串
# flags: 标志位

```



