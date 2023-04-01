---
title: Script
tags: Script
categories: Crypto
top: true
---


## 前言

本来不想写的，后来发现自己脑子不太行根本记不住，查又不想查，所以慢慢积累吧，总有一天会用到的
我的设想是等未来达到一定数量以后再按python库整理，现在的话就这样散着

## IDC爬数据

```C++
auto addr = 0x0000000100008000;
auto i = 0;
for(i; i < 10; i = i+1)
{
	Message("%x",Byte(addr+i));
	//PatchByte(addr+i,Byte());
}
```

## sha1加密

```C++
import hashlib
origin = input()
temp = hashlib.sha1(origin.encode())
print(temp.hexdigest())
```

## 求模乘逆元

```C++
# e*d mod φ(n) = 1
import gmpy2
e = 'e'
phi = 'φ(n)'
d = gmpy2.invert(e,phi)
print(d)
```

## RSA共模攻击

- 使用同一个模数n

- 用两个不同的公钥e1、e2对同一段明文m进行加密得到两份密文c1、c2

- 解密公式：

  > m = (c1s1 % n * c2s2 % n) % n

- 公式推导：

  - 由共模可推出
    - me1 % n = c1
    - me2 % n = c2
  - 欧几里得扩展算法取s1、s2使得
    - e1 * s1 + e2 * s2 = 1
  - m = m(e1*s1+e2*s2) % n
  - m = (c1s1 % n * c2s2 % n) % n

```C++
import gmpy2
import  binascii
c1 =
n =
e1 =
c2 =
e2 =
r,s1,s2 = gmpy2.gcdext(e1, e2)    # 扩展欧几里得算法
# 公式
m = (gmpy2.powmod(c1,s1,n)*gmpy2.powmod(c2,s2,n)) % n   # 计算明文m
m = hex(m)[2:]
print(binascii.unhexlify(m))
```

## RSA已知p高位攻击

- 生成模式

  ```C++
  from Crypto.Util.number import *
  from gmpy2 import *
  from secret import flag
  import random
  e=
  c=
  m=bytes_to_long(flag)
  p=getprime(512)
  q=getprime(512)
  n=p*q
  c=pow(m,e,n)
  print("n={}".format(n))
  print("c={}".format(c))
  tmp=random.randint(100,300)
  print("p>>tmp={}".format(p>>tmp))
  #c=6423951485971717307108570552094997465421668596714747882611104648100280293836248438862138501051894952826415798421772671979484920170142688929362334687355938148152419374972520025565722001651499172379146648678015238649772132040797315727334900549828142714418998609658177831830859143752082569051539601438562078140 
  
  #n=102089505560145732952560057865678579074090718982870849595040014068558983876754569662426938164259194050988665149701199828937293560615459891835879217321525050181965009152805251750575379985145711513607266950522285677715896102978770698240713690402491267904700928211276700602995935839857781256403655222855599880553
  
  #p>>200=8183408885924573625481737168030555426876736448015512229437332241283388177166503450163622041857
  ```

- sage脚本求p

  ```C++
  n = 
  p = 
  p = p << bits				//bits为求解位数，比如已知312位p2，p为512位，bits = 200
  PR. = PolynomialRing(Zmod(n))
  f = x + p
  roots = f.small_roots(X=2 ^ bits,beta = 0.4)
  print(roots)
  print(p)
  ```

## RSA已知明文m高位攻击

- 生成模式

  ```C++
  from Crypto.Util.number import *
  from secret import flag
  import libnum
  flag="UNCTF{*************************}" 
  m=libnum.s2n(flag)
  p=libnum.generate_prime(1024)
  q=libnum.generate_prime(1024)
  n=p*q
  e=6
  c=pow(m,e,n)
  M=((m>>60)<<60)
  print("n=",n)
  print("c=",c)
  print("((m>>60)<<60)=",M) 
  ```

- sage求解m

  ```C++
  n = 
  c = 
  high_m = 
  R.<x> = PolynomialRing(Zmod(n), implementation='NTL')
  m = high_m + x
  M = m((m^6 - c).small_roots()[0])
  print(hex(int(M))[2:])
  ```

- python还原m

  ```C++
  import libnum
  M =
  print(libnum.n2s(M))
  ```

## Base64变表解密

