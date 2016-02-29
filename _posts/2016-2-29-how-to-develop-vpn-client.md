---
layout: post
title: windows c++ 开发系列5 vpn客户端的开发
date: 2016/2/29 17:17:02    
category: c++
published: true
---

在windows 下开发常用协议的vpn客户端，还是比较方便的，window已经提供了相应的api接口函数，只要恰当调用，就可以实现自己的vpn客户端；

废话不多说，直接上干货
实现一个客户端的连接过程，分以下几步
## 1、创建入口 ##
    bool createEntry(const wchar_t* rasConn, const wchar_t* serverIP, const wchar_t* userName, const wchar_t* psk, int type)
    {
    DWORD size = 0;
    RasGetEntryProperties(NULL, L"", NULL, &size, NULL, NULL);
    LPRASENTRY pras = (LPRASENTRY)malloc(size);
    memset(pras, 0, size);
    pras->dwSize = size;
    pras->dwType = RASET_Vpn;
    pras->dwRedialCount = m_iNumRetry;// 重连次数
    pras->dwRedialPause = m_iIntervalTime;//重连间隔
    pras->dwfNetProtocols = RASNP_Ip;
    pras->dwEncryptionType = ET_Optional;
    pras->dwIdleDisconnectSeconds = 10;// set idle timeout 10 seconds
    wcscpy_s(pras->szLocalPhoneNumber, serverIP);
    wcscpy_s(pras->szDeviceType, RASDT_Vpn);
    pras->dwfOptions = RASEO_RemoteDefaultGateway;
    
    #if (WINVER >= 0x601)
    pras->dwNetworkOutageTime = 10;
    #endif
    
    if (pptp == type)
    {
    pras->dwfOptions |= RASEO_RequireEncryptedPw;
    pras->dwVpnStrategy = VS_PptpOnly;
    }
    else if (l2tp_psk == type)
    {
    pras->dwVpnStrategy = VS_L2tpOnly;
    pras->dwfOptions |= RASEO_RequireEncryptedPw;
    pras->dwfOptions2 |= RASEO2_UsePreSharedKey;
    }
    else if (l2tp_cert == type)
    {
    pras->dwVpnStrategy = (VS_L2tpOnly);
    //pras->dwfOptions2 |= RASEO2_DisableIKENameEkuCheck;
    }
    else if (ikev2_eap == type)
    {
    pras->dwfOptions |= (RASEO_RequireDataEncryption | RASEO_RequireEAP | RASEO_RequireMsCHAP2);
    //pras->dwVpnStrategy = VS_Ikev2Only;
    }
    else if (ikev2_cert == type)
    {
    pras->dwfOptions |= RASEO_RequireDataEncryption;
    //pras->dwfOptions2 |= RASEO2_RequireMachineCertificates;
    //pras->dwVpnStrategy = VS_Ikev2Only;
    }
    
    //RasSetEntryProperties(L"f://test.pbk", name, pras, pras->dwSize, NULL, 0);
    RasSetEntryProperties(NULL, rasConn, pras, pras->dwSize, NULL, 0);
    RASCREDENTIALS ras_cre = { 0 };
    ras_cre.dwSize = sizeof(ras_cre);
    ras_cre.dwMask = RASCM_UserName | RASCM_Password;
    wcscpy_s(ras_cre.szUserName, userName);
    wcscpy_s(ras_cre.szPassword, L"");
    RasSetCredentials(NULL, rasConn, &ras_cre, FALSE);
    
    if (l2tp_psk == type)
    {
    RASCREDENTIALS ras_cre_psk = { 0 };
    ras_cre_psk.dwSize = sizeof(ras_cre_psk);
    ras_cre_psk.dwMask = RASCM_PreSharedKey;
    /*wcscpy_s(ras_cre_psk.szUserName, username);
    wcscpy_s(ras_cre_psk.szPassword, password);*/
    wcscpy_s(ras_cre_psk.szPassword, psk);
    RasSetCredentials(NULL, rasConn, &ras_cre_psk, FALSE);
    }
    
    free(pras);
    
    return true;
    }

## 2、连接 ##
    bool connectVpn(int& errRet, const wchar_t* rasConn, const wchar_t* userHub, const wchar_t* password)
    {
    RASDIALPARAMSRasDialParams;
    //HRASCONNhRasConn;
    DWORDRet;
    
    ZeroMemory(&RasDialParams, sizeof(RASDIALPARAMS));
    RasDialParams.dwSize = sizeof(RASDIALPARAMS);
    lstrcpy(RasDialParams.szEntryName, rasConn);
    lstrcpy(RasDialParams.szUserName, userHub);
    lstrcpy(RasDialParams.szPassword, password);
    
    m_hRasConn = NULL;
    Ret = RasDial(NULL, NULL,/*L"f://test.pbk",*/ &RasDialParams, 0, &RasDialFunc, &m_hRasConn);
    
    if (Ret != 0)
    {
    LOG(WARNING) << "RasDial return: " << Ret;
    
    errRet = Ret;
    RasDeleteEntry(NULL, RasDialParams.szEntryName);
    return false;
    }
    else
    {
    /*RasHangUp(m_hRasConn);
    RasDeleteEntry(NULL,name);*/
    }
    
    // redial timeout
    SetTimer(g_pMainFrame->GetHWND(),300,10000,NULL);
    
    return true;
    }

