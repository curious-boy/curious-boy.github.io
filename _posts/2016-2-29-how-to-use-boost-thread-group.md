---
layout: post
title: windows c++ 开发系列1 boost线程组的使用
date: 2016/2/29 16:59:17  
category: c++
published: true
---

## 概念 ##
thread_group是线程组的意思，可以实现对多个线程的统一管理；

### 成员函数 ###
thread *  create_thread( const  boost::function0<void>& );  // 创建一个线程
void  add_thread(thread * );  // 加入一个已存在的线程 
void  remove_thread(thread * );  // 移除一个线程
void  join_all();  // 全部等待结束

### 示例代码 ###
    #include <boost/thread/thread.hpp>
    #include <boost/bind.hpp>
    #include <boost/thread/mutex.hpp>
    #include <iostream>
    
    using namespace boost;
    using namespace std;
    
    mutex iomutex;
    
    void runChild(const int n)
    {
    {
    mutex::scoped_lock lock(iomutex);
    cout << "我是第" << n << "个子线程" << endl;
    }
    {
    mutex::scoped_lock lock(iomutex);
    cout << "进程" << n << "退出" << endl;
     }
    }
    
    int main(int argc, char** argv)
    {
       thread_group group;
       for(int num=0;num<10;num++)
       group.create_thread(bind(&runChild,num));
       group.join_all();
    }

mutex是互斥，bind是绑定函数和函数的参数，thread_group即线程组

### 真实案例 ###

    boost::thread_group threads;
    indexProcessed = 0;

    for (int i=0;i<MAXTHREADSIZE;i++)
    {
        threads.create_thread(boost::bind(&ThreadProc, this));
    }

    threads.join_all();

创建一个有MAXTHREADSIZE个的线程组，创建线程所使用的函数为ThreadProc，并把当前类实例的this指针作为参数传给线程函数；

线程组的在一些并行任务的处理上，有着非常大的优势；在有很多相同的任务需要执行的时候，可采用这种方式，充分利用多核等优势；

具体的案例，请参考 vpn批量测速的实现