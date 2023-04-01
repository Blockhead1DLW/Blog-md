---
title: Buuctf-Re
tags: Reverse
categories: Reverse
top: false
---


## [网鼎杯 2020 青龙组]jocker

1. exe文件，32位无壳，拖入IDA32
2. ctrl+f12定位主体函数，修改变量名提高可读性
3. F5反汇编发现提示栈错误，option调出栈指针，修改栈指针为0，重新F5
4. 查看输入，24位字符串，主体：复制->加密->校对->encypt函数->final函数
5. 发现wrong函数加密，omg函数校对，简单异或解密，提交错误，为障眼法（但是得到了正确的输入字符串）
6. wrong函数上方对输入的flag进行了复制，真正处理的是复制后的flag
7. 找到相应的加密函数encypt，结果是动态加密函数，无法直接F5
8. exe拖入OD，找到调用加密函数地址，下断点，输入正确字符串运行
9. F7单步进入加密函数，ODdump脱壳，生成新文件
10. 新文件拖入IDA，查看encypt函数，异或解密得到前19位flag
11. 还剩最后5位，只剩final函数，未知加密，提示是隐藏，推断和之前加密方式相同，为异或加密
12. 利用最后一位”}”推断出异或key，进行解密得到正确flag

> 核心：修改由于后续动态加密造成的栈指针错位，解密正确输入，OD动态加密脱壳

## [SWPU2019]EasiestRe

1. 拖入exeinfope，32位无壳，进IDA
2. ctrl+f找到main函数发现GetThreadContext、SetThreadContext等等函数，多线程没跑了，先放一边
3. ctrl+f12字符串查找flag找到主体函数(交叉引用)，修改变量名提高可读性
4. F5反编发现__debugbreak()，int3断点，可以准备改二进制了，继续查看还不止一个
5. 重新看线程，是个父子线程保护，子线程加密，查看函数捕获重要长数组v16、v15
6. 所以又是动态加密，返回__debugbreak()处tab看汇编，int3+nop的长度和v16对的上，那就把v16搬过去，改二进制，重新反编
7. 观察主体函数：输入->加密->check->输出
8. 进入加密，又有__debugbreak()，和上面一样的方式，用v15改二进制码，反编
9. 进入check函数，tab查看汇编，408638地址处发现“cmp [ebp+var_20], 18h”校对指令
10. 查看对应地址，找到校对字符串（即加密结果）
11. 返回加密函数，分析，三次加密(异或–>分段–>移位)
12. 逆写解密

核心：双线程保护系统，int3断点(0xcc)，二进制修改

## [WMCTF2020]easy_re

- perl处理：一种解释语言，运行时解压，动调处理
  - 搜索script关键词，找到perl脚本调用处
  - 再关键点下面设断点，F8运行到脚本执行完毕
  - 下一个eax中可见原文

## [BJDCTF2020]hamburger competition

- 一个unity游戏逆向问题

- 工具：dnspy

- 核心代码一般路径：

  > \TPH\TPH_Data\Managed\Assembly-CSharp.dll