---
title: Linux
tags: Linux
categories: Linux
top: false
---

### linux指令

- uname，查看系统信息
  - -s：System name
  - -n：Network (domain) name
  - -r ：Kernel Release number
  - -v ：Kernel Version
  - -m：Machine (hardware) name
  - -a：All of the above
- file，查看文件类型
- head，查看文件内容
- ldd，查看文件依赖
- xxd，十六进制转储工具
  - xxd -c 32：每行显示32字节
  - xxb -b：二进制显示
  - -l：指定转储长度
- dd，同转储工具，容易造成文件覆盖
- readelf，elf查看解析
  - readelf -h：查看elf头
  - readelf -s：查看符号表
- nm，文件符号解析
  - nm -D：解析动态·符号表
- c++filt，符号解析（传入参数为要解析的函数）
- strings，查看字符串
  - -n：指定最小字符长度
  - -d：只输出data段字符串
- strace，显示文件执行时的系统调用
  - strace -p pid：附加到pid进程
- ltrace，显示文件执行时的库文件调用