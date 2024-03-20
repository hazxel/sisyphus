Protobuf 是 Google 开发的一款用于数据序列化的语言中立协议，可以用于在不同编程语言中高效的传输和储存结构化数据。



### C++

生成代码：`protoc --cpp-out=. my_proto.proto` ，生成 `my_proto.pb.h` 和 `my_proto.pb.cc` 

编译：`g++ -o my_program my_program.cc my_proto.pb.cc -lprotobuf`
