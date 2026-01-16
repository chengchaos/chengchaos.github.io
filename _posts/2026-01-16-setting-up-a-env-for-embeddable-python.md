---
title: Embeddable Python 开发环境搭建
key: 2026-01-16
tags: Windows Python Embeddable
---

嵌入式Python（Embeddable Python）是一种精简版的 Python，适用于无需安装完整 Python 环境的场景，如便携式开发或分发 Python 程序。以下是搭建嵌入式 Python 开发环境的步骤。

## 0x01. 下载嵌入式 Python

前往 Python 官网。

在 Downloads 页面选择适合的 Windows 版本，下载带有“embeddable”字样的压缩包。 注意：建议选择与现有 Python 版本一致的版本，避免兼容性问题。

## 0x02. 解压并配置环境

将下载的压缩包解压到目标目录，例如：`D:\python-embedded`。

修改解压目录中的 `pythonXY._pth` 文件： 取消对 `import site` 行的注释，以启用模块加载功能。

## 0x03. 安装 pip 工具

嵌入式 Python 默认不包含 `pip`，需要手动安装：

下载 get-pip.py 文件：

[get-pip](https://bootstrap.pypa.io/get-pip.py)

将文件保存到嵌入式 Python 目录中。

在命令行中运行以下命令安装 `pip`：

```sh
python get-pip.py
```

## 0x04. 安装第三方库

使用 pip 为嵌入式 Python 安装所需的第三方库：

```sh
D:\python-embedded\python.exe -m pip install 包名

## 例如安装 numpy：
D:\python-embedded\python.exe -m pip install numpy

```

## 0x05. 测试和使用

创建一个测试脚本（如test.py）：

```python
print("Hello from Embeddable Python!")
```

使用嵌入式Python运行脚本：

```sh
D:\python-embedded\python.exe test.py
```


## appendex 最佳实践

便携性：将整个嵌入式Python目录打包到U盘，可随时随地使用。

轻量化：仅安装必要的库，避免占用过多空间。

自动化启动：通过 `.bat` 文件简化启动流程。

通过以上步骤，您可以快速搭建一个高效、便携的嵌入式 Python 开发环境！