## 3、断开连接 ##
    bool disconnect()
    {
    DWORD ret;
    ret = RasHangUp(m_hRasConn);
    m_hRasConn = NULL;
    
    if (0 != ret)
    {
    return false;
    }
    
    return true;
    }
## 4、删除入口 ##
    bool VpnDial::deleteVpnEntry(const wchar_t* rasConn )
    {
    DWORD ret;
    
    ret = RasDeleteEntry(NULL, rasConn);
    
    if (0 != ret)
    {
    return false;
    }
    
    return true;
    }
    
通过以上几步，就已经实现了一个普通的vpn的客户端；

但若要实现更完善的功能，如连接过程的监控，连接成功后，连接状态的监控，连接超时的设置等；
请继续往下看
连接过程的监控
vpn连接时，可设置连接的回调函数


    Ret = RasDial(NULL, NULL,/*L"f://test.pbk",*/ &RasDialParams, 0, &RasDialFunc, &m_hRasConn);

## 回调函数的实现 ##
    void WINAPI RasDialFunc(UINT unMsg,
    RASCONNSTATE rasconnstate,
    DWORD dwError )
    {
    char szRasString[256] = {0}; // Buffer for storing the error string
    TCHAR szTempBuf[256] = {0};  // Buffer used for printing out the text
    
    if (dwError)  // Error occurred
    {
    RasGetErrorString(static_cast<UINT>(dwError), reinterpret_cast<LPWSTR>(szRasString), CELEMS(szRasString));
    
    ZeroMemory(static_cast<LPVOID>(szTempBuf), sizeof(szTempBuf));
    StringCchPrintf(szTempBuf, CELEMS(szTempBuf), _T("Error: %d - %s\n"), dwError, szRasString);
    printf(reinterpret_cast<const char*>(szTempBuf));
    
    //SetEvent(m_ehDisconnect);
    g_pMainFrame->disconnect();
    return;
    }
    
    // Map each of the states of RasDial() and display on the screen
    // the next state that RasDial() is entering
    switch (rasconnstate)
    {
    case RASCS_OpenPort:
    LOG(INFO) << "RASCS_OpenPort = " << rasconnstate;
    LOG(INFO) << "Opening port...";
    g_pMainFrame->setLinkStateInfo("正在打开端口",OPENPORT);
    //g_pFrame->setUserInfo("test","test","test","test","test");
    break;
    case RASCS_PortOpened:
    LOG(INFO) << "RASCS_PortOpened = " << rasconnstate;
    LOG(INFO) << "Port opened.";
    g_pMainFrame->setLinkStateInfo("端口打开",PORTOPENED);
    break;
    case RASCS_ConnectDevice:
    LOG(INFO) << "RASCS_ConnectDevice = " << rasconnstate;
    LOG(INFO) << "Connecting device...";
    g_pMainFrame->setLinkStateInfo("连接设备",CONNECTDEVICE);
    break;
    case RASCS_DeviceConnected:
    LOG(INFO) << "RASCS_DeviceConnected = " << rasconnstate;
    LOG(INFO) << "Device connected.";
    break;
    case RASCS_AllDevicesConnected:
    LOG(INFO) << "RASCS_AllDevicesConnected = " << rasconnstate;
    LOG(INFO) << "All devices connected.";
    g_pMainFrame->setLinkStateInfo("设备已连接",ALLDEVICESCONNECTED);
    break;
    case RASCS_Authenticate:
    LOG(INFO) << "RASCS_Authenticate = " << rasconnstate;
    LOG(INFO) << "Authenticating...";
    g_pMainFrame->setLinkStateInfo("开始认证",AUTHENTICATE);
    break;
    case RASCS_AuthNotify:
    LOG(INFO) << "RASCS_AuthNotify = " << rasconnstate;
    LOG(INFO) << "Authentication notify.";
    break;
    case RASCS_AuthRetry:
    LOG(INFO) << "RASCS_AuthRetry = \n" << rasconnstate;
    LOG(INFO) << "Retrying authentication...";
    break;
    case RASCS_AuthCallback:
    LOG(INFO) << "RASCS_AuthCallback = " << rasconnstate;
    LOG(INFO) << "Authentication callback...";
    break;
    case RASCS_AuthChangePassword:
    LOG(INFO) << "RASCS_AuthChangePassword = " << rasconnstate;
    LOG(INFO) << "Change password...";
    break;
    case RASCS_AuthProject:
    LOG(INFO) << "RASCS_AuthProject = " << rasconnstate;
    LOG(INFO) << "Projection phase started...";
    break;
    case RASCS_AuthLinkSpeed:
    LOG(INFO) << "RASCS_AuthLinkSpeed = " << rasconnstate;
    LOG(INFO) << "Negoting speed...";
    break;
    case RASCS_AuthAck:
    LOG(INFO) << "RASCS_AuthAck = " << rasconnstate;
    LOG(INFO) << "Authentication acknowledge...";
    break;
    case RASCS_ReAuthenticate:
    LOG(INFO) << "RASCS_ReAuthenticate = " << rasconnstate;
    LOG(INFO) << "Retrying Authentication...";
    break;
    case RASCS_Authenticated:
    LOG(INFO) << "RASCS_Authenticated = " << rasconnstate;
    LOG(INFO) << "Authentication complete.";
    g_pMainFrame->setLinkStateInfo("认证通过",AUTHENTICATED);
    break;
    case RASCS_PrepareForCallback:
    LOG(INFO) << "RASCS_PrepareForCallback = " << rasconnstate;
    LOG(INFO) << "Preparing for callback...";
    break;
    case RASCS_WaitForModemReset:
    LOG(INFO) << "RASCS_WaitForModemReset = " << rasconnstate;
    LOG(INFO) << "Waiting for modem reset...";
    break;
    case RASCS_WaitForCallback:
    LOG(INFO) << "RASCS_WaitForCallback = " << rasconnstate;
    LOG(INFO) << "Waiting for callback...";
    break;
    case RASCS_Projected:
    LOG(INFO) << "RASCS_Projected = " << rasconnstate;
    LOG(INFO) << "Projection completed.";
    break;
    #if (WINVER >= 0x400)
    case RASCS_StartAuthentication:// Windows 95 only
    LOG(INFO) << "RASCS_StartAuthentication = " << rasconnstate;
    LOG(INFO) << "Starting authentication...";
    
    break;
    case RASCS_CallbackComplete:   // Windows 95 only
    LOG(INFO) << "RASCS_CallbackComplete = " << rasconnstate;
    LOG(INFO) << "Callback complete.";
    break;
    case RASCS_LogonNetwork:   // Windows 95 only
    LOG(INFO) << "RASCS_LogonNetwork = " << rasconnstate;
    LOG(INFO) << "Login to the network.";
    break;
    #endif
    case RASCS_SubEntryConnected:
    LOG(INFO) << "RASCS_SubEntryConnected = " << rasconnstate;
    LOG(INFO) << "Subentry connected.";
    break;
    case RASCS_SubEntryDisconnected:
    LOG(INFO) << "RASCS_SubEntryDisconnected = " << rasconnstate;
    LOG(INFO) << "Subentry disconnected.";
    break;
    //PAUSED STATES:
    case RASCS_Interactive:
    LOG(INFO) << "RASCS_Interactive = " << rasconnstate;
    LOG(INFO) << "In Paused state: Interactive mode.";
    break;
    case RASCS_RetryAuthentication:
    LOG(INFO) << "RASCS_RetryAuthentication = " << rasconnstate;
    LOG(INFO) << "In Paused state: Retry Authentication...";
    break;
    case RASCS_CallbackSetByCaller:
    LOG(INFO) << "RASCS_CallbackSetByCaller = " << rasconnstate;
    LOG(INFO) << "In Paused state: Callback set by Caller.";
    break;
    case RASCS_PasswordExpired:
    LOG(INFO) << "RASCS_PasswordExpired = " << rasconnstate;
    LOG(INFO) << "In Paused state: Password has expired...";
    break;
    
    case RASCS_Connected: // = RASCS_DONE:
    LOG(INFO) << "RASCS_Connected = " << rasconnstate;
    LOG(INFO) << "#########Connection completed.";
    //SetEvent(gEvent_handle);
    g_pMainFrame->setLinkStateInfo("连接成功",CONNECTED);
    
    break;
    case RASCS_Disconnected:
    LOG(INFO) << "RASCS_Disconnected = " << rasconnstate;
    LOG(INFO) << "Disconnecting...";
    break;
    default:
    LOG(INFO) << "Unknown Status = " << rasconnstate;
    LOG(INFO) << "What are you going to do about it?";
    break;
    }
    }

## 连接状态的监控 ##
连接成功后，连接状态的监控;  
在连接成功后，设置连接断开事件的通知的事件

    if (m_ehDisconnect != nullptr)
    {
    CloseHandle(m_ehDisconnect);
    m_ehDisconnect = NULL;
    }
    m_ehDisconnect = CreateEvent(
    NULL,   // default security attributes
    TRUE,   // manual-reset event
    FALSE,  // initial state is nonsignaled
    TEXT("disconn")  // object name
    );    
    
    LOG_IF(WARNING,m_ehDisconnect==NULL) << "CreateEvent failed, Error: " << GetLastError();
    
    RasConnectionNotification(m_hRasConn, m_ehDisconnect, RASCN_Disconnection); 
    
## 连接超时的处理 ##
虽然，连接设置中有超时的参数设置，但在大多系统中，并不起作用；我们可以使用定时器进行连接超时的处理。
在拨号函数返回后，即开始进行拨号，设置定时器，如果在定时器期间没有拨号成功，则触发定时器，定时器断开正在拨号的连接。
如果拨号成功，在拨号成功的回调状态里，取消定时器；

通过以上的处理，一个完成的vpn连接客户端的核心代码就完成了；加上界面就可以很方便的使用了！

代码文件 


完！