---
title: Android-base
tags: Android
categories: Android
top: true
---

## 安卓系统架构

- 应用层
- 应用框架
  - 应用开发组件、开发包
- dalvik虚拟机
  - Google自己设计用于Android平台的虚拟机，可支持.dex格式的java应用程序运行
- linux内核
  - 包括一些系统驱动等

## apk目录结构

1. assets
   - 静态文件，apk所使用的各种资源，比如图片、视频等等，通过API获取
2. res
   - 编译后的资源文件
3. lib
   - so动态链接库，用于底层的一些调用
4. dex
   - 虚拟机可执行文件，主要逆向目标
5. META-INF
   - 数字签名，用于校验apk是否被篡改
6. AndroidManifest.xml
   - 一个应用程序的配置清单，声明权限、组件等的基本信息
7. resources.arsc
   - 编译后资源的位置与id之间的映射关系

## Android四大组件

1. activity
   - 用户可视化界面
   - safe：1.点击劫持 2.拒绝服务 3.组件导出
2. service
   - 用于后台实现操作，如后台接受微信消息
   - safe：1.串谋攻击 2.拒绝服务
3. content provider
   - 为应用提供数据，统一接口
   - safe：1.数据库注入
4. broadcast receiver
   - 信息传输机制
   - safe：恶意广播

注：所有组件均需在AndroidManifest中注册声明

参考文章：[(1条消息) android四大组件(详细总结)_87now的博客-CSDN博客_android的四大组件](https://blog.csdn.net/ican87/article/details/21874321)

## Android打包流程

## Android安装流程

- 复制APk安装包到data/app目录下，解压扫描安装包，把dex文件保存到dalvik-cache目录，并在data/data目录创建对应的应用数据。（AndroidManifest.xml）

## arm流水线

- PC：程序计数器。永远指向取指地址
- IF：取指。同时确定下一条指令地址（指针指向下一条指令）
- ID：译码。同时给出即将使用的寄存器，或者让立即数进行拓展，亦或者是给出转移目的寄存器与转移条件（转移指令）
- EX：执行。
- MEM：访存。若为load/store指令，这个阶段就要访问存储器。另外，在这个阶段还要判定是否有异常要处理，如果有，那么就清除流水线，然后转移到异常处理例程入口地址处继续执行
- WB：回写。将运算结果保存到目标寄存器

#### 三级流水线

- 取指–>译码–>执行
- pc–>pc-4–>pc-8 (arm7)

#### 五级流水线

- 取指–>译码–>执行–>访存–>回写 (arm9)

#### Notice

- 我觉得需要强调的一个点是理论和实践是不一样的。不一定所有指令都要经历流水线的每个过程
- 具体参考文章：[CPU设计—五级流水线 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/438259856)

## ADB

- adb devices 列出当前安卓设备
- adb shell 进入安卓linux命令行
- adb push 上传
- adb pull 下载
- adb connect ip连接设备
- adb kill-server 断开连接
- adb forward tcp:23946 tcp:23946 端口转移到本地
- adb shell dumpsys activity top 查看顶层页面信息
- adb shell dbinfo 查看app使用的数据库信息
- adb shell screencap -p [path] 截屏
- adb shell input text [text] 文本框输入
- adb shell pm install / pm uninstall / pm list packages
- adb shell am start-activity -D -N [包名] debug

## termux使用

- 下载地址：[Termux | F-Droid - Free and Open Source Android App Repository](https://f-droid.org/packages/com.termux/)

#### 命令

- termux-setup-storage 获取存储权限

- pkg install <软件名> 安装软件

- pkg upgrade 更新软件

  termux-change-repo 切换源

## 静态插桩

- 找到函数入口点

## 敏感信息

#### 动态注册

- Jni_OnLoad

#### 签名

- getPackageManager
- getPackageInfo
- signatures

#### 模拟器检测

- checkcpuinfo

#### 反调试

- android-server

## apktool反编译命令

```
apktool目录进入命令行
apktool.bat d 反编译文件
```

## dex2+jdgui反编译

1. 修改apk文件后缀为zip，解压
2. 将反编的dex文件复制到dex2jar工具目录下
3. 使用命令生成jar文件

```
d2j-dex2jar.bat document_name.dex
```

如果出现下列报错(版本号不匹配):

```
com.googlecode.d2j.DexException: not support version.
     at com.googlecode.d2j.reader.DexFileReader.<init>(DexFileReader.java:151)
     at com.googlecode.d2j.reader.DexFileReader.<init>(DexFileReader.java:211)
     at com.googlecode.dex2jar.tools.Dex2jarCmd.doCommandLine(Dex2jarCmd.java:104)
     at com.googlecode.dex2jar.tools.BaseCmd.doMain(BaseCmd.java:288)
     at com.googlecode.dex2jar.tools.Dex2jarCmd.main(Dex2jarCmd.java:32)
```


用文本编辑软件打开dex文件，将文件头改为035或者036

```
dex
037 9!Q<€颧€?滣^氶wl?  p   x
```


正常情况应该是这样的(没有任何多余)：

```
E:\SoftWarePackage\Jd-GUI+dex2\dex2jar-2.0>d2j-dex2jar.bat classes3.dex
    dex2jar classes3.dex -> .\classes3-dex2jar.jar
```



1. 用jd-guijar反编译jar文件，命令是这个：

   ```
   java -jar jd-gui-1.5.0.jar
   ```

   - 其实就是执行jar文件而已，可以直接写bat桌面一键打开