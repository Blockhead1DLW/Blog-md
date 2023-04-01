---
title: Android-Create
tags: Reverse
top: false
categories: Reverse
---


## 简述

### ast树(抽象语法树)

### 官方解释

- 是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。

### 简单描述

- 其实很简单，ast树就跟一本书的目录一样，只不过这个目录更加详细。以一种提取关键词和功能的方式完美展现一段代码的层次关系

### 简单介绍

例如js代码`x = a + b`生成`ast`树：

```
"type": "AssignmentExpression",
"start": 0,
"end": 7,
"operator": "=",
"left": {
    "type": "Identifier",
    "start": 0,
    "end": 1,
    "name": "x"
},
"right": {
    "type": "BinaryExpression",
    "start": 4,
    "end": 7,
    "left": {
    "type": "Identifier",
    "start": 4,
    "end": 5,
    "name": "a"
    },
    "operator": "+",
    "right": {
    "type": "Identifier",
    "start": 6,
    "end": 7,
    "name": "b"
    }
}
```

可以看到其内容是十分详细的：

- name：目标元素，作用对象
- start，end：name对象在程序代码中的起止位置
- operator：执行操作
- type：操作种类，比如说一个函数、一个返回值、一个运算、一个变量等等

### 在线生成ast树结构网站：

- https://astexplorer.net/

## ast逆向

- 目前我接触的这方面东西也很少，基本上要做的事情只有两种：
  1. 给一个ast树，然后让你去除其中的混淆操作
  2. 给定ast还原代码，这个代码可以是js，也可以是其他任何语言
- 对于代码还原，这种很难有现成工具使用，小代码手推，大代码自己写针对脚本
- 手推的话：
  - 定位operator先还原运算表达式
  - 由内层向外，还原一部分删一部分树结构
  - 独立度大的优先，按函数模块还原
  - 自己写对应代码生成ast树比对
- 大概就这样