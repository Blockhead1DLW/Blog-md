---
title: Affine
tags: Crypto
categories: Crypto
top: false
---


## 原理

### 实现原理

- 简单来说就是利用加密函数将一个字母映射为另一个字母

### 加密函数

#### E(x) = (ax+b)%n,其中

- n为所设置的字母编码表的大小
- a,b为自选数，只要满足a与n互质即可
- x为明文编码后的数字

### 解密函数

#### D(x) = a-1(x-b)%n

- a-1是a在Zn群的乘法逆元

> 乘法逆元这个东西等后续完全搞懂了再解释，主要的一点是满足a-1*a%n=1

## 例题 [BITSCTF2017]fanfie

> Brute and get the base 32 format of flag.
> encrypted.txt: MZYVMIWLGBL7CIJOGJQVOA3IN5BLYC3NHI

### 题解

题目是一如既往的简洁
首先题目告诉我们有base32编码
同时由题目来源可知flag格式为BITSCTF{}

接下来就是玄学猜想:
编码后变长了，或许刚好和题目给的密文等长吧，考虑单表加密
I变M，J变Z，E变Y，很明显没直接的映射规律，考虑[仿射密码](http://woodenmandu.cn/2022/09/06/Affine/)

下一步计算a，b

> 别忘记这是base32，加密函数为E(x) = (ax+b)%32

- 小算一下，拿两组数据：

  > 8(I)–>12(M)
  > 9(J)–>25(Z)

- 函数差值搞一下：25-12=(9-1)a
  a=13，b顺理成章等于4

- 然后google找个求逆元的网站得到逆元等于5
  得到解密函数D(x) = 5(x-4)%32

通过解密得到

```
密文：MZYVMIWLGBL7CIJOGJQVOA3IN5BLYC3NHI
明文：IJEVIU2DKRDHWUZSKZ4VSMTUN5RDEWTNPU
```

最后base32解码就能得到flag:

> BITSCTF{S2VyY2tob2Zm}