---
title: Android-Re
tags: Android
categories: Android
top: false
---

# Smail

### 寄存器命名

- 均为32位，64位用2个寄存器
- p：p0、p1 函数参数
- v：v0、v1 函数内部变量

### 数据类型

| 数据类型——Smail | 数据类型——Java |
| --------------- | -------------- |
| V               | void           |
| Z               | boolean        |
| B               | byte           |
| S               | string         |
| C               | char           |
| I               | int            |
| J               | long           |
| F               | float          |
| D               | double         |

- 数组：[+数据类型 eg：[I <==> int[]

### 文件头

- .class<访问权限><关键修饰字><类名>
- .super<父类名>
- .source<源文件名>

### 函数声明

```
.method<访问权限><关键修饰字><方法原型（返回值类型）>
<.locals/.registers>		    		# 非参数变量数目
[.param]								# 声明方法中的参数名
[.line]									# 源码中的行号
.end method
```

### 常量操作指令

- const
- const-string v1,”re” 将string类型常量re放进寄存器v1

### 方法调用指令

- invoke-kind {vA、vB}, method@DDDD
- invoke-(virtual、super、static | 正常函数、父类函数、静态函数) { 传入参数 }，方法名

### 移位指令

- move
- move-result-object v0 将上一步的函数返回值存进v0

### 分支判断指令

- if-eq：如果vA等于vB则跳转。Java语法表示为“if(vA == vB)”
- if-ne：如果vA不等于vB则跳转。Java语法表示为“if(vA != vB)”
- if-lt：如果vA小于vB则跳转。Java语法表示为“if(vA < vB)”
- if-ge：如果vA大于等于vB则跳转。Java语法表示为“if(vA >= vB)”
- if-gt：如果vA大于vB则跳转。Java语法表示为“if(vA > vB)”
- if-le：如果vA小于等于vB则跳转。Java语法表示为“if(vA <= vB)”
- if-testz vAA, +BBBB：条件跳转指令。拿vAA寄存器与0比较，如果比较结果满足或值为0时就跳转到BBBB指定的偏移处。偏移量BBBB不能为0。 if-testz类型的指令有以下几条：
- if-eqz：如果vAA为0则跳转。Java语法表示为“if(vAA == 0)”
- if-nez：如果vAA不为0则跳转。Java语法表示为“if(vAA != 0)”
- if-ltz：如果vAA小于0则跳转。Java语法表示为“if(vAA < 0)”
- if-gez：如果vAA大于等于0则跳转。Java语法表示为“if(vAA >= 0)”
- if-gtz：如果vAA大于0则跳转。Java语法表示为“if(vAA > 0)”
- if-lez：如果vAA小于等于0则跳转。Java语法表示为“if(vAA <= 0)”

# Frida

## frida（hook框架）安装配置及基本使用

#### 安装配置

- pip install frida > 核心
- pip install frida-tools > 工具包
- frida –version > 版本检测
- [Releases · frida/frida (github.com)](https://github.com/frida/frida/releases)安装对应版本的server，如server-android-x86
- 将刚才下载的server文件上传到安卓手机中

#### 使用

- 在手机中给server文件可执行权限并运行

  ```
  chmod +x 文件
  ./文件
  ```

- 在cmd中执行命令

  - frida-ps -Ua 列出当前正在运行的程序
  - frida -U -f 包名 hook对应程序

### Objection配置使用

- 基于frida框架的一个自动化脚本集成
- 安装：pip3 install objection

#### 使用

- Objection -g 包名/PID explore 注入app进程
- android root disable 绕过root检测
- android proxy set 设置代理
- android sslpinning disable 绕过证书固定

## 重载

```javascript
function main(){
    Java.perform(function(){
        var MainActivity = Java.use('com.example.retest.MainActivity')
        //overload重载
        MainActivity.fun.overload('int','int').implementation = function(x, y){
            
            ...
        
        }
    })
}
setImmediate(main)
```

## 类函数和实例方法主动调用

```javascript
function main(){
    Java.perform(function(){
        var MainActivity = Java.use('com.example.retest.MainActivity')
        //静态函数，直接调用
        MainActivity.staticsecret()
        
		//动态函数主动调用
        Java.choose('com.example.retest.MainActicity',{
            //实例化对象
            onMatch:function(instance){
                //调用
                instance.secret()
            },
            onComplete: function(){
                console.log('search Complete')
            }
        })
    })
}
setImmediate(main)
```