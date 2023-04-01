---
title: UNCTF2022-WP
tags: CTF
categories: CTF
top: false
---


# 总述

- 这场比赛算是我第一场长时间全程投入的比赛，每天从早上打到晚上12点，不得不说，还挺享受整个过程。全程rank最好的时候是第3，前期分数拉满，中期乏力，最后两天直接冥想坐牢，比赛结束前半小时才A掉培根密码题，守住了rank19。后10名的分差是真的小，再加上最后垂直上分的佬，节目效果拉满，刺激。
- 赛题方面，从我的角度看的话：
  - reverse方面的题可能不是很友好，重复+部分偏门。2个upx壳、1个手动壳，2个rust逆向，以及ast普及和小改xxtea。
  - crypto和misc方向的题目基本知识点覆盖还是比较全面的。很多东西之前都没有见过，纯google现场学习。尤其是隐写文章看了一大堆，虽然misc没写出几道…….太菜了
- 整个比赛打下来确实学到不少东西，挺好

# WP

## `Reverse`

### whereisyourkey-广东海洋大学

- 直接模拟加密过程就行了，不多说
- 远程linux调试elf执行到加密结束提取数据也行

### halo-绍兴元培

```
考点`：`upx脱壳`，`逻辑运算`，`异或解密
```

- 这个题多少有点绕

- 最后用来校对的那一堆异或和或，只有每个异或结果为0条件才为真，也就是说单个异或的结果都是相等的，通过这里就可以得到操作过后的flag

- 中间的加密，通过分析代码可以发现规律

  ```
  f0 = f0^0  
  ff1 = f0^0^f1^1
  f2 = f2^2^f0^0^f1^1
  
  相当于，顺序加密：fn = fn-1 ^ n
  
  所以反过来异或回去就完了
  ```

- 最上面还有个array^0x33，就是array数组第一位异或，弄一下就行

- 解密脚本

  ```c++
  list = [102, 0xb,0x68,0xc,0x73,0x3e,0xc,0x3a,0x5d,0x1b,0x21,0x75,0x4f,0x20,0x4c,0x71,0x58,0x7b,0x59,0x2c,0x0,0x77,0x58,0x77,0xe,0x72,0x5b,0x26,0xb,0x70,0xa,0x77,0x66,0x77,0x36,0x76,0x37,0x76,0x62,0x72,0x6d,0x27,0x3f,0x77,0x26]
  x = 44
  while x > 0:
      list[x] = list[x]^list[x-1]^x
      x -= 1
  for i in list:
      print(chr(i),end='')
  # flag{H41oO0_6bb2920f8b98ae3f1fdb10cced277c2c}
  ```

- 对了，还有个upx脱壳，直接工具搞一下

### ezzzzre-广东海洋大学

- 有个upx壳，工具脱掉

- F5主函数

- 核心代码就这一行：

  ```
  if ( Str[i] != 2 * aHelloctf[i] - 69 )
  ```

- 提取Helloctf[i]直接计算即可

### Sudoku-陆军工程大学

```
考点`：`动态调试`，`数据提取`，`python
```

- IDA反编F5+cmd运行测试

- 好像是个数独，分析一下

  ```c++
  int __cdecl main(int argc, const char **argv, const char **envp)
  {
    
    ......  
  //生成检测数独
    while ( (unsigned int)Prepare((int (*)[9])v12) )		  
    {
      for ( i = 0; i <= 8; ++i )
      {
        for ( j = 0; j <= 8; ++j )
        {
          if ( !v12[9 * i + j] )
          {
            for ( k = 0; k <= 8; ++k )
              v4[k] = 0;									//生成函数
            line((int (*)[9])v12, v4, i);
            row((int (*)[9])v12, v4, j);
            SquireNine((int (*)[9])v12, v4, i, j);
            FillBlank((int (*)[9])v12, v4, i, j);
          }
        }
      }
    }
  //提示信息输出
    puts("This is a game called Sudoku,just enjoy it!");
    puts("Pls input your answer in the following format:");
    qmemcpy(v14, &unk_4051C0, 0x144ui64);
    for ( i = 0; i <= 8; ++i )
    {
      for ( j = 0; j <= 8; ++j )
        printf("%d ", (unsigned int)v14[9 * i + j]);		
      putchar(10);
    }
  //输入数独
    puts("Then you will get your flag!");
    for ( i = 0; i <= 8; ++i )
    {
      for ( j = 0; j <= 8; ++j )
        scanf("%d", &v11[9 * i + j]);			              
    }
  //校验
    for ( k = 0; k <= 8; ++k )
    {
      for ( m = 0; m <= 8; ++m )
      {
        if ( v11[9 * k + m] != v12[9 * k + m] )			 
        {
          puts("Y0u_Ar3_Wr0ng!");
          exit(1);
        }
      }
    }
    printf("Y0u_Ar3_R1ght!");
  }
  ```

- 第一眼就看到v11和v12比较，所以我们要想办法把v12搞出来，首先跟踪上面的生成函数，很复杂，那就动态调试，让机器生成v12我们去提取数据

- v12生成函数结束后位置设断点，运行，然后f5反编找到v12对应栈位置，写脚本提取数据

  ```c++
  auto addr = 0x000000000062FA10;
  auto i = 0,cnt = 0;
  for(i; i < 81*4; i = i+4)
  {
  	Message("%x ",Byte(addr+i));
  	cnt = cnt + 1;
  	if(cnt % 9 == 0)
  	   Message("\n");
  }
  //8 5 2 4 9 1 6 7 3 
  //1 9 6 7 3 8 2 5 4 
  //4 3 7 5 6 2 9 1 8 
  //5 2 8 1 4 6 3 9 7 
  //3 7 4 9 2 5 8 6 1 
  //9 6 1 3 8 7 4 2 5 
  //2 1 9 8 5 4 7 3 6 
  //7 4 3 6 1 9 5 8 2 
  //6 8 5 2 7 3 1 4 9
  ```

