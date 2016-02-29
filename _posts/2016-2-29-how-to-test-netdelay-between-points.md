---
layout: post
title: windows c++ 开发系列6 vpn批量测速的实现
date: 2016/2/29 17:27:19     
category: c++
published: true
---

## 概念 ##
测速，这里特指机器间的网络延迟的长短

一般情况下，可直接使用ping命令测试与指定IP机器间的网络延迟；在软件中，也可以使用ping的实现，对指定IP的机器进行ping测速；
ping命令的实现代码如下

ping.h
    
    #pragma once
    
    #include <WinSock2.h>
    #include <windows.h>
    
    
    
    //这里需要导入库 Ws2_32.lib，在不同的IDE下可能不太一样 
    //#pragma comment(lib, "Ws2_32.lib")
    
    #define DEF_PACKET_SIZE 32
    #define ECHO_REQUEST 8
    #define ECHO_REPLY 0
    
    struct IPHeader
    {
    	BYTE m_byVerHLen; //4位版本+4位首部长度
    	BYTE m_byTOS; //服务类型
    	USHORT m_usTotalLen; //总长度
    	USHORT m_usID; //标识
    	USHORT m_usFlagFragOffset; //3位标志+13位片偏移
    	BYTE m_byTTL; //TTL
    	BYTE m_byProtocol; //协议
    	USHORT m_usHChecksum; //首部检验和
    	ULONG m_ulSrcIP; //源IP地址
    	ULONG m_ulDestIP; //目的IP地址
    };
    
    struct ICMPHeader
    { 
    	BYTE m_byType; //类型
    	BYTE m_byCode; //代码
    	USHORT m_usChecksum; //检验和 
    	USHORT m_usID; //标识符
    	USHORT m_usSeq; //序号
    	ULONG m_ulTimeStamp; //时间戳（非标准ICMP头部）
    };
    
    struct PingReply
    {
    	USHORT m_usSeq;
    	DWORD m_dwRoundTripTime;
    	DWORD m_dwBytes;
    	DWORD m_dwTTL;
    };
    
    class CPing
    {
    public:
    	CPing();
    	~CPing();
    	BOOL Ping(DWORD dwDestIP, PingReply *pPingReply = NULL, DWORD dwTimeout = 1000);
    	BOOL Ping(char *szDestIP, PingReply *pPingReply = NULL, DWORD dwTimeout = 1000);
    private:
    	BOOL PingCore(DWORD dwDestIP, PingReply *pPingReply, DWORD dwTimeout);
    	USHORT CalCheckSum(USHORT *pBuffer, int nSize);
    	ULONG GetTickCountCalibrate();
    
    private:
    	SOCKET	m_sockRaw;
    	WSAEVENT	m_event;
    	USHORT	m_usCurrentProcID;
    	char	*m_szICMPData;
    	BOOL	m_bIsInitSucc;
    private:
    	static USHORT s_usPacketSeq;
    };
    
