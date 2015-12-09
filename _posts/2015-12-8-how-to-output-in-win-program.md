---
layout: post
title: 如何在windows窗口程序调试时输出内容到命令行
date: 2015/12/8 15:58:25 
category: c++
published: true
---

说明：在windows下的vs环境中开发窗口应用程序，没有比较有效的输出内容的方式，在程序的调试和开发过程中，实时的显示程序中需要输出的内容，对程序的开发和调试有非常大的帮助。此种方法的结果是在显示窗口的同时，显示一个可以输出内容的命令行窗口，可以在此窗口中输出任务你想输出的内容，只需要调用打印命令即可，如printf/wprintf

### 示例环境 ###
vs2010,win7

### 代码函数  ###

    void InitConsoleWindow()
    {
    	int nCrt;
    	FILE* fp;
    	AllocConsole();
    	nCrt = _open_osfhandle(reinterpret_cast<long>(GetStdHandle(STD_OUTPUT_HANDLE)), _O_TEXT);
    	fp = _fdopen(nCrt, "w");
    	*stdout = *fp;
    	setvbuf(stdout, nullptr, _IONBF, 0);
    }
### 使用方法 ###
在程序启动的地方调用以上方法即可，需要输出的内容可以通过printf/wprintf进行输出；

#### 结束 ####