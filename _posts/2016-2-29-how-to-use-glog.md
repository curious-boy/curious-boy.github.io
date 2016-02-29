---
layout: post
title: windows c++ 开发系列4 glog日志库的使用
date: 2016/2/29 17:10:04    
category: c++
published: true
---

glog是google开发并开源的c++的日志库，使用是比较方便的；

下面就讲一下如何使用的
首先，需要引入相应的头文件和库文件 
头文件 
#include "glog/logging.h"

### 库文件 ###
    pragma comment(lib, "libglog.lib")

### 初始化 ###
    google::InitGoogleLogging(wstring2string(GetExeName()).c_str());

初始化之前，可设置生成的日志文件的存放目录

    wstring wstrPath = GetExePath();
    wstrPath += _T("log\\");
    _tmkdir(wstrPath.c_str());
    wstring tempPath;
    tempPath = wstrPath + _T("log_info_");
    google::SetLogDestination(google::GLOG_INFO,wch2chr(tempPath.c_str()));
    tempPath = wstrPath + _T("log_waring_");
    google::SetLogDestination(google::GLOG_WARNING,wch2chr(tempPath.c_str()));
    tempPath = wstrPath + _T("log_error_");
    google::SetLogDestination(google::GLOG_ERROR,wch2chr(tempPath.c_str()));
    tempPath = wstrPath + _T("log_fatal_");
    google::SetLogDestination(google::GLOG_FATAL,wch2chr(tempPath.c_str()));

### 相关的设置 ###
    FLAGS_logbufsecs  = 0;// 日志实时输出
    FLAGS_max_log_size = 100;// 日志文件最大为100M
    FLAGS_stop_logging_if_full_disk = true; //磁盘满时，不再记录日志
    
    #ifdef _DEBUG//日志级别
    FLAGS_minloglevel=0;
    #else
    FLAGS_minloglevel=1;
    #endif // _DEBUG

日志一共有四个级别  
INFO WARNING ERROR FATAL

反初始化，或关闭日志的输出

    google::ShutdownGoogleLogging();

在代码中可以使用宏进行日志的输出

    LOG(INFO)<<"info log";
    LOG(WARNING)<<"warning log";
    LOG(ERROR)<<"error log";
    LOG(FATAL)<<"fatal log";

完！