- 继续运行，输入提取的数独

- 运行到显示：

  > UNCTF{chr(29+vme)chr(15+vme)chr(29+vme)chr(24+vme)chr(39+vme)chr(25+vme)chr(29+vme)chr(20+vme)chr(32+vme)}

- python解一下

  ```python
  print('UNCTF{',end='')
  print(chr(29+50),end='')
  print(chr(15+50),end='')
  print(chr(29+50),end='')
  print(chr(24+50),end='')
  print(chr(39+50),end='')
  print(chr(25+50),end='')
  print(chr(29+50),end='')
  print(chr(20+50),end='')
  print(chr(32+50),end='')
  print('}',end='')
  # UNCTF{OAOJYKOFR}
  ```

### ezast-浙江师范大学

```
考点`：`ast树`，`js`，`代码还原
```

- 一口老血吐出来

- 第一次遇到这种题，研究了一下ast树，然后就开始手推源代码

- 一边构建代码看ast一边还原（得亏有个ast在线网站）

- 还原代码

  ```javascript
  function ezdecode(flag,key)
  {
    var arr_data = flag.split()
    return arr_data.map(i => String.fromCharCode(i.charCodeAt()^(key += 1))).join('')
  }
  var $_a = test()
  $_a -= 1145*100
  $_a += 0xb
  console.log(ezdecode("OTYN\\a[inE+iEl.hcEo)ivo+g",$_a))
  function test(){
    return 114514
  }
  ```

- emmmm，每次只能解密一个字符，然后我就直接手删了半天加密串把flag构造出来了

- 对了，还有一个用来转义的\要删掉

- UNCTF{Ast_1s_v4ry_u3slu1}

### HelloRust

- 这题，一切都在动调中

- 首先通过关键词检索识别为rc4加密

- 修改内存数据，将unctf等等一系列数据变为字符串

- 动调，在keyinit函数处找到密钥`Unctf2022`

- 在cmp函数上方找到密文

- 提取密文

  ```c++
  auto addr = 0x00005622B1A340A6;
  auto i = 0;
  Message("\n");
  for(i; i < (0x00005622B1A340C1 - addr + 1); i = i+1)
  {
  	Message("%02x",Byte(addr+i));
  }
  //876927216fc731261b6c3a749a626ea002811d85e0e2d071f4a3090e
  ```

- 在线网站解密得到

- unctf{Ru5t_Rc4_1s_2_e@zy!!!}

### shelled_babyxor-重庆大学

```
考点`：`手动脱壳`，`数据处理`，`逆写解密
```

- 提示脱壳+算法逆向，有点东西

- 脱壳只能手动，没发现常规的pushad之类的东西，但知道是运行自解密

- x64dbg运行到解密位置，把程序dump出来拖进ida64

- ctrl+f12搜索unctf，发现有，定位过去，x交叉引用，定位关键代码

  ```asm
  FOOL:0000000000402E30 53                            push    rbx
  FOOL:0000000000402E31 48 83 EC 50                   sub     rsp, 50h
  FOOL:0000000000402E35 E8 06 EA FF FF                call    sub_401840
  FOOL:0000000000402E35
  FOOL:0000000000402E3A 48 8B 0D 1F 15 00 00          mov     rcx, cs:off_404360
  FOOL:0000000000402E41 48 8D 5C 24 20                lea     rbx, [rsp+58h+var_38]
  FOOL:0000000000402E46 48 8D 15 B3 11 00 00          lea     rdx, aInputYourAnswe            ; "input your answer\n"
  FOOL:0000000000402E4D E8 26 E9 FF FF                call    sub_401778
  FOOL:0000000000402E4D
  FOOL:0000000000402E52 48 8B 0D F7 14 00 00          mov     rcx, cs:off_404350
  FOOL:0000000000402E59 48 89 DA                      mov     rdx, rbx
  FOOL:0000000000402E5C E8 0F E9 FF FF                call    sub_401770
  FOOL:0000000000402E5C
  FOOL:0000000000402E61 48 89 D9                      mov     rcx, rbx
  FOOL:0000000000402E64 E8 07 E7 FF FF                call    sub_401570
  	;核心函数
  FOOL:0000000000402E64
  FOOL:0000000000402E69 85 C0                         test    eax, eax
  FOOL:0000000000402E6B 75 1B                         jnz     short loc_402E88
  FOOL:0000000000402E6B
  FOOL:0000000000402E6D 48 8B 0D EC 14 00 00          mov     rcx, cs:off_404360
  FOOL:0000000000402E74 48 8D 15 B4 11 00 00          lea     rdx, aSorryYouAreWro            ; "sorry you are wrong"
  FOOL:0000000000402E7B E8 F8 E8 FF FF                call    sub_401778
  FOOL:0000000000402E7B
  FOOL:0000000000402E80
  FOOL:0000000000402E80                               loc_402E80:                             ; CODE XREF: sub_402E30+6B↓j
  FOOL:0000000000402E80 31 C0                         xor     eax, eax
  FOOL:0000000000402E82 48 83 C4 50                   add     rsp, 50h
  FOOL:0000000000402E86 5B                            pop     rbx
  FOOL:0000000000402E87 C3                            retn
  FOOL:0000000000402E87
  FOOL:0000000000402E88                               ; ---------------------------------------------------------------------------
  FOOL:0000000000402E88
  FOOL:0000000000402E88                               loc_402E88:                             ; CODE XREF: sub_402E30+3B↑j
  FOOL:0000000000402E88 48 8B 0D D1 14 00 00          mov     rcx, cs:off_404360
  FOOL:0000000000402E8F 48 8D 15 7D 11 00 00          lea     rdx, aYouAreRightWel            ; "you are right!welcome to re"
  FOOL:0000000000402E96 E8 DD E8 FF FF                call    sub_401778
  ```

