---
layout: post
title: 详解thrift的异步模式
date: 2015/12/8 16:14:14  
category: c++
published: true
---

## 前言 ##
Thrift采用的是C/S模型，通过编写标准的接口，再通过编译程序，可以很容易的完成标准的接口调用;  

- 客户端，调用接口函数，得到函数返回结果；
- 服务端，对相应的接口调用进行处理，返回相应的结果给客户端；

根据对返回结果的处理方式，接口函数分为三种
 
- 接口调用，等待返回，返回处理，由同一个函数完成
- 接口调用，不需要返回结果
- 接口调用，直接返回，返回结果由另一个函数进行接收并处理

前两种方式很容易理解，通过thrift生成的代码，很容易实现；  
本文主要讨论第三种方式，即异步的函数调用(asynchronous invoke).
编程语言使用c++,IDE为vs2010,操作系统为win7

## 接口函数的编写 ##
与同步接口函数文件的编写相同；

    /* @file:asyncTest.thrift */
    
    namespace cpp thrift.example
    
    struct twitterReturnStruct{
    1:string strMethod,
    2:string strSend,
    3:string strGet
    }
    
    service Twitter{
    
    twitterReturnStruct sendLongString(1:string data);
    twitterReturnStruct sendString(1:string data);
    bool testBool(1:string data);
    }

## 代码生成 ##

> thrift -r --gen cpp:cob_style .\test.thrift

### 生成的文件 ###

