---
title: pip / uv 使用清华源
key: 2026-05-16
tags: pip uv python 
---

## 0x10 pip

The Tsinghua University Open Source Mirror provides a fast and reliable alternative for Python package installations in regions with slow access to the default PyPI repository. Below are the steps to use the Tsinghua mirror with pip.

### 0x11. Temporary Use

To install a package using the Tsinghua mirror temporarily, use the `-i` option:

```sh
pip install -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple <package-name>
```

Replace `<package-name>` with the desired package.

### 0x12. Set as Default Source

To configure the Tsinghua mirror as the default source for all pip operations:
Upgrade pip to the latest version:

```sh
python -m pip install --upgrade pip
```

Set the global index URL:

```sh
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

### 0x13. Upgrade pip Using Tsinghua Mirror

If your connection to the default PyPI is slow, you can upgrade pip using the Tsinghua mirror:

```sh
python -m pip install -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple --upgrade pip
```

### 0x14. Configure Multiple Mirrors

To balance load or add fallback mirrors, configure additional sources:

```sh
pip config set global.extra-index-url "<url1> <url2>"
```

For example:

```sh
pip config set global.extra-index-url "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
```

### 0x15 Best Practices

- Always ensure HTTPS is used for secure connections.
- Use temporary configurations (-i) for one-off installations to avoid altering global settings unnecessarily.

- Regularly update pip and verify mirror reliability.

By following these steps, you can significantly improve your Python package installation experience in regions with limited access to PyPI.

## UV

<https://zhuanlan.zhihu.com/p/2025230187210515041>

### 0x21 方法一：使用配置文件 (推荐，通杀所有模式)

通过配置 uv 的原生配置文件，无论是 uv pip 还是 uv add 都能直接生效。这也是目前最推荐的做法。

#### 1. 全局配置 (uv.toml)

如果你希望这台电脑上所有的 uv 操作默认都使用国内源，请修改全局 uv.toml。

在 Linux / macOS 下：
配置文件路径为：`~/.config/uv/uv.toml`
打开终端，执行以下命令直接写入配置：

```sh
mkdir -p ~/.config/uv
cat <<EOF > ~/.config/uv/uv.toml
[[index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
EOF
```

在 Windows 下：
配置文件路径为：`%APPDATA%\uv\uv.toml` (通常是 `C:\Users\你的用户名\AppData\Roaming\uv\uv.toml`)

你可以打开 PowerShell 执行以下命令：

```sh
$uvConfigDir = "$env:APPDATA\uv"
New-Item -ItemType Directory -Force -Path $uvConfigDir
$configContent = @"
[[index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
"@
Set-Content -Path "$uvConfigDir\uv.toml" -Value $configContent
```

(注意：default = true 表示将该源作为默认源，替换官方的 PyPI。)