```C++
import base64
# 原表
origin = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
# 变表
base = 'yzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/abcdefghijklmnopqrstuvwx'
# 密文
c = '1ovhXETgZUDgXFS='
# 映射表
table = str.maketrans(base,origin)
# 明文
m = str(base64.b64decode(c.translate(table)),encoding=('utf-8'))
print(m)
```

## SM4解密

- 题目部分解密源码

```C++
from libnum import s2n, n2s
from pysm4 import decrypt
cipher = 119384314703134531538573762305905297230
print(cipher)
sm4_key = s2n(b"where_are_u_now?")
print(n2s(decrypt(cipher, sm4_key)).decode())
```

## MD5加密

```C++
import hashlib

# 待加密信息
str = 'this is a md5 test.'

# 创建md5对象
hl = hashlib.md5()
hl.update(str.encode(encoding='utf-8'))

print('MD5加密前为 ：' + str)
print('MD5加密后为 ：' + hl.hexdigest())
```

## 参数注入

- hackgame2022 flag自动机

```C++
#include <windows.h>
#include <stdio.h>

int main(void){
    HWND target = NULL;

    // 获取窗口句柄
    target = FindWindowW(L"flag 自动机", L"flag 自动机");
    if (target == NULL){
        printf("error!");
        return -1;
    }
    printf("0x%x", target);

    // 发送消息
    PostMessageW(target, 0x111, 3, 114514);
    return 0;
}
```

## CRC32解密

- RoarCTF2019 polyre

```C++
secret = [0xBC8FF26D43536296, 0x520100780530EE16, 0x4DC0B5EA935F08EC,
          0x342B90AFD853F450, 0x8B250EBCAA2C3681, 0x55759F81A2C68AE4]
key = 0xB0004B7679FA26B3

flag = ""

# 产生CRC32查表法所用的表
for s in secret:
    for i in range(64):
        sign = s & 1                   #判断首位正负
        if sign == 1:
            s ^= key
        s //= 2
        if sign == 1:
            s |= 0x8000000000000000    # 防止负值除2，溢出为正值
    print(hex(s))                      # 输出表
    # 计算CRC64
    j = 0
    while j < 8:
        flag += chr(s&0xFF)
        s >>= 8
        j += 1
print(flag)
```

# CR4加密

- 密钥长度可变，加解密同一套代码

```c++
#include <stdio.h>
#include <string.h>

typedef unsigned longULONG;
 
/*初始化函数*/
void rc4_init(unsigned char *s, unsigned char *key, unsigned long len)
{
    int i = 0, j = 0;
    char k[256] = {0};
    unsigned char tmp;
    for (i = 0; i < 256; i++)
    {
        s[i] = i;
        k[i] = key[i % len];
    }
    for (i = 0; i < 256; i++)
    {
        j = (j + s[i] + k[i]) % 256;
        tmp = s[i];
        s[i] = s[j];
        s[j] = tmp;
    }
}
 
/*加解密*/
void rc4_crypt(unsigned char *s, unsigned char *Data, unsigned long len)
{
    int i = 0, j = 0, t = 0;
    unsigned long k = 0;
    unsigned char tmp;
    for (k = 0; k < len; k++)
    {
        i = (i + 1) % 256;
        j = (j + s[i]) % 256;
        tmp = s[i];
        s[i] = s[j];
        s[j] = tmp;
        t = (s[i] + s[j]) % 256;
        Data[k] ^= s[t];
    }
}
 
int main()
{
    unsigned char s[256] = { 0 };//S-box
    char key[256] = {"justfortest"};		
    //密文十六进制形式
    int pata[512] = {0x10,0xa8,0x1,0x36,0xa0,0x48,0x6a,0xb6,0xdb,0xf1,0x64,0xad,0x84,0x59,0x2,0x9d,0x8a,0x54,0x9e,0xc2,0x4d,0x27};
    //转换为字符串形式
    char pData[512] = {0};
    for(int i = 0; pata[i]; i++){
    	pData[i] = (char)pata[i];
    }

    rc4_init(s, (unsigned char*)key, strlen(key));//已经完成了初始化
    rc4_crypt(s, (unsigned char*)pData, strlen(pData));//crypt
    //out
    for(int i = 0; pData[i]; i++){
    	printf("0x%02x,",pData[i]);
    }
    printf("字符串格式为：\n%s",pData);
    return 0;
}
```