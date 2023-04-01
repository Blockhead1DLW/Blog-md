---
title: Pyc decomplier
tags: Reverse
categories: Reverse
top: false
---


## 2022/10/25 重写

- 加一个内容：exe到pyc的转化

  - 可操作exe的识别（用python编译生成的exe文件一般来讲会比较大）
  - 具体处理（利用脚本完成）

- py脚本（pyinstxtractor）

  （[mirrors / extremecoders-re / pyinstxtractor · GitCode](https://gitcode.net/mirrors/extremecoders-re/pyinstxtractor?utm_source=csdn_github_accelerator)）

- 使用

  - 建议官方文档readme.md

  - 比较常见的操作就是将要转化的exe程序复制到脚本文件目录下，执行命令

    > python pyinstxtractor.py test.exe

### pyc文件反编

py生成的中间文件

> 先省略无数字，需要深入了解底层再说

### 处理

- 借用外部资源，安装uncompyle6

  > pip install uncompyle

- 使用

  > uncompyle6 name.pyc > name.py

- 或者直接用网上的在线反编译：
  (https://tool.lu/pyc/)

> 因为比较bug的一件事情是：
> 我遇到了某pyc文件用uncompyle6无法反编的情况，目前推测应该是版本不对的问题
> 后续再研究研究