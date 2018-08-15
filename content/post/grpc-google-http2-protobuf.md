---
title: "gRPC：Google开源的基于HTTP/2和ProtoBuf的通用RPC框架"
date: 2018-08-15T11:16:15+08:00
draft: false
menu: "main"
Categories: ["Golang", "gRPC", "Guide"]
Description: "gRPC是一个高性能、通用的开源RPC框架，该框架已广泛应用于Google的云产品和谷歌的对外提供的API服务中。gRPC由Google主要面向移动应用开发并基于HTTP/2协议标准而设计，基于ProtoBuf(Protocol Buffers)序列化协议开发，且支持众多开发语言。"
Tags: ["gRPC","guide","solutions","golang","RPC框架","开源"]
---

[gRPC][^grpc]是一个高性能、通用的开源RPC框架，其由Google主要面向移动应用开发并基于[HTTP/2][^http2]协议标准而设计，基于[ProtoBuf][^protobuf](Protocol Buffers)序列化协议开发，且支持众多开发语言。gRPC提供了一种简单的方法来精确地定义服务和为iOS、Android和后台支持服务自动生成可靠性很强的客户端功能库。客户端充分利用高级流和链接功能，从而有助于节省带宽、降低的TCP链接次数、节省CPU使用、和电池寿命。

gRPC具有以下重要特征：

* 强大的IDL特性
  
  gRPC使用ProtoBuf来定义服务，ProtoBuf是由Google开发的一种数据序列化协议（类似于XML、JSON、hessian）。ProtoBuf能够将数据进行序列化，并广泛应用在数据存储、通信协议等方面。不过，当前gRPC仅支持 Protobuf ，且不支持在浏览器中使用。由于gRPC的设计能够支持支持多种数据格式，所以读者能够很容易实现对其他数据格式（如XML、JSON等）的支持。

  定义服务的示例代码如下：

  ```
  message HelloRequest {
    string greeting = 1;
  }
  message HelloResponse {
    string reply = 1;
  }
  service HelloService {
    rpc SayHello(HelloRequest) returns (HelloResponse);
  }
  ```

* 支持多语言

  gRPC支持多种语言，并能够基于语言自动生成客户端和服务端功能库。目前，在GitHub上已提供了 C版本`grpc`、Java版本`grpc-java` 和 Go版本`grpc-go`，其它语言的版本正在积极开发中，其中 grpc支持 `C`、`C++`、`Node.js`、`Python`、`Ruby`、`Objective-C`、`PHP` 和 `C#` 等语言，`grpc-java` 已经支持Android开发。

* 基于HTTP/2标准设计
  
  由于gRPC基于HTTP/2标准设计，所以相对于其他RPC框架，gRPC带来了更多强大功能，如双向流、头部压缩、多复用请求等。这些功能给移动设备带来重大益处，如节省带宽、降低TCP链接次数、节省CPU使用和延长电池寿命等。同时，gRPC还能够提高了云端服务和Web应用的性能。gRPC既能够在客户端应用，也能够在服务器端应用，从而以透明的方式实现客户端和服务器端的通信和简化通信系统的构建。

gRPC已经应用在Google的云服务和对外提供的API中，其主要应用场景如下：

* 低延迟、高扩展性、分布式的系统
* 同云服务器进行通信的移动应用客户端
* 设计语言独立、高效、精确的新协议
* 便于各方面扩展的分层设计，如认证、负载均衡、日志记录、监控等

近日，gRPC开发团队宣布gRPC基于 [BSD 3-Clause License][^bsd3] 许可协议开源，相关代码已托管在[GitHub][^grpc]上。当前已有Google和移动支付公司[Square][^Square]以及其他组织或个人为该项目贡献代码。有兴趣的读者可以在GitHub选择需要的语言版本，并根据提供的README文档尝试gRPC的功能，或者参考FAQ，以获得对gRPC更多信息。此外，在[gRPC-common][^gRPC-common]仓库中，还提供了例子、快速入门指南等相关文档。

[^bsd3]: http://opensource.org/licenses/BSD-3-Clause/  "BSD 3-Clause License"
[^grpc]: https://github.com/grpc/  "gRPC"
[^Square]: https://squareup.com/  "Square"
[^gRPC-common]: https://github.com/grpc/grpc-common
[^http2]: https://http2.github.io/
[^protobuf]: http://en.wikipedia.org/wiki/Protocol_Buffers
