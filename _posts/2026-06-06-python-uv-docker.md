---
title: Python UV Dockerfile 构建指
key: 2026-06-06
tags: Python uv docker dockerfile
---

在 Docker 中使用 uv 可以高效管理 Python 项目的依赖与环境，并通过多阶段构建优化镜像体积与构建速度。

<!--more-->

## 0x01 方法一：基于官方 uv 镜像

步骤：

选择基础镜像

可用 `ghcr.io/astral-sh/uv:python3.12-bookworm` 等镜像，已预装 uv 与对应 Python 版本。

复制项目文件

```Dockerfile
FROM ghcr.io/astral-sh/uv:python3.12-bookworm
ADD . /app
WORKDIR /app
RUN uv sync --locked
CMD ["uv", "run", "my_app"]
```

优化 在 .dockerignore 中添加 .venv，避免本地虚拟环境被打包进镜像。

方法二：基于 Python 官方镜像安装 uv

步骤：

安装 uv 二进制文件

FROM python:3.12-slim-bookworm
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
复制
安装依赖与项目

ADD . /app
WORKDIR /app
RUN uv sync --locked
CMD ["uv", "run", "my_app"]
复制
最佳实践 使用 --no-install-project 分离依赖安装层，加快构建。 指定 uv 版本或 SHA256 以保证可复现构建。

方法三：多阶段构建减少最终镜像体积

步骤：

构建阶段安装依赖

FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
WORKDIR /app
RUN uv sync --locked --no-editable
ADD . /app
RUN uv sync --locked --no-editable
复制
生产阶段仅复制虚拟环境

FROM python:3.12-slim
COPY --from=builder /app/.venv /app/.venv
CMD ["/app/.venv/bin/my_app"]
复制
✅ 验证方式：构建并运行容器，确保 uv run 能正常执行项目命令。 💡 提示：结合 docker compose watch 可实现热更新开发环境。

了解详细信息:
1 -
uv.doczh.com
2 -
juejin.cn

