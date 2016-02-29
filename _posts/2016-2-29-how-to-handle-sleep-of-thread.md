---
layout: post
title: windows c++ 开发系列3 windows下线程休眠的处理
date: 2016/2/29 17:06:46   
category: c++
published: true
---

## Sleep函数 ##
- 功能：执行挂起一段时间
- 用法： void Sleep(DWORD dwMillseconds);
- 在VC中使用带上头文件
- #include<windows.h>

在程序中，如果需要上线程或进程等一段时间，会使用sleep函数；
即使在线程中调用sleep函数，如果sleep的时间比较长，如大于2000（ ms）时，如果是界面程序，会发现整个程序处理挂起状态，线程挂起的时间内，整个程序对于用户的操作无反应；
经过测试验证，在线程中调用sleep 也会影响到整个程序的挂起；
事实上，在.net的线程api中，确实提供了线程的休眠函数，但对于c++开发者来说，只能另想办法。

在windows中，并没有提供其它的可选择的休眠函数，如果在界面程序的线程休眠期间，不影响到其它线程和主进程界面的运行呢？
方案 利用定时器
### 步骤1 ###
在线程需要休眠的地方，由原来的直接调用sleep 函数，改为设置定时器，时间与休眠函数的时间一致；然后，挂起线程；
### 代码 ###
    SetTimer(this->GetHWND(),400,10000,NULL);
    SuspendThread(hThreadPid);

### 步骤2 ###
编写定时器的处理，让挂起的线程重新运行，并取消定时器
代码
ResumeThread(hThreadPid);
KillTimer(this->GetHWND(),400);

### 步骤3 ###
添加定时器的消息处理

以上步骤，就实现了线程的休眠，同时也不会影响到其它的线程和主进程界面的运行；