- 跟进函数发现只有sub_401570在大面积操作数据，F5跟进

  ```c++
  while ( 1 )
    {
      ++v2;
      v1 += v3 % 22;			//取余得到一个v1
      if ( v2 == &v7[5] )
        break;
      v3 = *v2;
    }
    v4 = 0i64;
    for ( i = 115; ; i = *&v7[4 * v4 + 5] )	//取数
    {
      if ( (v1 ^ *(a1 + v4)) != i )	//校对
        return 0;
      if ( ++v4 == 41 )
        break;
    }
  ```

- 终于找到真正的核心了，直接逆写解密，先求v1

  ```c++
  #include <bits/stdc++.h>
  using namespace std;
  int main()
  {
    int v1 = 0; 
    char v7[] = "unctfs";
    for(int i = 0; i < 5; i++){
      v1 += (((int)v7[i]) % 22);
    }
    cout <<v1 <<endl;			//38
    return 0;
  }
  ```

- 然后拿38跟这函数给出的一堆数据异或了一下，能搞出UNCTF，那就是答案没跑了，整理数据（痛苦面具），直接写解密

  ```python
  num = []
  for i in range(41):
      num.append(4*i+5)
  v9 = [0x65,0x72,0x60,0x5D,0x5F,0x49,0x53,0x79,0x4C,0x53,0x55,0x52, 0x79,0x53, 0x48,0x56,0x47,0x45, 0x4D,0x43, 0x42,0x79, 0x5F,0x49, 0x53,0x54, 0x79,0x40,0x4F,0x54,0x55,0x52,0x79,0x56, 0x54,0x49, 0x41,0x54]
  print(len(v9))
  for i in v9:
      print(chr(i^38),end='')
  print(chr(71^38))
  # CTF{you_just_unpacked_your_first_progra
  ```

- 然后，UN我之前验证了，没写，最后program}猜得出来，寄

## `Crypto`

### md5-1-西南科技大学

```
知识点`：`md5
```

- 像我这种大冤种直接造md5加密字典

```python
list = {}
import hashlib
for i in range(65,123):
    # 待加密信息
    str2 = chr(i)
    # 创建md5对象
    hl = hashlib.md5()
    hl.update(str2.encode(encoding='utf-8'))
    list[str2] = hl.hexdigest()
for i in range(0,10):
    str1 = str(i)
    # 创建md5对象
    hl = hashlib.md5()
    hl.update(str1.encode(encoding='utf-8'))
    list[str1] = hl.hexdigest()
l = dict(zip(list.values(),list.keys()))
li = ['4c614360da93c0a041b22e537de151eb','8d9c307cb7f3c4a32822a51922d1ceaa','0d61f8370cad1d412f80b84d143e1257','b9ece18c950afbfa6b0fdbfa4ff731d3','800618943025315f869e4e1f09471012','f95b70fdc3088560732a5ac135644506','e1671797c52e15f763380b45e841ec32','c9f0f895fb98ab9159f51fd0297e236d','a87ff679a2f3e71d9181a67b7542122c','8fa14cdd754f91cc6554c9e71929cce7','e1671797c52e15f763380b45e841ec32','8277e0910d750195b448797616e091ad','cfcd208495d565ef66e7dff9f98764da','c81e728d9d4c2f636f067f89cc14862c','c9f0f895fb98ab9159f51fd0297e236d','92eb5ffee6ae2fec3ad71c777531578f','45c48cce2e2d7fbdea1afc51c7c6ad26','cfcd208495d565ef66e7dff9f98764da','a87ff679a2f3e71d9181a67b7542122c','1679091c5a880faf6fb5e6087eb1b2dc','8fa14cdd754f91cc6554c9e71929cce7','4a8a08f09d37b73795649038408b5f33','cfcd208495d565ef66e7dff9f98764da','e1671797c52e15f763380b45e841ec32','c9f0f895fb98ab9159f51fd0297e236d','8fa14cdd754f91cc6554c9e71929cce7','cfcd208495d565ef66e7dff9f98764da','c9f0f895fb98ab9159f51fd0297e236d','cfcd208495d565ef66e7dff9f98764da','e1671797c52e15f763380b45e841ec32','45c48cce2e2d7fbdea1afc51c7c6ad26','1679091c5a880faf6fb5e6087eb1b2dc','e1671797c52e15f763380b45e841ec32','8f14e45fceea167a5a36dedd4bea2543','c81e728d9d4c2f636f067f89cc14862c','c4ca4238a0b923820dcc509a6f75849b','c9f0f895fb98ab9159f51fd0297e236d','a87ff679a2f3e71d9181a67b7542122c','cbb184dd8e05c9709e5dcaedaa0495cf']
for i in li:
    if l.get(i):
        print(l[i],end='')
    else:
        print('wrong',end='')
```

### md5-2-西南科技大学

```
知识点`：`md5`，`异或
```

- 多了个异或罢了，注意长度不够补位就行^_^

