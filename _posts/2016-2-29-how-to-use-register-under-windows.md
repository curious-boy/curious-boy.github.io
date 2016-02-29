
---
layout: post
title: windows c++ 开发系列2 windows 注册表的使用
date: 2016/2/29 16:10:23 
category: c++
published: true
---

## 概念 ##
注册表(Registry），是microsoft windows中的一个重要的数据库，用于存储系统和应用程序的设置信息；
是windows系统一个特有的一个概念。

### 打开注册表编辑器 ###
开始-->运行，输入regedit，确定。

### 注册表的写入与读取 ###
#### 相关工具函数 ####
- //返回基项
- HKEY GetBaseRegKey(char* keystr);
- //删除键值
- int RegDelValue(char* baseKey,char* subKey,char* subKeyValueName);
- //读取字符串类型的键值
- int RegGetStringValue(char* baseKey,char* subKey,char* subKeyValueName,string& value);
- //写入字符串类型的键值
- int RegSetStringValue(char* baseKey,char* subKey,char* subKeyValueName,string value);
- //设置数值类型的键值 
- int RegSetNumValue(char* baseKey,char* subKey,char* subKeyValueName,const int& value);
- //读取数值类型的键值 
- int RegGetNumValue(char* baseKey,char* subKey,char* subKeyValueName,int& value);

### 实现 ###
    HKEY GetBaseRegKey(char* keystr)
    {   
    HKEY hKey; 
    if(strcmp(keystr,"HKEY_CLASSES_ROOT")==0)   
    hKey=HKEY_CLASSES_ROOT; 
    if(strcmp(keystr,"HKEY_CURRENT_CONFIG")==0) 
    hKey=HKEY_CURRENT_CONFIG;   
    if(strcmp(keystr,"HKEY_CURRENT_USER")==0)hKey=HKEY_CURRENT_USER; 
    if(strcmp(keystr,"HKEY_LOCAL_MACHINE")==0)hKey=HKEY_LOCAL_MACHINE;   
    if(strcmp(keystr,"HKEY_USERS")==0)hKey=HKEY_USERS;   
    return hKey;
    }

    int RegDelValue(char* baseKey,char* subKey,char* subKeyValueName)
    {
    HKEY bKey,hKey; 
    LONG retVal; 
    char BaseKey[512];   
    char SubKey[512]; 
    char SubKeyValueName[512];   
    strcpy(BaseKey,baseKey); 
    strcpy(SubKey,subKey); 
    strcpy(SubKeyValueName,subKeyValueName);   
    bKey = GetBaseRegKey(BaseKey); 
    
    retVal = RegOpenKeyEx(bKey,chr2wch(SubKey),0,KEY_ALL_ACCESS,&hKey); //-- 打开子键
    
    if (retVal != ERROR_SUCCESS)  return 1;   
    
    retVal = RegDeleteValue(hKey,(LPCTSTR)chr2wch(SubKeyValueName));   
    
    if(retVal !=ERROR_SUCCESS)   
    {
    RegCloseKey(hKey);   
    return 2;   
    }   
    RegCloseKey(hKey);   
    return 0;
    }

    int RegSetStringValue(char* baseKey,char* subKey,char* subKeyValueName,string value)
    {
    HKEY bKey,hKey; 
    LONG retVal; 
    char BaseKey[512];   
    char SubKey[512]; 
    char SubKeyValueName[512];   
    strcpy(BaseKey,baseKey); 
    //strcpy(SubKey,"Software\\360desktop"); 
    strcpy(SubKey,subKey); 
    strcpy(SubKeyValueName,subKeyValueName);   
    bKey = GetBaseRegKey(BaseKey); 
    
    char* sclass="";
    DWORD nbf=0; 
    retVal = RegCreateKeyEx(bKey,chr2wch(SubKey),0,chr2wch(sclass),REG_OPTION_NON_VOLATILE,KEY_ALL_ACCESS,NULL,&hKey,&nbf);
    
    if (retVal != ERROR_SUCCESS)  return 1;   
    
    retVal = ::RegSetValueEx(hKey, chr2wch(SubKeyValueName), 0, 
    REG_SZ, (const BYTE *)value.c_str(), value.size()+1);
    
    if(retVal !=ERROR_SUCCESS)   
    {
    RegCloseKey(hKey);   
    return 2;   
    }   
    RegCloseKey(hKey);   
    return 0;
    }
    
    int RegSetNumValue(char* baseKey,char* subKey,char* subKeyValueName,const int& value)
    {
    HKEY bKey,hKey; 
    LONG retVal; 
    char BaseKey[512];   
    char SubKey[512]; 
    char SubKeyValueName[512];   
    strcpy(BaseKey,baseKey); 
    strcpy(SubKey,subKey); 
    strcpy(SubKeyValueName,subKeyValueName);   
    bKey = GetBaseRegKey(BaseKey); 
    
    char* sclass="";
    DWORD nbf=0; 
    retVal = RegCreateKeyEx(bKey,chr2wch(SubKey),0,chr2wch(sclass),REG_OPTION_NON_VOLATILE,KEY_ALL_ACCESS,NULL,&hKey,&nbf);
    
    if (retVal != ERROR_SUCCESS)  return 1;   
    
    retVal = ::RegSetValueEx(hKey, chr2wch(SubKeyValueName), 0, 
    REG_DWORD, (const BYTE *)&value, sizeof(int));
    
    if(retVal !=ERROR_SUCCESS)   
    {
    RegCloseKey(hKey);   
    return 2;   
    }   
    RegCloseKey(hKey);   
    return 0;
    }
    
    int RegGetStringValue(char* baseKey,char* subKey,char* subKeyValueName,string& value)
    {
    HKEY bKey,hKey; 
    LONG retVal; 
    char BaseKey[512];   
    char SubKey[512]; 
    char SubKeyValueName[512];   
    strcpy(BaseKey,baseKey); 
    strcpy(SubKey,subKey); 
    strcpy(SubKeyValueName,subKeyValueName);   
    bKey = GetBaseRegKey(BaseKey); 
    
    retVal = RegOpenKeyEx(bKey,chr2wch(SubKey),0,KEY_ALL_ACCESS,&hKey);
    
    if (retVal != ERROR_SUCCESS)  return 1;   
    
    char name[512] = {0};
    DWORD size = 512;
    DWORD type = 0;
    
    retVal = ::RegQueryValueEx(hKey,chr2wch(SubKeyValueName),NULL,&type,(LPBYTE)name,&size);
    
    if(retVal !=ERROR_SUCCESS)   
    {
    RegCloseKey(hKey);   
    return 2;   
    } 
    else
    {
    name[size] = '\0';
    value = name;
    }
    
    RegCloseKey(hKey);   
    return 0;
    }
    
    int RegGetNumValue(char* baseKey,char* subKey,char* subKeyValueName,int& value)
    {
    HKEY bKey,hKey; 
    LONG retVal; 
    char BaseKey[512];   
    char SubKey[512]; 
    char SubKeyValueName[512];   
    strcpy(BaseKey,baseKey); 
    strcpy(SubKey,subKey); 
    strcpy(SubKeyValueName,subKeyValueName);   
    bKey = GetBaseRegKey(BaseKey); 
    
    retVal = RegOpenKeyEx(bKey,chr2wch(SubKey),0,KEY_ALL_ACCESS,&hKey);
    
    if (retVal != ERROR_SUCCESS)  return 1;   
    
    DWORD size = 512;
    DWORD type = 0;
    
    retVal = ::RegQueryValueEx(hKey,chr2wch(SubKeyValueName),NULL,&type,(LPBYTE)&value,&size);
    
    if(retVal !=ERROR_SUCCESS)   
    {
    RegCloseKey(hKey);   
    return 2;   
    } 
    
    RegCloseKey(hKey);   
    return 0;
    }

### 调用示例 ###
    // 删除
    RegDelValue("HKEY_CURRENT_USER",REGISTER_PATH,"ikey");
    //读取数值
    if(RegGetNumValue("HKEY_CURRENT_USER",REGISTER_PATH,"iKey",iValue) == 0)
    {
    cout<<“success" <<endl;
    }
    //读取字符串
    if(RegGetStringValue("HKEY_CURRENT_USER",REGISTER_PATH,"strKey",strValue) == 0)
    {
    cout<<“success" <<endl;
    }
    //写入数值 
    RegSetNumValue("HKEY_CURRENT_USER",REGISTER_PATH,"iKey",iValue);
    //写入字符串
    RegSetStringValue("HKEY_CURRENT_USER",REGISTER_PATH,"strKey",strValue);

以上是常用的两种类型，供参考.