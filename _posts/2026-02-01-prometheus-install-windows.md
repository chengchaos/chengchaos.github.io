---
title: Windows 中安装 Prometheus 和其他
key: 2026-02-01
tags: Windows Prometheus
---

Windows 中安装 Prometheus 也是可以的. 

<!--more-->

## 1. Prometheus 基础介绍

Prometheus 是一个功能强大的开源监控和警报工具包，由 SoundCloud 公司于 2012 年创建，后来成为云原生计算基金会（CNCF）的项目之一。Prometheus 的设计理念是提供一个高效且易于查询的多维数据模型，并配备了一套强大的查询语言 PromQL（Prometheus Query Language），使得用户能够方便地检索时间序列数据。

Prometheus 的架构设计为单体架构，以微服务模式运行。它由多个组件组成，其中包括：

- Prometheus Server： 负责收集和存储时间序列数据。
- Exporters： 用于暴露不同服务的监控数据，例如HTTP接口、数据库、云服务等。
- Pushgateway： 用于短周期作业的数据收集。
- Alertmanager： 处理警报并发送通知。
- Client Libraries： 用于集成Prometheus监控到应用程序。 

Prometheus 通过拉取（Pulling）方式工作，即由 Prometheus Server 定时从配置好的 exporters 端点获取数据。此外，它也支持通过 Pushgateway 接收客户端推送的数据。

为了展示 Prometheus 的工作原理，我们以监控一个简单的 HTTP 服务器为例。首先，服务器需要配置一个 exporter 来暴露监控数据。然后，Prometheus Server 会定期从该 exporter 收集数据并存储在本地时序数据库中。当需要检索数据时，用户可以通过 PromQL 编写查询语句，在 Prometheus UI 中进行查询，或者通过 API 来获取数据。

通过简单的配置和使用，Prometheus 不仅可以监控基础设施的状态，还能跟踪应用的性能指标，是现代 IT 环境监控工具箱中不可或缺的一部分。随着对监控数据的深入分析和警报机制的运用，系统管理员和 DevOps 工程师能够更加有效地管理服务的可用性和性能。

## 2. Prometheus 在 Windows 上的下载与安装指南 

### 2.1 下载Prometheus和相关组件

#### 2.1.1 选择合适的版本进行下载

下载 Prometheus 对于新手来说可能会有些许困惑，因为 Prometheu s的开发者提供了不同版本供用户选择，包括预编译的二进制文件、Docker 镜像和源码包。对于 Windows 平台，推荐下载预编译的二进制文件以简化安装过程。

首先，访问Prometheus的官方GitHub发布页面 [prometheus](https://github.com/prometheus/prometheus/releases)。

在 GitHub 页面上，你会看到不同版本的标签。在这里，选择与你的 Windows 系统架构（32位或64位）相匹配的最新稳定版本。例如，如果你的 Windows系统是 64 位的，那么你应该寻找类似 prometheus-2.x.x.windows-amd64.zip 的文件。

下载完成后，解压该ZIP文件。解压后，你将得到一个包含 Prometheus 可执行文件（ prometheus.exe ）和默认配置文件的目录。

#### 2.1.2 下载与安装 wmi_exporter

wmi_exporter 是一个适用于 Windows 系统的 Prometheus exporter，它能够收集系统级别的指标数据，比如 CPU、内存和磁盘使用情况。它需要以特定权限运行，因此安装和配置也需要特别注意。

前往 wmi_exporter 的GitHub页面（https://github.com/prometheus-community/windows_exporter）下载预编译的Windows版本。选择与你的系统架构相匹配的版本进行下载。

下载完成并解压后，你需要按照以下步骤进行安装：

- 将 wmi_exporter.exe 放置在你希望它在的目录下，比如 `C:\Program Files\wmi_exporter` 。
- 添加一个新的系统环境变量，名称为 `WMI_EXPORTER_PATH` ，值为你存放 `wmi_exporter.exe` 的完整路径。 

### 2.2 安装 Prometheus 服务器

#### 2.2.1 解压缩安装包

对于已经下载并解压的 Prometheus 二进制文件，安装过程非常直接。在你的 Prometheus 解压缩目录中，你会找到 `prometheus.exe` 。如果需要，可以通过右键发送到桌面快捷方式以便快速访问。

#### 2.2.2 设置服务启动方式

为了方便运行和管理 Prometheus 服务，建议将其设置为 Windows 服务。这可以通过第三方工具或编写一个简单的批处理脚本来完成。以下是通过一个批处理文件 `install_service.bat` 来设置 Prometheus 为 Windows 服务的示例代码：

```sh
@echo off
setlocal
set PROM_VERSION=2.25.0
set PROM_DIR=C:\Program Files\prometheus
set PROM_EXE=%PROM_DIR%\prometheus-%PROM_VERSION%.windows-amd64\prometheus.exe
set PROM_SERVICE=prometheus-%PROM_VERSION%

sc.exe create %PROM_SERVICE% binPath= "%PROM_EXE% --config.file=%PROM_DIR%\prometheus.yml" start= auto

if %ERRORLEVEL% NEQ 0 (
    echo Failed to install Prometheus as a service. Exit code: %ERRORLEVEL%
    exit /b %ERRORLEVEL%
)
echo Prometheus service installed successfully.

endlocal
```

## 0x01 Prometheus

### 下载

### 安装

### 配置

### 其他

## 0x02 AlertManater

### 下载

### 安装

### 配置

### 其他


## 0x03 Grafana

### 下载

### 安装

### 配置

### 其他

## Appendix

- [Prometheus在Windows上的安装与服务器监控实战](https://blog.csdn.net/weixin_42584507/article/details/149590143)