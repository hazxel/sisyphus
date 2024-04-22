# RPC

RPC 框架的目标就是让调用者像调用本地接口一样调用远程服务，而不需要关心底层细节。 RPC 框架负责屏蔽底层传输方式（TCP/UDP）、序列化方式（XML/Json/Binary）、以及其他通信细节。



# Protobuf

Protobuf 是 Google 开发的一款用于数据序列化的语言中立协议，可以用于在不同编程语言中高效的传输和储存结构化数据。

- with schema, message is compact.

### C++

生成代码：`protoc --cpp-out=. my_proto.proto` ，生成 `my_proto.pb.h` 和 `my_proto.pb.cc` 

编译：`g++ -o my_program my_program.cc my_proto.pb.cc -lprotobuf`

### proto2 VS proto3

ProtoBuf 目前有两个版本，分别是 proto2 和 proto3，虽然 proto3 确实修正了 proto2 的很多问题和做了精简，但在默认值和未定义的字段等问题的处理上其实被诟病较多，但总体差别不大。

### gRPC

由 google 开发的一个基于 protobuf 的开源的高性能通用RPC框架。



# msgpack


MessagePack是一种有效的二进制序列化格式。它使您可以在多种语言之间交换数。它的格式与 JSON 类似，但是在存储时对数字、多字节字符、数组等都做了很多优化，减少了无用的字符。

- It’s like JSON, but fast and small. (from msgpack ßofficial site)
- no schema, need to **store the element name strings inside the message**. 



# thrift

thrift 是一种可伸缩的跨语言的高效 RPC 框架，由 facebook 贡献到 Apache 基金会。

- 消息定义文件支持注释
- 数据结构与传输表现分离
- ...



# Others

### bRPC

bRPC 是百度开源的使用 c++ 编写的工业级RPC框架，常用于搜索、存储、机器学习、广告、推荐等高性能系统。（据说代码质量十分优秀）

### flatbuffers, bebop...



# VS

the message size: JSON > msg pack > protobuf 

