# thrift-python

thrift_file 目录执行：$ thrift -out .. --gen py example.thrift，就会在 thrift_file 的同级目录下生成 python 的包：example

##Transport

Transport网络读写（socket，http等）抽象，用于和其他thrift组件解耦。

Transport的接口包括：open, close, read, write, flush, isOpen, readAll。

Server端需要ServerTransport（对监听socket的一种抽象），用于接收客户端连接，接口包括：listen, accept, close。

python中Transport的实现包括：TSocket, THttpServer, TSSLSocket, TTwisted, TZlibTransport，都是对某种协议或框架的实现。还有两个装饰器，用于为已有的Transport添加功能，TBufferedTransport（增加缓冲）和TFramedTransport（添加帧）。

在创建server时，传入的时Tranport的工厂，这些Factory包括：TTransportFactoryBase（没有任何修饰，直接返回），TBufferedTransportFactory（返回带缓冲的Transport）和TFramedTransportFactory（返回帧定位的Transport）。

##Protocol
Protocol用于对数据格式抽象，在rpc调用时序列化请求和响应。

TProtocol的实现包括：TJSONProtocol，TSimpleJSONProtocol，TBinaryProtocol，TBinaryPotocolAccelerated，TCompactProtocol。

##Processor
Processor对stream读写抽象，最终会调用用户编写的handler已响应对应的service。具体的Processor有compiler生成，用户需要实现service的实现类。

##Server

Server创建Transport，输入、输出的Protocol，以及响应service的handler，监听到client的请求然后委托给processor处理。

TServer是基类，构造函数的参数包括：

1) processor, serverTransport
2) processor, serverTransport, transportFactory, protocolFactory
3) processor, serverTransport, inputTransportFactory, outputTransportFactory, inputProtocolFactory, outputProtocolFactory 

TServer内部实际上需要3）所列的参数，1）和2）会导致对应的参数使用默认值。

TServer的子类包括：TSimpleServer, TThreadedServer, TThreadPoolServer, TForkingServer, THttpServer, TNonblockingServer, TProcessPoolServer

TServer的serve方法用于开始服务，接收client的请求。

## Code generated
constants.py: 包含声明的所有常量

ttypes.py: 声明的struct，实现了具体的序列化和反序列化

SERVICE_NAME.py: 对应service的描述文件，包含了：

Iface: service接口定义

Client: client的rpc调用桩

Thrift的用法实际上很简单，定义好IDL，然后实现service对应的handler（方法名、参数列表与接口定义一致接口），最后就是选择各个组件。

需要选择的包括：Transport（一般都是socket，只是十分需要选择buffed和framed装饰器factory），Protocol，Server。

# 传输协议 

在传输协议上总体划分为文本和二进制 ,为节约带宽，提高传输效率，一般情况下使用二进制类型的传输协议为多数.

TBinaryProtocol — 二进制编码格式进行数据传输

TCompactProtocol — 高效率的、密集的二进制编码格式进行数据传输

TJSONProtocol — 使用 JSON 的数据编码协议进行数据传输

TSimpleJSONProtocol — 只提供 JSON 只写的协议，适用于通过脚本语言解析

TDebugProtocol – 使用易懂的可读的文本格式，以便于 debug

## 数据传输

TSocket — 使用阻塞式 I/O 进行传输，是最常见的模式

TFramedTransport — 使用非阻塞方式，按块的大小进行传输

TNonblockingTransport — 使用非阻塞方式，用于构建异步客户端

TMemoryTransport – 将内存用于 I/O

TZlibTransport – 使用 zlib 进行压缩， 与其他传输方式联合使用

TFileTransport – 以文件形式进行传输

## 服务端类型

TSimpleServer — 单线程服务器端使用标准的阻塞式 I/O

TThreadPoolServer —— 多线程服务器端使用标准的阻塞式 I/O

TNonblockingServer —— 多线程服务器端使用非阻塞式 I/O

## 数据类型 

Thrift 脚本可定义的数据类型包括以下几种类型：

基本类型：
bool：布尔值，true 或 false

byte：8 位有符号整数

i16：16 位有符号整数

i32：32 位有符号整数

i64：64 位有符号整数

double：64 位浮点数

string：未知编码文本或二进制字符串

结构体类型：

struct：定义公共的对象，类似于 C 语言中的结构体定义

容器类型：

list：一系列 t1 类型的元素组成的有序表，元素可以重复

set：一系列 t1 类型的元素组成的无序表，元素唯一

map<t1,t2>：key/value 对（key 的类型是 t1 且 key 唯一，value 类型是 t2）

异常类型：

exception 异常在语法和功能上类似于结构体，它在语义上不同于结构体—当定义一个 RPC 服务时，开发者可能需要声明一个远程方法抛出一个异常。

服务类型：

service：对应服务的类
