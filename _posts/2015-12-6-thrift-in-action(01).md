---
layout: post
title: thrift实用教程(1.1 基本介绍)
date: "2015/12/6 15:29:18"
category: c++
published: true
---


# 1 前言 
Thrift是一个跨语言的服务部署框架，最初由Facebook于2007年开发，2008年进入Apache开源项目。Thrift通过一个中间语言(IDL, 接口定义语言)来定义RPC的接口和数据类型，然后通过一个编译器生成不同语言的代码（目前支持C++,Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk和OCaml）,并由生成的代码负责RPC协议层和传输层的实现。  
本文组织结构如下：  
- 1）前言 
- 2）架构
- 3）支持的数据传输格式、数据传输方式和服务模型 
- 4）Thrift安装 
- 5）利用Thift部署服务

# 2 架构 
![](http://i.imgur.com/aIAk26B.jpg)

Thrift实际上是实现了C/S模式，通过代码生成工具将接口定义文件生成服务器端和客户端代码（可以为不同语言），从而实现服务端和客户端跨语言的支持。用户在Thirft描述文件中声明自己的服务，这些服务经过编译后会生成相应语言的代码文件，然后用户实现服务（客户端调用服务，服务器端提服务）便可以了。其中protocol（协议层, 定义数据传输格式，可以为二进制或者XML等）和transport（传输层，定义数据传输方式，可以为TCP/IP传输，内存共享或者文件共享等）被用作运行。

# 3 支持的数据传输格式、数据传输方式和服务模型   

## 3.1 支持的传输格式   

- TBinaryProtocol – 二进制格式.
- TCompactProtocol – 压缩格式
- TJSONProtocol – JSON格式
- TSimpleJSONProtocol –提供JSON只写协议, 生成的文件很容易通过脚本语言解析。
- TDebugProtocol – 使用易懂的可读的文本格式，以便于debug  

## 3.2 支持的数据传输方式   
>>>>>>> 1b23cc47c92ad308cf3d3232f6a7b44c6810eec0
- TSocket -阻塞式socker
- TFramedTransport – 以frame为单位进行传输，非阻塞式服务中使用。
- TFileTransport – 以文件形式进行传输。
- TMemoryTransport – 将内存用于I/O. java实现时内部实际使用了简单的ByteArrayOutputStream。
- TZlibTransport – 使用zlib进行压缩， 与其他传输方式联合使用。当前无java实现。  

## 3.3 支持的服务模型   
>>>>>>> 1b23cc47c92ad308cf3d3232f6a7b44c6810eec0
- TSimpleServer – 简单的单线程服务模型，常用于测试
- TThreadPoolServer – 多线程服务模型，使用标准的阻塞式IO。
- TNonblockingServer – 多线程服务模型，使用非阻塞式IO（需使用TFramedTransport数据传输方式）

# 4 Thrift安装   
- 下载：[http://thrift.apache.org/download](http://thrift.apache.org/download)
- 安装要求：
- Unix/linux 系统，windows+cygwin
- C++语言：g++、boost
- java 语言：JDK、Apache Ant
- 其他语言：Python、PHP、Perl, etc…
- 编译安装：./configure –》make –》make install

# 5 利用Thrift部署服务   
主要流程：编写服务说明，保存到.thrift文件–》根据需要， 编译.thrift文件，生成相应的语言源代码–》根据实际需要， 编写client端和server端代码。  
.thrift文件编写  
一般将服务放到一个.thrift文件中，服务的编写语法与C语言语法基本一致，在.thrift文件中有主要有以下几个内容：变量声明、数据声明（struct）和服务接口声明（service, 可以继承其他接口）。  
下面分析Thrift的tutorial中带的例子tutorial.thrift  
> 包含头文件：  
> 59行：include “shared.thrift”  
> –  
> 指定目标语言：  
> 65行：namespace cpp tutorial  
> –  
> 定义变量：  
> 80行：const i32 INT32CONSTANT = 9853  
> –  
> 定义结构体：  
> 103行：struct Work {  
> 1: i32 num1 = 0,  
> 2: i32 num2,  
> 3: Operation op,  
> 4: optional string comment,  
> }  
> —  
> 定义服务：  
> service Calculator extends shared.SharedService {  
> void ping(),  
> i32 add(1:i32 num1, 2:i32 num2),  
> i32 calculate(1:i32 logid, 2:Work w) throws (1:InvalidOperation ouch),  
> oneway void zip()  
> }    

要生成c++代码：./thrift -r --gen cpp tutorial.thrift,代码存放在gen-cpp目录下  
要生成java代码:./thrift -r --gen java tutorial.thrift,代码存放在gen-java目录下

client端和server端代码编写  
client端和sever端代码要调用编译.thrift生成的中间文件。

下面分析cpp文件下面的CppClient.cpp和CppServer.cpp代码    

![](http://i.imgur.com/nzm2zjA.jpg)

在client端，用户自定义CalculatorClient类型的对象（用户在.thrift文件中声明的服务名称是Calculator， 则生成的中间代码中的主类为CalculatorClient）， 该对象中封装了各种服务，可以直接调用（如client.ping()）, 然后thrift会通过封装的rpc调用server端同名的函数。  

在server端，需要实现在.thrift文件中声明的服务中的所有功能，以便处理client发过来的请求。