ping.cpp
    
    #include "Ping.h"
    #include <stdio.h>
    
    USHORT CPing::s_usPacketSeq = 0;
    
    CPing::CPing() : 
    m_szICMPData(NULL), 
    	m_bIsInitSucc(FALSE)
    {
    	WSADATA WSAData;
    	WSAStartup(MAKEWORD(1, 1), &WSAData);
    	
    	m_event = WSACreateEvent();
    	m_usCurrentProcID = (USHORT)GetCurrentProcessId();
    
    	if ((m_sockRaw = WSASocket(AF_INET, SOCK_RAW, IPPROTO_ICMP, NULL, 0, 0)) != SOCKET_ERROR)
    	{
    		WSAEventSelect(m_sockRaw, m_event, FD_READ);
    		m_bIsInitSucc = TRUE;
    
    		m_szICMPData = (char*)malloc(DEF_PACKET_SIZE + sizeof(ICMPHeader));
    
    		if (m_szICMPData == NULL)
    		{
    			m_bIsInitSucc = FALSE;
    		}
    	}
    	else if (m_sockRaw == INVALID_SOCKET) 
    	{
    		printf("WSASocket() failed: %d\n", WSAGetLastError());
    	}
    }
    
    CPing::~CPing()
    {
    	WSACleanup();
    
    	if (NULL != m_szICMPData)
    	{
    		free(m_szICMPData);
    		m_szICMPData = NULL;
    	}
    }
    
    BOOL CPing::Ping(DWORD dwDestIP, PingReply *pPingReply, DWORD dwTimeout)
    {  
    	return PingCore(dwDestIP, pPingReply, dwTimeout);
    }
    
    BOOL CPing::Ping(char *szDestIP, PingReply *pPingReply, DWORD dwTimeout)
    {  
    	if (NULL != szDestIP)
    	{
    		return PingCore(inet_addr(szDestIP), pPingReply, dwTimeout);
    	}
    	return FALSE;
    }
    
    BOOL CPing::PingCore(DWORD dwDestIP, PingReply *pPingReply, DWORD dwTimeout)
    {
    	//判断初始化是否成功
    	if (!m_bIsInitSucc)
    	{
    		return FALSE;
    	}
    
    	//配置SOCKET
    	sockaddr_in sockaddrDest; 
    	sockaddrDest.sin_family = AF_INET; 
    	sockaddrDest.sin_addr.s_addr = dwDestIP;
    	int nSockaddrDestSize = sizeof(sockaddrDest);
    
    	//构建ICMP包
    	int nICMPDataSize = DEF_PACKET_SIZE + sizeof(ICMPHeader);
    	ULONG ulSendTimestamp = GetTickCountCalibrate();
    	USHORT usSeq = ++s_usPacketSeq;
    	memset(m_szICMPData, 0, nICMPDataSize);
    	ICMPHeader *pICMPHeader = (ICMPHeader*)m_szICMPData;
    	pICMPHeader->m_byType = ECHO_REQUEST; 
    	pICMPHeader->m_byCode = 0; 
    	pICMPHeader->m_usID = m_usCurrentProcID;
    	pICMPHeader->m_usSeq = usSeq;
    	pICMPHeader->m_ulTimeStamp = ulSendTimestamp;
    	pICMPHeader->m_usChecksum = CalCheckSum((USHORT*)m_szICMPData, nICMPDataSize);
    
    	//发送ICMP报文
    	if (sendto(m_sockRaw, m_szICMPData, nICMPDataSize, 0, (struct sockaddr*)&sockaddrDest, nSockaddrDestSize) == SOCKET_ERROR)
    	{
    		return FALSE;
    	}
    
    	//判断是否需要接收相应报文
    	if (pPingReply == NULL)
    	{
    		return TRUE;
    	}
    
    	char recvbuf[256] = {"\0"};
    	while (TRUE)
    	{
    		//接收响应报文
    		if (WSAWaitForMultipleEvents(1, &m_event, FALSE, 100, FALSE) != WSA_WAIT_TIMEOUT)
    		{
    			WSANETWORKEVENTS netEvent;
    			WSAEnumNetworkEvents(m_sockRaw, m_event, &netEvent);
    			
    			if (netEvent.lNetworkEvents & FD_READ)
    			{
    				ULONG nRecvTimestamp = GetTickCountCalibrate();
    				int nPacketSize = recvfrom(m_sockRaw, recvbuf, 256, 0, (struct sockaddr*)&sockaddrDest, &nSockaddrDestSize);
    				if (nPacketSize != SOCKET_ERROR)
    				{
    					IPHeader *pIPHeader = (IPHeader*)recvbuf;
    					USHORT usIPHeaderLen = (USHORT)((pIPHeader->m_byVerHLen & 0x0f) * 4);
    					ICMPHeader *pICMPHeader = (ICMPHeader*)(recvbuf + usIPHeaderLen);
    
    					if (pICMPHeader->m_usID == m_usCurrentProcID //是当前进程发出的报文
    						&& pICMPHeader->m_byType == ECHO_REPLY //是ICMP响应报文
    						&& pICMPHeader->m_usSeq == usSeq //是本次请求报文的响应报文
    						)
    					{
    						pPingReply->m_usSeq = usSeq;
    						pPingReply->m_dwRoundTripTime = nRecvTimestamp - pICMPHeader->m_ulTimeStamp;
    						pPingReply->m_dwBytes = nPacketSize - usIPHeaderLen - sizeof(ICMPHeader);
    						pPingReply->m_dwTTL = pIPHeader->m_byTTL;
    						return TRUE;
    					}
    				}
    			}
    		}
    		//超时
    		if (GetTickCountCalibrate() - ulSendTimestamp >= dwTimeout)
    		{
    			return FALSE;
    		}
    	}
    }
    
    USHORT CPing::CalCheckSum(USHORT *pBuffer, int nSize)
    {
    	unsigned long ulCheckSum=0; 
    	while(nSize > 1) 
    	{ 
    		ulCheckSum += *pBuffer++; 
    		nSize -= sizeof(USHORT); 
    	}
    	if(nSize ) 
    	{ 
    		ulCheckSum += *(UCHAR*)pBuffer; 
    	} 
    
    	ulCheckSum = (ulCheckSum >> 16) + (ulCheckSum & 0xffff); 
    	ulCheckSum += (ulCheckSum >>16); 
    
    	return (USHORT)(~ulCheckSum); 
    }
    
    ULONG CPing::GetTickCountCalibrate()
    {
    	static ULONG s_ulFirstCallTick = 0;
    	static LONGLONG s_ullFirstCallTickMS = 0;
    
    	SYSTEMTIME systemtime;
    	FILETIME filetime;
    	GetLocalTime(&systemtime);
    	SystemTimeToFileTime(&systemtime, &filetime);
    	LARGE_INTEGER liCurrentTime;
    	liCurrentTime.HighPart = filetime.dwHighDateTime;
    	liCurrentTime.LowPart = filetime.dwLowDateTime;
    	LONGLONG llCurrentTimeMS = liCurrentTime.QuadPart / 10000;
    
    	if (s_ulFirstCallTick == 0)
    	{
    		s_ulFirstCallTick = GetTickCount();
    	}
    	if (s_ullFirstCallTickMS == 0)
    	{
    		s_ullFirstCallTickMS = llCurrentTimeMS;
    	}
    
    	return s_ulFirstCallTick + (ULONG)(llCurrentTimeMS - s_ullFirstCallTickMS);
    }
    