```python
# 生成表
import hashlib
list = {}
for i in range(65,123):
    # 待加密信息
    str2 = chr(i)
    # 创建md5对象
    hl = hashlib.md5()
    hl.update(str2.encode(encoding='utf-8'))
    list[str2] = hl.hexdigest()
for i in range(0,10):
    str1 = str(i)
    # 创建md5对象
    hl = hashlib.md5()
    hl.update(str1.encode(encoding='utf-8'))
    list[str1] = hl.hexdigest()
x = ['?','_','@','!','}','{','C','$','%',"^",'&','*','(',')']
for i in x:
    str2 = i
    # 创建md5对象
    hl = hashlib.md5()
    hl.update(str2.encode(encoding='utf-8'))
    list[str2] = hl.hexdigest()
l = dict(zip(list.values(),list.keys()))

# 检验
li = [0x4c614360da93c0a041b22e537de151eb,0xc1fd731c6d60040369908b4a5f309f41,0x80fdc84bbb5ed9e207a21d5436efdcfd,0xb48d19bb99a7e6bb448f63b75bc92384,0x39eaf918a52fcaa5ed9195e546b021c1,0x795d6869f32db43ff5b414de3c235514,0xf59a054403f933c842e9c3235c136367,0xc80b37816048952a3c0fc9780602a2fa,0x810ecef68e945c3fe7d6accba8b329bd,0xcad06891e0c769c7b02c228c8c2c8865,0x470a96d253a639193530a15487fea36f,0x470a96d253a639193530a15487fea36f,0x4bdea6676e5335f857fa8e47249fa1d8,0x810ecef68e945c3fe7d6accba8b329bd,0xedbb7ab78cde98a07b9b5a2ab284bf0a,0x44b43e07e9af05e3b9b129a287e5a8df,0xa641c08ed66b55c9bd541fe1b22ce5c0,0xabed1f675819a2c0f65c9b7da8cab301,0x738c486923803a1b59ef17329d70bbbd,0x7e209780adf2cd1212e793ae8796ed7c,0xa641c08ed66b55c9bd541fe1b22ce5c0,0xa641c08ed66b55c9bd541fe1b22ce5c0,0x636a84a33e1373324d64463eeb8e7614,0x6ec65b4ab061843b066cc2a2f16820d5,0xa4a39b59eb036a4a8922f7142f874114,0x8c34745bd5b5d42cb3efe381eeb88e4b,0x5b1ba76b1d36847d632203a75c4f74e2,0xd861570e7b9998dbafb38c4f35ba08bc,0x464b7d495dc6019fa4a709da29fc7952,0x8eb69528cd84b73d858be0947f97b7cc,0xdd6ac4c783a9059d11cb0910fc95d4a,0x4b6b0ee5d5f6b24e6898997d765c487c,0xb0762bc356c466d6b2b8f6396f2e041,0x8547287408e2d2d8f3834fc1b90c3be9,0x82947a7d007b9854fa62efb18c9fd91f,0x8ddafe43b36150de851c83d80bd22b0a,0xc7b36c5f23587e285e528527d1263c8b,0x2a0816e8af86e68825c9df0d63a28381,0x63ce72a42cf62e6d0fdc6c96df4687e3]
for cnt in range(1,len(li)):
    li[cnt] ^= li[cnt - 1]
for cnt in range(0, len(li)):
    li[cnt] = hex(li[cnt])
    li[cnt] = li[cnt][2:]
for i in li:
    if(len(i) != 32):
        i1 = '0' + i
    else:
        i1 = i
    if l.get(i1):
        print(l[i1],end='')
    else:
        print('false')
```

### dddd-西南科技大学

```
摩斯密码
```

- 摩斯密码01解密，这不写了吧

### caesar-西南科技大学

- 题目提示了base64表

- UNCTF校验发现是+45替换，逆写即可

- 脚本(坑点是得去掉末尾空格)

  ```python
  s=  'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
  w = 'B6vAy{dhd_AOiZ_KiMyLYLUa_JlL/HY_}'
  for i in w:
      if i in s:
          if(s.index(i) >= 45):
              print(s[s.index(i) - 45],end='')
          else:
              print(s[s.index(i)+45-64-26],end='')
      else:
          print(i,end='')
  ```

### ezxor-浙江师范大学

```
知识点`：`多次一密`，`数据还原
```

- 一个假题，劣质多次一密QAQ

- decode密文，伪造flag与密文异或得到残缺明文，寻找英文单词特征比对求解，1小时的快乐没了

  > UNCTF{Y0u_are_very_Clever!!!}

### Single table-西南科技大学

```
知识点`：`playfair
```

- 很明显变种Playfair加密

- 根据实例修改表

- B C D E F
  G H I K M
  N O Q R S
  T U V W X
  Z P L A Y

- 还原即可

  - 不同行不同列优先取后一个字母对应行元素
  - 同行取左
  - 同列取上

- 得到（注意下划线位置要换一下，哦对还有去掉x，我记得有人有群u吐槽这个下划线来着）

  > UNCTF{GOD_YOU_KNOW_PLAYFAIR}

### Multi table-西南科技大学

```
知识点`：`维吉尼亚密码（Vigenere）
```

#### 题意

- 这个题就是构造了一个循环序的字母字典table
- 还有一个乱序的匹配字母表base_table
- 然后随机生成查询key
- 加密flag
  - 首先查询base_table中字母位置
  - 替换为字典key键对应位置的值

#### 解题

- 首先用UNCT还原出key是多少，再逆写加密就可

