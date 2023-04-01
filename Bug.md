---
title: Loophole
tags: Reverse
categories: Reverse
top: false
---

# 漏洞相关

### 栈溢出

```
需要注意栈向低地址生长，栈中的变量向高地址生长，所以溢出后覆盖的是高地址区域，逆向中可用来隐藏信息以及关键函数入口等等
```

#### 案例

```c++
#include "stdafx.h"
#include <stdio.h>
#include <windows.h>

void testFunc(char *Buf)
{
	char testBuf[8];
	memcpy(testBuf, Buf, 64);
	return;
}
int main(int argc, char *argv[])
{
	char Buf[64] = {0};
	memset(Buf, 0x41, 64);
	testFunc(Buf);
	return 0;
}
```

#### 分析

- testfunc中给8字节的testbuf数组赋值了64字节数据溢出，从而导致函数返回地址错误

- ```asm
  0019FE90 41414141
  0019FE94 00401057 返回到 test.00401057 自 test.00401430
  0019FE98 41414141
  0019FE9C 41414141
  0019FEA0 41414141
  0019FEA4 41414141
  0019FEA8 41414141
  0019FEAC 41414141
  0019FEB0 41414141
  0019FEB4 41414141
  0019FEB8 41414141
  0019FEBC 41414141
  0019FEC0 41414141
  0019FEC4 41414141
  0019FEC8 41414141
  0019FECC 41414141
  ```
#### 防范

  ```c++
  //加入长度判断
  int maxsize = 8;
  if(maxsize > sizeof(testBuf))
  {
      //数据截断
   	memcpy(testBuf, Buf, maxsize);   
  }
  ```

### 堆溢出

### 整型溢出（赋值类型大小不匹配）

1. 存储溢出

   ```c++
   #include <iostream>
   using namespace std;
   int main(void)
   {
   	int a = 0x12345678;
   	short b = a;
   	cout <<"0x" <<hex <<b <<endl;
   	return 0;
   }
   //0x5678,丢失数据
   ```

2. 运算溢出（计算后超出原类型数据范围，导致溢出）

3. 符号问题

### UAF漏洞

- use-after-free

- 大致过程

  - 函数A顺序调用B、C、D三个子函数
  - 函数B释放了函数中的一部分资源，让函数中的某些指针变为空指针
  - 函数C又申请使用刚才释放的资源空间（干坏事）
  - D函数判断不严密，导致刚才释放的空指针被继续使用

- 相当于就是：在一个空间资源被释放后，将恶意信息注入该空间，然后又在不知情的情况下继续使用了该空间，引发漏洞

- 星盟公开课的一段代码（不知道哪出了问题，暂时没有预期输出）

  ```c++
  #include <stdio.h>
  #include <string.h>
  #include <malloc.h>
  #include <stdlib.h>
  unsigned long long target[100];
  int main()
  {
  	unsigned long long *p = malloc(0x10);
  	free(p);
  	p[0] = target; // p_chunk->fd is modified
  	target[0] = 0; // prev size
  	target[1] = 0x21; // size
  	malloc(0x10); // must be same as p
  	char *q = malloc(0x10); // must be same as target
  	memcpy(q, "hello",6);
  	printf("%s\n",&target[2]);
  	return 0;
  }
  ```

# 加脱壳相关

- 通俗地讲，不管是压缩壳还是加密壳，都是附着于原程序，对其进行各种处理。然后在程序加载到内存中时，先运行壳的解压引擎还原程序，再执行程序。二者的界限也越来越模糊
- 为了保证效率，对于一个压缩引擎，我们要保证他有足够快的解压速度，以确保程序在运行时的速度。至于压缩速度就略显不重要

#### 压缩壳

- upx
- ASpack

#### 加密壳

- ASProtect
- Armadillo：双进程运行，cc保护
- EXECryptor
- Themida

#### 虚拟机保护

- VMProtect