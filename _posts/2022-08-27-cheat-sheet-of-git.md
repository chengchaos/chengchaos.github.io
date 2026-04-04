---

title: Cheat sheet of git
key: 2021-08-27
tags: git
---

Get **[ERROR] Invalid syntax in configuration ini file.**

So you are trying to commit to git and getting [ERROR] Invalid syntax in configuration ini file. Right? Well this is generally caused by hooks installed in your git directory. You may check in .git/hooks
and see which one is throwing this error.

To resolve this error, you may commit using -n
or --no-verify flag.

```bash
$ git commit -m 'update before 2022-08-27'
[ERROR] Invalid syntax in configuration ini file.
[ERROR] Invalid syntax in configuration ini file.
$ git commit -m "commit without verify" -n
```

<!--more-->

EOF