- 脚本

  ```python
  # 字典表
  table = {0: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ', 1: 'BCDEFGHIJKLMNOPQRSTUVWXYZA', 2: 'CDEFGHIJKLMNOPQRSTUVWXYZAB', 3: 'DEFGHIJKLMNOPQRSTUVWXYZABC', 4: 'EFGHIJKLMNOPQRSTUVWXYZABCD', 5: 'FGHIJKLMNOPQRSTUVWXYZABCDE', 6: 'GHIJKLMNOPQRSTUVWXYZABCDEF', 7: 'HIJKLMNOPQRSTUVWXYZABCDEFG', 8: 'IJKLMNOPQRSTUVWXYZABCDEFGH', 9: 'JKLMNOPQRSTUVWXYZABCDEFGHI', 10: 'KLMNOPQRSTUVWXYZABCDEFGHIJ', 11: 'LMNOPQRSTUVWXYZABCDEFGHIJK', 12: 'MNOPQRSTUVWXYZABCDEFGHIJKL', 13: 'NOPQRSTUVWXYZABCDEFGHIJKLM', 14: 'OPQRSTUVWXYZABCDEFGHIJKLMN', 15: 'PQRSTUVWXYZABCDEFGHIJKLMNO', 16: 'QRSTUVWXYZABCDEFGHIJKLMNOP', 17: 'RSTUVWXYZABCDEFGHIJKLMNOPQ', 18: 'STUVWXYZABCDEFGHIJKLMNOPQR', 19: 'TUVWXYZABCDEFGHIJKLMNOPQRS', 20: 'UVWXYZABCDEFGHIJKLMNOPQRST', 21: 'VWXYZABCDEFGHIJKLMNOPQRSTU', 22: 'WXYZABCDEFGHIJKLMNOPQRSTUV', 23: 'XYZABCDEFGHIJKLMNOPQRSTUVW', 24: 'YZABCDEFGHIJKLMNOPQRSTUVWX', 25: 'ZABCDEFGHIJKLMNOPQRSTUVWXY'}
  # 乱序表
  base_table=['J', 'X', 'I', 'S', 'E', 'C', 'R', 'Z', 'L', 'U', 'K', 'Q', 'Y', 'F', 'N', 'V', 'T', 'P', 'O', 'G', 'A', 'H', 'D', 'W', 'M', 'B']
  # 密文flag
  flag = 'SDCGW{MPN_VHG_AXHU_GERA_SM_EZJNDBWN_UZHETD}'
  # 还原key
  print(base_table.index('U')) #0  9
  print(base_table.index('N')) #1  14
  print(base_table.index('C')) #2  5
  print(base_table.index('T')) #3  16
  for i in range(26):
      if table[i][9] == 'S':
          print('key0=' + str(i))
      if table[i][14] == 'D':
          print('key1=' + str(i))
      if table[i][5] == 'C':
          print('key2=' + str(i))
      if table[i][16] == 'G':
          print('key3=' + str(i))
  # 还原flag
  x = 0
  key=[9,15,23,16]
  for i in range(len(flag)):
      if flag[i] in base_table:
          print(base_table[table[key[x%4]].index(flag[i])],end='')
          x += 1
      else:
          print(flag[i],end='')
  # UNCTF{WOW_YOU_KNOW_THIS_IS_VIGENERE_CIPHER}
  ```

### easy_RSA-中国人民公安大学

```
知识点`:`rsa已知p高位攻击
```

- 看到>>200，是个RSA高位攻击，直接开搞

