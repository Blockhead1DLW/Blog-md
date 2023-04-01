---
title: GFSJ-Re
tags: Reverse
categories: Reverse
top: false
---


### secret-galaxy-300

- 在初始化函数中隐藏关键代码
- 找到关键代码动调就行

### rev300

- 水题，恢复switch递归（模拟递归求解序号应该最快）得到a1，然后异或

### hackme

- 乱序循环->异或加密

  ```c++
  #include <iostream>
  using namespace std;
  int main()
  {
  	int a[100] = {0x5f,0xf2,0x5e,0x8b,0x4e,0xe,0xa3,0xaa,0xc7,0x93,0x81,0x3d,0x5f,0x74,0xa3,0x9,0x91,0x2b,0x49,0x28,0x93,0x67};
  	for(int i = 1; i <= 22; i++){
  		unsigned long long  j = 0, v24 = 0;
  		while(j < i){
  			v24 = 1828812941 * v24 + 12345;
  			j++;
  		}
  		cout <<(char)(v24^a[i-1]);
  	}
  }
  ```

### re4-unvm-me

- 纯水题，pyc反编md5直接解密

### tt3441810

- 抽象十六进制数据文件，过滤得到flag

### BABYRE

- 这个题我觉得不错

- `if ( v5 == 14 && (*(unsigned int (__fastcall **)(char *))judge)(s) )`

- 这里的judge明显是个函数，但是看反编又是个数组，说明是个花指令，通过前面的执行修改机器码实现函数

- 所以想PatchByte、CP恢复函数，再写脚本如下

  ```c++
  #include <iostream>
  using namespace std;
  int main()
  {
  	string  s = "fmcd";
   	int v2 = 127;
    	string s1 = "k7d;V`;np";
    	for(int i = 0; i <= 3; i++){
    		cout <<(char)(s[i] ^ i);
    	}
    	cout <<(char)(v2^4);
    	for(int i = 5; i <= 13; i++){
    		cout <<(char)(s1[i-5] ^ i);	
    	}
  }
  ```

### crackme

- 一个nspack壳，利用pushad，popad手动脱掉就行，然后就是个简单异或

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
	string s = "this_is_not_flag";
	int a[100] = {0x12,0x04,0x08,0x14,0x24,0x5c,0x4a,0x3d,0x56,0x0a,0x10,0x67,0x00,0x41,0x00,0x01,0x46,0x5a,0x44,0x42,0x6e,0x0c,0x44,0x72,0x0c,0x0d,0x40,0x3e,0x4b,0x5f,0x02,0x01,0x4c,0x5e,0x5b,0x17,0x6e,0x0c,0x16,0x68,0x5b,0x12,0x00};
	for(int i = 0; i < 42; i++){
		cout <<(char)(s[i%16]^a[i]);
	}
}
```

## debug

- .net题目，dnspy打开，通过调试到入口点找到关键代码

### EASYHOOK

- hook函数修改值跳转到真正加密函数

```c++
#include <bits/stdc++.h>
using namespace std;
int main()
{
	int lis[100] = {0x61,0x6a,0x79,0x67,0x6b,0x46,0x6d,0x2e,0x7f,0x5f,0x7e,0x2d,0x53,0x56,0x7b,0x38,0x6d,0x4c,0x6e,0x00};
	for(int i = 0; i <= 18; i++){
		if(i == 18){
			lis[i] ^= 0x13;
		}else{
			if(i % 2){
				lis[i] ^= i;
				lis[i] += i;
			}else
				lis[i] ^= i;
		}
	}
	for(int i = 18; i >= 2; i-=2){
		lis[i] = lis[i-2];
	}
	for(int i = 0; i <= 18; i++){
		cout <<(char)lis[i];
	}
}
```

### ReverseMe-120

- 输入–>base64解码–>异或

## key

