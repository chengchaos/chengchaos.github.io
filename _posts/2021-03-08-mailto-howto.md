---
title: 如何使用 mailto
key: 2021-03-08
tags: html mailto
---

mailto 怎样写。

<!--more-->

基本用法：

```html
<a href='mailto:sample@163.com'>Send Mail</a>

<form action='mailto:sample@163.com'>
    
</form>

```

mailto 后面跟的是收信人。

可用的参数列表：

- to 收信人
- subject 主题
- cc 抄送
- bcc  暗送
- body 内容

可以用查询字符串，也可以用 form

```html
<a href='mailto:sample@163.com?subject=test&cc=sample@hotmail.com&body=user maiot sample'>Send Mail</a>

<form name="sendmail" action="mailto:sample@163.com">
    <input name="cc" type="text" value="sample@hotmail.com" />
    <input name="subject" type="text" value="test" />
    <input name="body" type="text" value="use mailto sample" />
</form>
```

EOF

---

Power by TeXt.
