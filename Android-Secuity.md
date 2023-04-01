---
title: Android-Secuity
tags: Android
categories: Android
top: false
---



## Android反调试

### 文件检测

- 检测某一路径下是否存在调试文件

```c++
dir = opendir("/data/local/tmp");		//check dir
pid = getpid();	//获取pid
v2 = pid;
if ( dir ){
    while ( 1 ){
        v3 = readdir(dir);
        v4 = v3 == 0;
        filename = (const char *)(v3 + 19);
        if ( v4 ) break;
        if ( !strncmp(filename, "android_server", 0xEu) )
            kill(v2, 9);	//kill debug process
    }
    pid = closedir(dir, "android_server", 14);
}
```

### 端口检测

- 检测app运行时是否有常用调试端口被使用

```c++
FILE* pfile=NULL;
char buf[0x1000]={0};
char* strCatTcp= "cat /proc/net/tcp |grep :5D8A";	//check tcp
char* strNetstat="netstat |grep :23946";	//check the state of port 
//calls popen(), executes a shell and runs commands 
pfile=popen(strCatTcp,"r");		
int pid=getpid();
if(NULL==pfile){
    printf("fail\n");
    return;
}
//kill debug process
while(fgets(buf,sizeof(buf),pfile)){
    int ret=kill(pid,SIGKILL);
}
pclose(pfile);
```

### 进程名称检测

```c++
const int bufsize = 1024;
char filename[bufsize];
char line[bufsize];
char name[bufsize];
char nameline[bufsize];
int pid = getpid();
//先读取Tracepid的值
sprintf(filename, "/proc/%d/status", pid);
FILE *fd=fopen(filename,"r");
if(fd!=NULL){
    while(fgets(line,bufsize,fd)){
        if(strstr(line,"TracerPid")!=NULL){
            int statue =atoi(&line[10]);
            if(statue!=0){
                sprintf(name,"/proc/%d/cmdline",statue);
                FILE *fdname=fopen(name,"r");
                if(fdname!= NULL){
                    while(fgets(nameline,bufsize,fdname))
                        if(strstr(nameline,"android_server")!=NULL)
                            int ret=kill(pid,SIGKILL);
                }
                fclose(fdname);
            }
        }
    }
}
fclose(fd);
```

## Android安全机制

### 区域隔离

- 系统区和用户区分离，系统区只有读取权限

### 进程保护

- 独立进程空间，沙箱隔离

### 权限模型

- 权限声明 —AndroidManifest.xml
  - READ_CONTACTS：读取用户通讯录数据
  - RECEIVE_SMS：监测短信通知
  - ACCESS_COARSE_LOCATION：通过基站和wifi获取位置信息
  - ACCESS_FINE_LOCATION：通过GPS获取位置信息
- 权限保护划级
  - normal：申请即可用
  - dangerous：需要用户确认
  - signature：只会赋予与声明权限使用相同证书
  - signatureosystem：需要系统或者平台签名

### 文件访问

- Android apk的安装目录
  - /data/data 下会有每个应用程序的私有目录
  - /data/app 会保存所有安装文件的APK
  - /data/da1vik-cache 会有每个应用程序的核心文件DEX的缓存文件，主要是为了提高效率。
  - /System/app 通常是系统自带的应用程序目录
- 每个Android应用程序会在安装时分配一个独有的Linux用户ID，建立一个沙盒，这个用户ID会在安装时分配给它，并在该设备上一直保持同一个数值
- Android 文件存储的4种方式
  - Context. MODE_ PRIVATE: 默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容。
  - Context. MODE_ APPEND: 内容追加
  - Context. MODE_ WORLD_ READABLE：读取权限
  - Context. MODE_ WORLD_ WRITEABLE：写入权限

### apk签名

- jks、keystore等

### 加密

- 对称算法，包括AES、 DES、3DES、 RC5、RC2、PBE
- 非对称算法，包括RSA、 DSA、ECC
- 杂凑算法，包括SHA1、MD5、MD2
- 传输加密
  - Android系统的传输加密支持L2TP/IPSEC、 L2TP、 PPTPVPN， 支持SSLV 2. 0\3.0\3.1。

## 漏洞分析

### 端口扫描

- 传统扫描
  - 通过因特网包探索器判断主机是否存在
  - ICMP-Echo、Broadcast ICMP、Non-Echo ICMP
- TCP（传输控制协议）连接扫描
  - 三次握手
- TCP同步扫描
  - 两次握手
- TCPFIN扫描
  - 发送FIN控制报文
    - CLOSED端口发送RST控制报文
    - LISTEN端口无应答
- FTP（文件传输协议）反弹攻击
- UDP（用户数据报协议）扫描

### 指纹识别

### 特征匹配

### 模拟攻击

### 沙箱执行