- 这题有点别调，字符串检索找到关键代码，然后根据加密逆写就行，建议7.0版本以下ida

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
	string s = "themidathemidathemida";
	string t = ">----++++....<<<<.";
	for(int i = 0; s[i]; i++){
		s[i] = (s[i]^t[i]) + 22 + 9;
	}
	for(int i = 0; s[i]; i++){
		cout <<(char)s[i];
	}
}
```

## easyre-153

- pipe、fork双进程
- 利用必然跳转让ida反汇编错误

```asm
mov     [ebp+var_D], al
mov     [ebp+var_C], 0
cmp     [ebp+var_C], 1
jnz     short loc_80486D3
```

- 去掉必然跳转以后重新反汇编仿写逻辑即可

```c++
#include <iostream>
#include <cstring>
using namespace std;
int main()
{
	int v2[100];
	string a1 = "69800876143568214356928753";

	  v2[0] = 2 * a1[1];
	  v2[1] = a1[4] + a1[5];
	  v2[2] = a1[8] + a1[9];
	  v2[3] = 2 * a1[12];
	  v2[4] = a1[18] + a1[17];
	  v2[5] = a1[10] + a1[21];
	  v2[6] = a1[9] + a1[25];
	 for(int i = 0; v2[i]; i++){
	 	std::cout <<(char)v2[i];
	 }
}
```

### Newbie_calculations

- 三个函数组成的一长串，就是简单的加减乘混淆

### notsequence

- 杨辉三角题目，贴个简单分析代码

```python
int __cdecl main()
{
  //不断输入数字flag
  do{
    v0 = v3++;
    scanf("%d", v0);
  }while (*(v3 - 1));
  //验证杨辉三角每一层之和为2^<sub>n-1</sub>
  v2 = sub_80486CD(&unk_8049BE0);
  if ( v2 == -1 )
    exit(0);
  //验证杨辉三角第n层的第i个元素等于前n-1层第i-1个元素的和
  if ( sub_8048783(&unk_8049BE0, v2) != 1 )
    exit(0);
  if(v2 == 20){   //确定一共20层
    puts("Congratulations! fl4g is :\nRCTF{md5(/*what you input without space or \\n~*/)}");
  }
  return 0;
}
```

- 所以将杨辉三角前20层的数字顺序拼接即可，网友脚本：

```python
def triangles():
	s=[1]
	while True:
		yield s
		s.append(0)
		s=[s[i-1]+s[i] for i in range(len(s))]
n=0
flag=''
for i in triangles():
	flag+=''.join(map(str,i))
	n+=1
	if n==20:
		break
