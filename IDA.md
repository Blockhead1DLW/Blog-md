---
title: IDA
tags: Tool
categories: Reverse
top: false
---


## 快捷操作

| 操作           | 作用                           |
| -------------- | ------------------------------ |
| F5             | 反汇编                         |
| 函数列表Ctrl+F | 搜索函数                       |
| X              | 交叉引用                       |
| G              | 跳转到指定地址                 |
| Shift+F12      | 字符串列表                     |
| Alt+T          | 按指令查找                     |
| N              | 重命名(函数名等)               |
| Ctrl+Z         | 操作撤销                       |
| D              | 将字符串等元素转为数据         |
| A              | 将数据转变为字符串             |
| C              | 将数据转变为汇编代码           |
| U              | 将字符串转变为原始数据         |
| Shift+E        | 导出数据                       |
| Shift+F2       | 脚本嵌入                       |
| ;              | 添加注释(所有交叉参考处均显示) |
| :              | 添加注释(仅在此处显示)         |
| Esc            | 后退一步                       |
| Ctrl+Enter     | 前进一步                       |
| Alt+M          | 标记当前位置                   |
| Ctrl+M         | 跳转到标记位置                 |
| P              | 创建函数                       |

## elf调试

- 将linux_server/64复制到kali运行
- ida设置调试

## so调试

- 将android_server复制到安卓环境中（模拟器 or 测试机）
- chmod 777 android_server并运行
- adb forward将调试端口转移到本机
- 将so文件所在apk以debug模式启动
- IDA打开要调试的文件，附加apk进程