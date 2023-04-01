---
title: Z3 & Yafu
tags: Crypto
categories: Crypto
top: false
---


## Z3
### 下载

> 直接搞到网盘上用链接吧：
> 链接：(https://pan.baidu.com/s/1gni_eYUC-Y_dQVogJh9QyA?pwd=1234)
> 提取码：1234

1. 下载解压
2. 将文件中bin目录配置到系统变量path中
3. 新建PYTHONPATH系统变量，路径为bin/python目录

### Use

1. open cmd

> Open the bin/Python directory in cmd

or

> 在bin/python目录下shift右键powershell

1. run python
2. import code

### 代码

- example for authority

  ```
  from z3 import *
  x = Real('x')
  y = Real('y')
  s = Solver()
  s.add(x + y > 5, x > 1, y > 1)
  print(s.check())
  print(s.model())
  ```

- 简单说一下

1. 导包z3
2. a = Real(b), a为方程中的未知数，b为输出时候用来表示a的变量
3. Solver()，简言之：解决函数，方程接收器
4. add()，括号里塞方程就行，用“,”隔开，或者你多用几个add()也行
5. 后面俩不用动了，就那样
6. 回车出结果


## yafu
### Explain
- Yafu is a tool that can be used for prime factor decomposition
### Download
- https://sourceforge.net/projects/yafu/

### Use
- Execute yafu-x64.exe in the file in cmd
### Function
- factor(num)
