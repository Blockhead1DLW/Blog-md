---
title: Base64
tags: Crypto
categories: Crypto
top: false
---

- 细琐编码过程

------

### 编码过程

1. 将目标字符依照ASCII码表变为纯二进制
2. 按每6位一个单元进行重新切割，（不够6位的地方补0）
3. 对照base64编码表，编码出每个单元所代表的字符

### base64编码表

![v2-ff0ffbecbd68704bf1ddf8f7d53095b8_r.jpg](https://s2.loli.net/2022/09/06/QiUuEKzCvZIFNDV.jpg)

### eg：

- 目标字符：dlw
- 第一步：转变为二进制串

> 01100100(d) 01101100(l) 01110111(w)

- 第二步：6位切割

> 011001 | 000110 | 110001 | 110111

- 第三步：查找每一单元对应的字符，重新编码

> 011001(Z) | 000110(G) | 110001(x) | 110111(3)

- 得到结果：ZGx3

### python编码

```
import base64
s = input()
t = base64.b64encode(s.encode())
print(t.decode())
```

### python解码

```
import base64
s = input()
t = base64.b64decode(s.encode())
print(t.decode())
```

### Notice

1. base64的特征之一：结果串长度一定为4的倍数
2. 为了补齐4倍的单元，编码时可能会出现末尾n个单元全部为空的情况，这时候就用“=”补齐，这也是识别base64编码的一种方式

#### 比如两个字符“dl”

- 二进制：01100100(d) 01101100(l)
- 切割：011001(Z)) | 000110(G) | 1100（补两个0）(w) | （空单元）(=)
- 结果：ZGw=

### 涉及题目

- python-trade