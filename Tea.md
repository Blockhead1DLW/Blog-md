---
title: Tea
tags: Reverse
top: false
categories: Reverse
---


- 一写题就是变种xxtea，我已经无语了
- 茶类算法一般来讲顺序逆写加密代码即可
- 代码基于网络开源代码修改，侵删

## Tea

#### 变量

- delta常数：0x9e3779b9
- key：4个32位无符号整数
- v：2个32位无符号整数

#### 简述

- 利用key和delta对数据进行32轮简单和加密
- 特征：
  - 三异或
  - 4、5移位

#### 加密

```c++
void encrypt (uint32_t* v, uint32_t* k) {  
    uint32_t delta=0x9e3779b9;
    uint32_t v0 = v[0], v1 = v[1], sum = 0;               
    for (int i = 0; i < 32; i++) {                      
        sum += delta;  
        v0 += ((v1<<4) + k[0]) ^ (v1 + sum) ^ ((v1>>5) + k[1]);  
        v1 += ((v0<<4) + k[2]) ^ (v0 + sum) ^ ((v0>>5) + k[3]);  
    }                                                
    v[0] = v0; 
    v[1] = v1;  
} 
```

#### 解密

```c++
void decrypt (uint32_t* v, uint32_t* k) {  
    uint32_t delta = 0x9e3779b9; 
	uint32_t sum = 0xc6ef3720     //sum = delta*32 = 0x13c6ef3720，取32位，高位数据溢出 
    uint32_t v0 = v[0], v1 = v[1]; 
    for (int i = 0; i < 32; i++) {                         
        v1 -= ((v0<<4) + k[2]) ^ (v0 + sum) ^ ((v0>>5) + k[3]);  
        v0 -= ((v1<<4) + k[0]) ^ (v1 + sum) ^ ((v1>>5) + k[1]);  
        sum -= delta;  
    }                                            
    v[0] = v0; 
    v[1] = v1;  
}  
```

## xxTea

- key和delta如上，加密轮数由数据长度n决定
- 改魔数的就另当别论，一般解tea都是用c++写

#### c++

```c++
#include <bits/stdc++.h>
using namespace std;
//常见变化点
#define MX (((z>>5^y<<2) + (y>>3^z<<4)) ^ ((sum^y) + (key[(p&3)^e] ^ z)))  
//key定义，4个32位无符号整数
const uint32_t key[4] = {};
//delta
uint32_t delta = 0x9E3779B9;


void encrypt(uint32_t* flag, int n){
  uint32_t y, z, sum;  
  rounds = 52 / n + 6;
  sum = 0;
  z = flag[n - 1];
  do
  {
    sum += delta;
    e = (sum>>2) & 3;
    for ( i = 0; i < n - 1; ++i )
    {
      y = flag[i+1];
      z = flag[i] += MX;
    }
    y = flag[0];
    z = flag[n-1] += MX;
  }
  while (--rounds);
}


void decrypt(uint32_t* flag, int n){
  uint32_t y, z, sum;  
  rounds = 52 / n + 6;
  sum = rounds*delta;
  y = flag[0];
  do
  {
    e = (sum>>2) & 3;
    for ( i = n-1; i > 0; i--)
    {
      z = flag[i-1];
      y = flag[i] -= MX;
    }
    z = flag[n-1];
    y = flag[0] -= MX;
    sum -= delta;
  }
  while (--rounds);
}
```

#### python

```python
"""
参数描述:
  DELTA: 神秘常数δ，它来源于黄金比率，以保证每一轮加密都不相同。但δ的精确值似乎并不重要，这里 TEA 把它定义为 δ=「(√5 - 1)231」
      v: 需要加解密的数据，格式为32位的无符号整数组成的数组
      n: n表示需要加密的32位无符号整数的个数（例：n为1时，只有v数组中的第一个元素被加密了），n不能大于v的长度
      k: 密钥，格式为4个32位无符号整数组成的数组，即密钥长度为128位
"""
import struct
from ctypes import c_uint32
 
DELTA = 0x9E3779B9
 
def encrypt(v, n, k):
    rounds = 6 + int(52 / n)
    sum = c_uint32(0)
    z = v[n - 1].value
    while rounds > 0:
        sum.value += DELTA
        e = (sum.value >> 2) & 3
        p = 0
        while p < n - 1:
            y = v[p + 1].value
            v[p].value += (((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4)) ^ ((sum.value ^ y) + (k[(p & 3) ^ e] ^ z)))
            z = v[p].value
            p += 1
        y = v[0].value
        v[n - 1].value += (((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4)) ^ ((sum.value ^ y) + (k[(p & 3) ^ e] ^ z)))
        z = v[n - 1].value
        rounds -= 1
 
 
def decrypt(v, n, k):
    rounds = 6 + int(52 / n)
    sum = c_uint32(rounds * DELTA)
    y = v[0].value
    while rounds > 0:
        e = (sum.value >> 2) & 3
        p = n - 1
        while p > 0:
            z = v[p - 1].value
            v[p].value -= (((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4)) ^ ((sum.value ^ y) + (k[(p & 3) ^ e] ^ z)))
            y = v[p].value
            p -= 1
        z = v[n - 1].value
        v[0].value -= (((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4)) ^ ((sum.value ^ y) + (k[(p & 3) ^ e] ^ z)))
        y = v[0].value
        sum.value -= DELTA
        rounds -= 1
 
 
def test1():
    k = [2, 2, 3, 4]
    v = [c_uint32(1), c_uint32(2)]
    print('plain: 1 2')
    encrypt(v, len(v), k)
    print('encrypted: %X %X' % (v[0].value, v[1].value))
    decrypt(v, len(v), k)
    print('decrypted: %X %X' % (v[0].value, v[1].value))

————————————————
此段代码为转载，原文链接：https://blog.csdn.net/koxb/article/details/120088234
```