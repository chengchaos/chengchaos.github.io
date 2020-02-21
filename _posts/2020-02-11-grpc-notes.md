---
title: gRPC 学习笔记[0001]
key: 2020-02-09
tags: java gRPC
---

As The Titile

## 简介

gRPC 是 google 开源实现的一个 RPC 框架,以支持多语言和使用移动场景。

gRCP 为了实现对多语言的支持，采用了一种叫做 Protobuf 的服务中立语言来定义服务接口。

Protobuf 是一种序列化机制，也是一种中立接口。Protobuf 自成体系，不与任何特定编程语言绑定。这这个体系中，它拥有对各种语言特性的描述。使用 Protobuff 定义接口后，只需要使用 Protobuf 对应各语言的编译器，反向生成编程语言，就能实现跨语言特性。


<!--more-->

## Protubuf

Protobuf 全程 protocol buffers，是 google 开发的一种序列化结构。

Protobuf 是一种灵活、高效、自动化的序列化机制，与语言无关，平台无关，用于通信协议，数据存储等。

目前（2020年），Protobuf 已经发展到第 3 版，又成为 proto3，相对 proto2 添加了一些新功能，简化了 protocol buffer 语言。

使用 Protobuf，第一部要创建 .proto 文件，在其中定义 protocol buffer 消息类型。

例如：

```proto3

syntax = "proto3";

option java_package = "com.java.grpc.geral";
option java_outer_classname = "Messages";

message Messages {
    int32 id = 1;
    int64 code = 2;
    string texts = 3;
}

```

以上使用 protobuf 定义了一条 Messages 消息。

版本信息 proto3 必须写在 .proto 文件第一行。

Protobuf 有自己的数据类型，比如 int32、int63、string 等。也就是前面说的 Protobuf 对各种语言特性的描述。Protobuf 对应不同的语言都有响应的类型转换规则，通过 Protobuf 各语言的编译器生成目标语言的代码，自动完成类型转换。

Java 的例子：

| .protobuf | java |
| ---- | ---- |
| int32 | init |
| int64 | long |
| bool | boolean |
| string | String |
| bytes | ByteString |


格式定义:

```
    <数据类型> <名称> <编号>
```

其中编号是用于标识序列化位置， Protobuf 是一种二进制序列化机制，可以指定可选字段、必填字段和重复字段。



## 定义请求、响应协议

使用 gRPC 时，首先要定义一个通用的请求（Request）、响应（Response）协议。

### 简单请求响应协议

一个例子：

```proto3

syntax = "proto3"

option java_package = "com.java.grpc.geral";
option java_outer_classname = "Request";

message RPCRequest {
    int64 id = 1;
    int32 number = 2;
    int32 limit = 3;
    string extras = 4;
    string tokens = 5;
}

```

响应：

```proto3

syntax = "proto3";

option java_package = "com.java.grpc.geral";
option java_outer_classname = "Response";

message RPCResponse {
    int32 code = 1;
    int32 total = 2;
    int32 pages = 3;
    string mesg = 4;
}
```

## gRPC 的发送与数据解析

定义一个 RPC 服务接口:

```
syntax = "proto3";

option java_package = "com.java.grpc.geral.service";
option java_outer_classname = "EchoService";

import "RPCRequest.proto";
import "RPCResponse.proto";

service RPCServiceEcho {

    rpc echo(RPCRequest) returns (RPCResponse);
    
}


```





<< EOF >>


If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