- 去网站[Sage Cell Server (sagemath.org)](https://sagecell.sagemath.org/) 先把roots爆出来

  ```python
  n=102089505560145732952560057865678579074090718982870849595040014068558983876754569662426938164259194050988665149701199828937293560615459891835879217321525050181965009152805251750575379985145711513607266950522285677715896102978770698240713690402491267904700928211276700602995935839857781256403655222855599880553
  p2 = 8183408885924573625481737168030555426876736448015512229437332241283388177166503450163622041857s
  p4 = p4 << 200
  PR.<x> = PolynomialRing(Zmod(n))
  f = x + p4
  roots = f.small_roots(X=2^200,beta=0.4)
  print(roots)
  print(p)
  ```

- 解码

  ```python
  import gmpy2
  from Crypto.Util.number import *
  roots = [358950849615278333731635244854025425463656033006805723630685]
  n = 102089505560145732952560057865678579074090718982870849595040014068558983876754569662426938164259194050988665149701199828937293560615459891835879217321525050181965009152805251750575379985145711513607266950522285677715896102978770698240713690402491267904700928211276700602995935839857781256403655222855599880553
  p2 = 13150231070519276795503757637337326535824298772055543325920447062237907554543786311611680606623830215547787830024125178567428699965091733811241451081695232
  p= p2 + roots[0]
  if n%p == 0:
      print(1)
  q = n//p
  phi = (p-1)*(q-1)
  e=0x10001
  c=6423951485971717307108570552094997465421668596714747882611104648100280293836248438862138501051894952826415798421772671979484920170142688929362334687355938148152419374972520025565722001651499172379146648678015238649772132040797315727334900549828142714418998609658177831830859143752082569051539601438562078140
  d = gmpy2.invert(e,phi)
  m = pow(c,d,n)
  print(long_to_bytes(m))
  ```

### Today_is_Thursday_V_me_50-海南大学

- 这个题就是纯考察逆向解密能力，大喜，啊不，大悲

- 对flag先加密1，再加密2，得到结果。逆写，还是逆写

- 脚本

  ```python
  import random
  from Crypto.Util.strxor import strxor
  from Crypto.Util.number import *
  # 题目信息
  key1 = b'Today_is_Thursday_V_me_50'
  key1_num = 530007872419584476649862008487908643412379763189507583587632
  x = 'Q\x19)T\x18\x1b(\x03\t^c\x08QiF>Py\x124DNg3P'
  #生成25位随机数序列
  random.seed(key1_num)
  ran = []
  for i in range(25):
      temp_num = random.randint(1,128)
      ran.append(temp_num)
  # 异或解密（encrypt2）
  flag = []
  for i in range(0,len(x)):
      temp = bytes_to_long(bytes(x[i], encoding="utf8"))
      flag.append(temp^ran[i])
  
  # 仿写encrypt1，得出最后的异或串
  guess=[('u', 'n', 'c', 't', 'f'), ('u', 'n', 'c', 'f', 't'), ('u', 'n', 't', 'c', 'f'), ('u', 'n', 't', 'f', 'c'), ('u', 'n', 'f', 'c', 't'), ('u', 'n', 'f', 't', 'c'), ('u', 'c', 'n', 't', 'f'), ('u', 'c', 'n', 'f', 't'), ('u', 'c', 't', 'n', 'f'), ('u', 'c', 't', 'f', 'n'), ('u', 'c', 'f', 'n', 't'), ('u', 'c', 'f', 't', 'n'), ('u', 't', 'n', 'c', 'f'), ('u', 't', 'n', 'f', 'c'), ('u', 't', 'c', 'n', 'f'), ('u', 't', 'c', 'f', 'n'), ('u', 't', 'f', 'n', 'c'), ('u', 't', 'f', 'c', 'n'), ('u', 'f', 'n', 'c', 't'), ('u', 'f', 'n', 't', 'c'), ('u', 'f', 'c', 'n', 't'), ('u', 'f', 'c', 't', 'n'), ('u', 'f', 't', 'n', 'c'), ('u', 'f', 't', 'c', 'n'), ('n', 'u', 'c', 't', 'f'), ('n', 'u', 'c', 'f', 't'), ('n', 'u', 't', 'c', 'f'), ('n', 'u', 't', 'f', 'c'), ('n', 'u', 'f', 'c', 't'), ('n', 'u', 'f', 't', 'c'), ('n', 'c', 'u', 't', 'f'), ('n', 'c', 'u', 'f', 't'), ('n', 'c', 't', 'u', 'f'), ('n', 'c', 't', 'f', 'u'), ('n', 'c', 'f', 'u', 't'), ('n', 'c', 'f', 't', 'u'), ('n', 't', 'u', 'c', 'f'), ('n', 't', 'u', 'f', 'c'), ('n', 't', 'c', 'u', 'f'), ('n', 't', 'c', 'f', 'u'), ('n', 't', 'f', 'u', 'c'), ('n', 't', 'f', 'c', 'u'), ('n', 'f', 'u', 'c', 't'), ('n', 'f', 'u', 't', 'c'), ('n', 'f', 'c', 'u', 't'), ('n', 'f', 'c', 't', 'u'), ('n', 'f', 't', 'u', 'c'), ('n', 'f', 't', 'c', 'u'), ('c', 'u', 'n', 't', 'f'), ('c', 'u', 'n', 'f', 't'), ('c', 'u', 't', 'n', 'f'), ('c', 'u', 't', 'f', 'n'), ('c', 'u', 'f', 'n', 't'), ('c', 'u', 'f', 't', 'n'), ('c', 'n', 'u', 't', 'f'), ('c', 'n', 'u', 'f', 't'), ('c', 'n', 't', 'u', 'f'), ('c', 'n', 't', 'f', 'u'), ('c', 'n', 'f', 'u', 't'), ('c', 'n', 'f', 't', 'u'), ('c', 't', 'u', 'n', 'f'), ('c', 't', 'u', 'f', 'n'), ('c', 't', 'n', 'u', 'f'), ('c', 't', 'n', 'f', 'u'), ('c', 't', 'f', 'u', 'n'), ('c', 't', 'f', 'n', 'u'), ('c', 'f', 'u', 'n', 't'), ('c', 'f', 'u', 't', 'n'), ('c', 'f', 'n', 'u', 't'), ('c', 'f', 'n', 't', 'u'), ('c', 'f', 't', 'u', 'n'), ('c', 'f', 't', 'n', 'u'), ('t', 'u', 'n', 'c', 'f'), ('t', 'u', 'n', 'f', 'c'), ('t', 'u', 'c', 'n', 'f'), ('t', 'u', 'c', 'f', 'n'), ('t', 'u', 'f', 'n', 'c'), ('t', 'u', 'f', 'c', 'n'), ('t', 'n', 'u', 'c', 'f'), ('t', 'n', 'u', 'f', 'c'), ('t', 'n', 'c', 'u', 'f'), ('t', 'n', 'c', 'f', 'u'), ('t', 'n', 'f', 'u', 'c'), ('t', 'n', 'f', 'c', 'u'), ('t', 'c', 'u', 'n', 'f'), ('t', 'c', 'u', 'f', 'n'), ('t', 'c', 'n', 'u', 'f'), ('t', 'c', 'n', 'f', 'u'), ('t', 'c', 'f', 'u', 'n'), ('t', 'c', 'f', 'n', 'u'), ('t', 'f', 'u', 'n', 'c'), ('t', 'f', 'u', 'c', 'n'), ('t', 'f', 'n', 'u', 'c'), ('t', 'f', 'n', 'c', 'u'), ('t', 'f', 'c', 'u', 'n'), ('t', 'f', 'c', 'n', 'u'), ('f', 'u', 'n', 'c', 't'), ('f', 'u', 'n', 't', 'c'), ('f', 'u', 'c', 'n', 't'), ('f', 'u', 'c', 't', 'n'), ('f', 'u', 't', 'n', 'c'), ('f', 'u', 't', 'c', 'n'), ('f', 'n', 'u', 'c', 't'), ('f', 'n', 'u', 't', 'c'), ('f', 'n', 'c', 'u', 't'), ('f', 'n', 'c', 't', 'u'), ('f', 'n', 't', 'u', 'c'), ('f', 'n', 't', 'c', 'u'), ('f', 'c', 'u', 'n', 't'), ('f', 'c', 'u', 't', 'n'), ('f', 'c', 'n', 'u', 't'), ('f', 'c', 'n', 't', 'u'), ('f', 'c', 't', 'u', 'n'), ('f', 'c', 't', 'n', 'u'), ('f', 't', 'u', 'n', 'c'), ('f', 't', 'u', 'c', 'n'), ('f', 't', 'n', 'u', 'c'), ('f', 't', 'n', 'c', 'u'), ('f', 't', 'c', 'u', 'n'), ('f', 't', 'c', 'n', 'u')]
  for i in range(4):
      what = guess.pop(50)
      name = ''.join(j for j in what)
      mask = strxor(5*name.encode(),key1)
  # 虽然计算了四轮，但最终用于使用的串只有最后一轮
  print(mask)
  n4 = '7\x1a\x02\x15\x17<\x1c\x15+:\x0b\x00\x14\x07\n\x02\x0c9"1\x0e\x109A^'
  for i in range(25):
      flag[i] ^= ord(n4[i])
  	print(chr(flag[i]),end='')
  # UNCTF{1_l0ve_Thurs4Ay!!!}
  ```

### ezRSA-广东海洋大学

```
知识点`：`z3求解`，`rsa基础流程
```

- 非常规rsa公私钥制作，反而变简单了

- 第一眼 p^4 = n，z3直接求解

  ```python
  from z3 import *
  p = Real('p')
  s = Solver()
  s.add(p**4 == 62927872600012424750752897921698090776534304875632744929068546073325488283530025400224435562694273281157865037525456502678901681910303434689364320018805568710613581859910858077737519009451023667409223317546843268613019139524821964086036781112269486089069810631981766346242114671167202613483097500263981460561)
  print(s.check())
  print(s.model())
  ```

- cdn正常解密即可

  ```python
  import libnum
  e = 65537
  n = 62927872600012424750752897921698090776534304875632744929068546073325488283530025400224435562694273281157865037525456502678901681910303434689364320018805568710613581859910858077737519009451023667409223317546843268613019139524821964086036781112269486089069810631981766346242114671167202613483097500263981460561
  c = 56959646997081238078544634686875547709710666590620774134883288258992627876759606112717080946141796037573409168410595417635905762691247827322319628226051756406843950023290877673732151483843276348210800329658896558968868729658727981445607937645264850938932045242425625625685274204668013600475330284378427177504
  p = 89065756791595323358603857939783936930073695697065732353414009005162022399741
  phi = p**4 - p**3
  d = libnum.invmod(e,phi)
  m = pow(c,d,n)
  print(m)
  print(libnum.n2s(m))
  # b'unctf{pneum0n0ultram01cr0sc0p01cs01l01c0v0lcan0c0n010s01s}'
  ```

### babyRSA-广东海洋大学

- 明文高位攻击，直接sage脚本

  ```python
  n = 25300208242652033869357280793502260197802939233346996226883788604545558438230715925485481688339916461848731740856670110424196191302689278983802917678262166845981990182434653654812540700781253868833088711482330886156960638711299829638134615325986782943291329606045839979194068955235982564452293191151071585886524229637518411736363501546694935414687215258794960353854781449161486836502248831218800242916663993123670693362478526606712579426928338181399677807135748947635964798646637084128123883297026488246883131504115767135194084734055003319452874635426942328780711915045004051281014237034453559205703278666394594859431
  c = 15389131311613415508844800295995106612022857692638905315980807050073537858857382728502142593301948048526944852089897832340601736781274204934578234672687680891154129252310634024554953799372265540740024915758647812906647109145094613323994058214703558717685930611371268247121960817195616837374076510986260112469914106674815925870074479182677673812235207989739299394932338770220225876070379594440075936962171457771508488819923640530653348409795232033076502186643651814610524674332768511598378284643889355772457510928898105838034556943949348749710675195450422905795881113409243269822988828033666560697512875266617885514107
  high_m = 11941439146252171444944646015445273361862078914338385912062672317789429687879409370001983412365416202240
  R.<x> = PolynomialRing(Zmod(n), implementation='NTL')
  m = high_m + x
  M = m((m^6 - c).small_roots()[0])
  print(hex(int(M))[2:])
  ```

- 然后把M转一下。。。。。。。

  ```python
  import libnum
  x =0x554e4354467b32376130616163372d373663622d343237642d393132392d3134373633363064356431627d
  print(libnum.n2s(x))
  # UNCTF{27a0aac7-76cb-427d-9129-1476360d5d1b}
  ```

### 今晚吃什么-金陵科技学院

```
知识点`：`培根密码
```

- 10000和00000串，5位，结合题目今晚吃什么，估计是培根密码

- 提取每个串的首位构成新串，写脚本替换为培根密码格式

  ```python
  lis = ['10000','10000','10000','00000','10000','00000','10000','10000','10000','10000','00000','10000','00000','00000','10000','10000','00000','00000','00000','10000','00000','10000','10000','10000','10000','10000','00000','00000','10000','00000','10000','00000','10000','10000','10000','00000','10000','10000','10000','00000','10000','10000','00000','10000','00000','00000','10000','10000','00000','00000','10000','00000','00000','10000','10000']
  for i in lis:
      if i == '10000':
          print('A',end='')
      else:
          print('B',end='')
      cnt += 1
      if cnt % 5 == 0:
          print('/',end='')
          
  # AAABA/BAAAA/BABBA/ABBBA/BAAAA/ABBAB/ABAAA/BAAAB/AABAB/BAABB/ABBAA
  ```

- 去网站解密得到flag

  > UNCTF{CRYPROISFUN}

### Fermat

- 费马小定理： 如果p是一个质数，而整数a不是p的倍数，则有a(p-1)≡1（mod p）

- 推导

  - gift+x = xp
  - gift = (p-1)*x
  - 2x*(p-1) ≡ 1(mod p)
  - 2x*(p-1) - 1 = x * p
  - 2gift -1 = x*p
  - 结合n = p*q
  - p = gcd(n, 2gift -1)
  - p = gcd(n, powmod(2, gift, n))

- 解密脚本

  ```python
  from Crypto.Util.number import *
  import gmpy2
  import libnum
  
  e = 0x10001
  n =  19793392713544070457027688479915778034777978273001720422783377164900114996244094242708846944654400975309197274029725271852278868848866055341793968628630614866044892220651519906766987523723167772766264471738575578352385622923984300236873960423976260016266837752686791744352546924090533029391012155478169775768669029210298020072732213084681874537570149819864200486326715202569620771301183541168920293383480995205295027880564610382830236168192045808503329671954996275913950214212865497595508488636836591923116671959919150665452149128370999053882832187730559499602328396445739728918488554797208524455601679374538090229259
  c = 388040015421654529602726530745444492795380886347450760542380535829893454552342509717706633524047462519852647123869277281803838546899812555054346458364202308821287717358321436303133564356740604738982100359999571338136343563820284214462840345638397346674622692956703291932399421179143390021606803873010804742453728454041597734468711112843307879361621434484986414368504648335684946420377995426633388307499467425060702337163601268480035415645840678848175121483351171989659915143104037610965403453400778398233728478485618134227607237718738847749796204570919757202087150892548180370435537346442018275672130416574430694059
  gift = 28493930909416220193248976348190268445371212704486248387964331415565449421099615661533797087163499951763570988748101165456730856835623237735728305577465527656655424601018192421625513978923509191087994899267887557104946667250073139087563975700714392158474439232535598303396614625803120915200062198119177012906806978497977522010955029535460948754300579519507100555238234886672451138350711195210839503633694262246536916073018376588368865238702811391960064511721322374269804663854748971378143510485102611920761475212154163275729116496865922237474172415758170527875090555223562882324599031402831107977696519982548567367160
  
  gift = pow(2,gift,n) - 1
  p = libnum.gcd(gift,n)
  q = n // p
  phi = (p-1)*(q-1)
  d = gmpy2.invert(e,phi)
  m = pow(c,d,n)
  print(long_to_bytes(m))
  # b'UNCTF{DO_y0u_Fermat_1ittle_theOrem}'
  ```

## `Misc`

### magic_word-西南科技大学

```
知识点`：`零宽隐写
```

- 提示明显，零宽隐写，文档cv到[Unicode Steganography with Zero-Width Characters (330k.github.io)](http://330k.github.io/misc_tools/unicode_steganography.html)
- 解密就完事了

> unctf{We1come_new_ctfer}

- 一开始没改大写WA了一发，血亏

### 巨鱼-河南理工大学

```
知识点`：`图片大小隐写`，`zip伪加密`，`文件合成`，`ppt隐写
```

- 拿到图片先常规检查，改了改宽高，发现多出一行字“无所谓我会出手”，可能有用

- 然后kali试着foremost了一下，发现图片隐藏了zip，提取出来打开

- 010查看zip，真加密，拿刚才找出来的字试了一下，密码正确

- 提取出来又是一层zip，还是有加密，010打开一眼看到总标志位0000，伪加密

- 翻看标志位unshortdeflag，为1的全改回0，去伪加密

- 一个flag文件夹，一个化学式和一个加密的pptx，网上查了化学式名字都不对，发现叫六六粉，试了666，正确，出题人真6

- 打开pptx，提示flagnothere，最后一页pptx无内容，猜测同色隐藏到背景里了，全选更换字体颜色，得到flag

  > UNCTF{y0u_F1nd_1t!}

- 这题多写几个字以表敬意（

### syslog-浙江师范大学

- 提示看压缩包，结合题目的syslog

- 上kali，binwalk -e文件，打开日志查看

- 搜索password发现base64

- ```
  cGFzc3dvcmQgaXMgVTZudTJfaTNfYjNTdA==
  ```

- 解密得到密码

- 输入得到flag

  > unctf{N1_sH3_D0n9_L0g_dE!}

### 找得到我吗-闽南师范大学

- 打开文档全选设定文字颜色，发现3k字大作文，但是好像没用，废话文学是玩明白了

- 常规检测，binwalk分离转出xml

- 一大堆东西挨个看

- document.xml中搜索flag得到结果

- > UNCTF{You_find_me!}

### 社什么社-湖南警察学院

- 凤凰古城不错，以后一定去看看

- > UNCTF{4F0198127A45F66C07A5B1A2DDA8223C}

### In_the_Morse_Garden-陆军工程大学

```
知识点`：`base64`，`word隐写`，`摩斯密码
```

- pdf打开，ctrl+A发现图片下面有东西，提取出来

- UNCTF{….=}，base64解密

- 花园宝宝乱入

- 只有玛卡巴卡和依古比古，加上题目，morse解密（空格是摩斯断点，因为这个卡了半天，服了）

- > UNCTF{WAN_AN_MAKA_BAKAAAAA!}