import hashlib
flag=hashlib.md5(flag.encode()).hexdigest()
print("RCTF{"+flag+"}")
```

### easy_Maze

- 迷宫题，首先根据传参确定7*7迷宫以及迷宫存储位置
- 动调到输入之前，即step_2函数之前，获取迷宫，得到flag

### serial-150

- 有花指令，动调恢复代码推断即可

### reverse-for-the-holy-grail-350

- c++逆向，关键函数stringmod分析如下

```c++
__int64 __fastcall stringMod(__int64 *flag)
{
  __int64 v1; // r9
  __int64 v2; // r10
  __int64 cnt; // rcx
  int v4; // er8
  int *v5; // rdi
  int *v6; // rsi
  int v7; // ecx
  int v8; // er9
  int v9; // er10
  unsigned int v10; // eax
  int mod3; // esi
  int v12; // esi
  int v14[18]; // [rsp+0h] [rbp-60h] BYREF
  __int64 v15; // [rsp+48h] [rbp-18h] BYREF

  memset(v14, 0, sizeof(v14));
  v2 = *flag;
  cnt = v4 = 0;
//第三位判断逻辑
  do
  { 
    v14[cnt] = v2[cnt]; 
    if(cnt % 3 == 0){ v14[cnt] == firstchar[cnt / 3]; }
    ++cnt;
  }
  while ( cnt != len(flag));

//flag进行异或
  v5 = v14;
  v6 = v14;
  v7 = 666;
  do
  {
    *v6 ^= v7;
    v7 += v7 % 5;
    ++v6;
  }
  while ( len(flag) != v6 );
//第一三位的验证逻辑
  v8 = 1;
  v9 = 0;
  v10 = 1;
  mod3 = 0;
  do
  {
    if ( mod3 == 2 )
    {
      if ( *v5 != thirdchar[v9] ) wrong;			//mod2 == third
      if ( v10 % *v5 != masterArray[v9] ) wrong;	//mod1*mod2%mod3 == master
      ++v9;
      v10 = 1;
      mod3 = 0;
    }
    else{
      v10 *= *v5;
      if ( ++mod3 == 3 ) mod3 = 0;
    }
    ++v8,++v5;
  }
  while ( v8 != 19 );
```

- 解题脚本

```python
f3 = [0x41,0x69,0x6e,0x45,0x6f,0x61]
f2 = [0x2ef,0x2c4,0x2dc,0x2c7,0x2de,0x2fc]
f1 = [0x1d7,0xc,0x244,0x25e,0x93,0x6c]
flag = [0 for i in range(18)]
#mod3=0填充
for i in range(0,18,3):
	flag[i] = f3[i//3]
# 生成异或序列
x = 666
xor = []
for i in range(18):
	xor.append(x)
	x += x % 5
# mod3=2填充
for i in range(2,18,3):
	flag[i] = f2[i//3] ^ xor[i]
# 剩余位填充
for i in range(1,18,3):
	for j in range(35,125):
		if ((j^xor[i]) * (flag[i-1]^xor[i-1]) % f2[i//3] == f1[i//3]):
			flag[i] = j
			break
print("tuctf{",end='')
for i in flag:
	print(chr(i),end='')
print("}")
```

### babymips

- 数字高低位置换问题

```python
//cmp2
for ( i = 5; i < strlen(a1); ++i )
  {
    if ( (i & 1) != 0 )
      v1 = (a1[i] >> 2) | (a1[i] << 6);
    else
      v1 = (a1[i] << 2) | (a1[i] >> 6);
    a1[i] = v1;
  }
  if ( !strncmp(a1 + 5, (const char *)off_410D04, 0x1Bu) )
//cmp1为异或
```

- Solve

```python
x = "Q|j{g"
for i in range(len(x)):
    print(chr(ord(x[i]) ^ (32-i)),end='')
lis = [0x52,0xfd,0x16,0xa4,0x89,0xbd,0x92,0x80,0x13,0x41,0x54,0xa0,0x8d,0x45,0x18,0x81,0xde,0xfc,0x95,0xf0,0x16,0x79,0x1a,0x15,0x5b,0x75,0x1f]
for i in range(5,32):
    if(i & 1 != 0):
        print(chr(((lis[i-5]<<2)&0xff | (lis[i-5]>>6)&0xff)^(32-i)),end='')
    else:
        print(chr(((lis[i-5]>>2)&0xff | (lis[i-5]<<6)&0xff) ^ (32-i)), end='')
```

### APK-逆向2

```c++
private static void Main(string[] args)
{
    //监听端口
    string hostname = "127.0.0.1";
    int port = 31337;
    tcpClient.Connect(hostname, port);
    //自我查找输出flag
    string text = "Super Secret Key";
    string text2 = Program.read();
    client.Send(Encoding.ASCII.GetBytes("CTF{"));
    foreach (char x in text)
	{
		client.Send(Encoding.ASCII.GetBytes(Program.search(x, text2)));
	}
	Console.WriteLine("Success!");
}

//find & cmp
private static string search(char x, string text)
{
    int length = text.Length;
    for (int i = 0; i < length; i++)
    {
        if (x == text[i])
        {
            int value = i * 1337 % 256;
            return Convert.ToString(value, 16).PadLeft(2, '0');
        }
    }
    return "??";
}
```

- 监听本机端口运行得到flag
- nc -l -p 31773

```
import http.server

server_address = ('127.0.0.1', 31337)
handler_class = http.server.BaseHTTPRequestHandler
httpd = http.server.HTTPServer(server_address, handler_class)
httpd.serve_forever()
```

- 代码仿写得到flag
  - 本质上就是在源程序上取和key字符相同的字节下标，然后做一个计算

```python
key1="Super Secret Key"
key2=open('1.exe','r',encoding = 'unicode-escape').read()	
num=len(key2)

flag="CTF{"

def search(x,key2,num):
	for i in range(num):
		if x==key2[i]:
			value=i*1337%256;
			return '%02x' %value						
for j in key1:
	flag+=search(j,key2,num)


flag+="}"
print(flag)
```

### secret-string-400

- 虚拟机题目
- loadcode内是真正的cmp代码，将其转换为字节码逆写即可，简单异或