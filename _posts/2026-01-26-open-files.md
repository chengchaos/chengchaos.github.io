---
title: 用程序打开一个文件
key: 2026-01-26
tags: Python open-file
---

如何读写一个文件呢?

<!--more-->

## 0x01 基本用法

Reading files in Python is a common I/O operation, and the built-in open() function provides a simple way to handle it. You can read text or binary files, process them line-by-line, or load the entire content at once depending on your needs.

### Reading a Text File

#### Step 1 – Open the file Use open() with mode 'r' for reading text:

```python
with open('example.txt', 'r', encoding='utf-8') as f:
    content = f.read()
    print(content)
```

This reads the entire file into memory as a string. The with statement ensures the file is closed automatically.

#### Step 2 – Read line-by-line For large files, read incrementally to save memory:

```python
with open('example.txt', 'r', encoding='utf-8') as f:
    for line in f:
    print(line.strip())
```

Alternatively, use `readline()` for manual control or `readlines()` to get a list of all lines.

### Reading a Binary File

When working with images, videos, or other non-text data, use 'rb' mode:

```python
with open('image.jpg', 'rb') as f:
    data = f.read()
    print(data[:20]) # Print first 20 bytes
```

Binary mode prevents Python from decoding bytes into text.

Handling Encodings and Errors

If the file uses a different encoding (e.g., GBK), specify it:

```python
with open('data_gbk.txt', 'r', encoding='gbk', errors='ignore') as f:
    print(f.read())
```

The `errors='ignore'` option skips invalid characters.

Best Practices

- Always use `with open(...)` to ensure proper resource cleanup.

- For huge files, prefer iterating line-by-line instead of loading all content at once.

- Specify encoding explicitly when dealing with non-UTF-8 files to avoid UnicodeDecodeError.

- Use binary mode for non-text data to prevent corruption.

By combining these techniques, you can efficiently and safely read any type of file in Python while managing memory and handling errors gracefully.



## Appendix

[使用asyncio](https://liaoxuefeng.com/books/python/async-io/asyncio/index.html)