![](https://raw.githubusercontent.com/curious-boy/myfiles/master/pic/mysite/asyncThrift/async_thrift_gen_files.png)

红色框中为定义的数据类型或结构体；  
蓝色框中为服务接口类；  
其它两个文件为同步和异步的示例服务端的代码；

## 创建服务端程序 ##
### 创建空项目 ###
### 引入数据类型文件和接口类文件 ###
### 创建主程序文件 ###
如main.cpp  
在Twitter_async_server.skeleton.cpp文件中生成了服务函数处理类TWitterAsyncHandler，直接使用，将函数的处理逻辑改成自己需要的，这一点和同步模式是一致的。  
### main函数的编写 ###

    int main(int argc, char **argv) {
		// 初始化网络，这个是我后来加的，
		// 由于自动生成的代码编译的时候出现错误
    	WORD wVersionRequested;
    	WSADATA wsaData;
    	int err;
    	wVersionRequested = MAKEWORD(2,1);
    	err = WSAStartup(wVersionRequested,&wsaData);
    	if (err != 0)
    	{
    		return -1;
    	}
    
		// 创建服务对象，监听并调用TwitterAsyncHandler处
		// 理客户端的请求
    	shared_ptr<TAsyncProcessor> underlying_pro(new TwitterAsyncProcessor( shared_ptr<TwitterCobSvIf>(new TwitterAsyncHandler()) ) );
    	shared_ptr<TAsyncBufferProcessor> processor( new TAsyncProtocolProcessor( underlying_pro, shared_ptr<TProtocolFactory>(new TBinaryProtocolFactory()) ) );
    
    	TEvhttpServer server(processor, 9090);
    
    	printf("server is listening...\n");
    	server.serve();
    	return 0;
    }
### 服务程序头文件的引用目录 ###
![](https://raw.githubusercontent.com/curious-boy/myfiles/master/pic/mysite/asyncThrift/async_thrift_include_server.png)
### 库文件的引用目录 ###
![](https://raw.githubusercontent.com/curious-boy/myfiles/master/pic/mysite/asyncThrift/async_thrift_lib_server.png)

### 引用的库文件 ###

- wsock32.lib
- libevent.lib
- libthrift.lib
- libthriftnb.lib


## 创建客户端程序 ##

### 创建空项目 ###
### 引入数据类型文件和接口类文件 ###
### 创建主程序文件 ###
如client.cpp 
 
客户端对象  

    class testClient : public TwitterCobClient
    {
    public:
    	testClient(boost::shared_ptr< ::apache::thrift::async::TAsyncChannel> channel, TProtocolFactory* protocolFactory)
    		: TwitterCobClient(channel, protocolFactory)
    	{
    		bRet = false;
    	};
    
    	//返回的结果处理
    	virtual void completed__(bool success)
    	{
    		if (success)
    		{
    			if (stuRes.strMethod == "sendString")
    			{
    			}
    			printf("respone : %s \n", stuRes.strMethod.c_str());   // 输出返回结果
    			printf("send: %s \tGet: %s \n",stuRes.strSend.c_str(),stuRes.strGet.c_str());
    
    		}
    		else
    		{
    			printf("failed to respone\n");
    		}
    		fflush(0);
    	};
    
    	thrift::example::twitterReturnStruct stuRes;
    };

返回结果处理的回调函数  

    // callback function 
    static void my_recv_sendString(TwitterCobClient* client)
    {
    	client->recv_sendString(dynamic_cast<testClient *>(client)->stuRes);
    };
    
    static void my_recv_sendLongString(TwitterCobClient* client)
    {
    	client->recv_sendLongString(dynamic_cast<testClient *>(client)->stuRes);
    };
    
    // 元类型的返回结果，除string类型外，需要在这个函数中进行返回结果的处理
    static void my_recv_testBool(TwitterCobClient* client)
    {
    	bool bRet = client->recv_testBool();
    	
    	if (bRet)
    	{
    		printf("handle something after recv_testBool\n");
    	}
    }

接口的调用与执行

    static void sendString(testClient& client)   
    {
    	boost::function<void(TwitterCobClient* client)> cob = boost::bind(&my_recv_sendString, _1);
    	client.sendString(cob, "hello");   // 发送并注册回调函数
    	printf("sendString end\n");
    }
    
    static void sendLongString(testClient& client)   
    {
    	boost::function<void(TwitterCobClient* client)> cob = boost::bind(&my_recv_sendLongString, _1);
    	client.sendLongString(cob,"lhello");
    	printf("sendString end\n");
    }
    
    static void testBool(testClient& client)
    {
    	boost::function<void(TwitterCobClient* client)> cob = boost::bind(&my_recv_testBool, _1);
    	client.testBool(cob, "");   // 发送并注册回调函数
    	printf("testBool end\n");
    }
    
    static void DoSimpleTest(const std::string& host, int port)
    {
    	printf( "running DoSimpleTest( %s, %d) ...\n",host.c_str(), port);
    	event_base* base = event_base_new();
    	boost::shared_ptr< ::apache::thrift::async::TAsyncChannel>  channel1( new TEvhttpClientChannel( host, "/", host.c_str(), port, base  ) );
    
    	testClient client1( channel1,  new TBinaryProtocolFactory() );
    	sendLongString(client1);   // 发送第一个请求
    	printf("sendLongString(client1) was sended!\n");
    	boost::shared_ptr< ::apache::thrift::async::TAsyncChannel>  channel2( new TEvhttpClientChannel( host, "/", host.c_str(), port, base  ) );
    	testClient client2( channel2,  new TBinaryProtocolFactory() );
    	sendString(client2);  // 发送第二个请求
    	printf("sendString(client2) was sended!\n");
    
    	boost::shared_ptr< ::apache::thrift::async::TAsyncChannel>  channel3( new TEvhttpClientChannel( host, "/", host.c_str(), port, base  ) );
    	testClient client3( channel3,  new TBinaryProtocolFactory() );
    	testBool(client3);  // 发送第二个请求
    	printf("testBool(client3) was sended!\n");
    
    	event_base_dispatch(base);
    	event_base_free(base);
    	printf( "done DoSimpleTest().\n" );
    }
    
    // 初始化socket
    bool initSocket()
    {
    	WORD wVersionRequested;
    	WSADATA wsaData;
    	int err;
    	wVersionRequested = MAKEWORD(2,1);
    	err = WSAStartup(wVersionRequested,&wsaData);
    	if (err != 0)
    	{
    		return false;
    	}
    	return true;
    }

### 主函数的编写 ###


    int main( int argc, char* argv[] )
    {
    	initSocket();
    	DoSimpleTest( "192.168.2.179", 9090 );
    	return 0;
    }

### 服务程序头文件引用目录 ###

![](https://raw.githubusercontent.com/curious-boy/myfiles/master/pic/mysite/asyncThrift/async_thrift_include_client.png)

### 库文件引用目录 ###
![](https://raw.githubusercontent.com/curious-boy/myfiles/master/pic/mysite/asyncThrift/async_thrift_lib_client.png)
### 库文件引用 ###

- libthrift.lib
- libthriftnb.lib
- libevent.lib
- libevent_core.lib
- libevent_extras.lib

[示例程序](https://github.com/curious-boy/asyncThrift.git)  
注：如果在编译过程中出现库冲突的问题，可能需要在忽略特定库；如msvcprtd.lib;msvcrtd.lib；