在windows下，代码中ping操作是需要管理员权限的；如果在程序中使用了以上的代码进行测速操作，需要在程序运行的时候赋予程序以管理员权限，否则会执行失败。

在测试与大量机器间的网络延迟，一个个的测下去，尽管每次的测试只需要很少的时间，全部测下来，也需要不少的时间；ping中可设置超时，如设置超时的时间为1s，如果测试100台机器有3台机器出现了超时的情况，其它的机器的平均测速的时间为30ms，那么总的运行时间为>1000*3+30*(100-3) ,大约6s的时间；如果有较多的机器超时，或平均的测速结果为60ms时；总之时间可能会变的比较长；

如何才能较快的给批量的机器进行测速呢？

使用线程池进行处理；
### 流程是这样的 ###
- 首先，批量的机器的信息是存放到一个列表中；
- 第二，创建一个线程组，里面的线程个数，根据情况，自己设定；
- 第三，每个线程的处理逻辑是，检查需要测速的列表，看是否有需要测速的记录，如果有，则取出一条记录，进行测速，同时把此记录标记为测速完毕；如果没有，线程返回；
- 第四，线程在一条记录测速完成后，逻辑回到第三步；
- 第五，创建线程组后，等待线程组的返回；

具体线程组的使用，可参考同系统的其它文章；

完！