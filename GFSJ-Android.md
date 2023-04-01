---
title: GFSJ-Android
tags: Android
categories: Android
top: false
---


### app1

- jad直接反编译，MainActivity发现版本号与版本名的异或
- `水题`，`简单使用工具`

### easy-apk

- 变表base64
- `水题`

### app2

- 逻辑分析SecondActivity，从而找到加密so文件，确定AES加密，解密
- `so分析`，`AES`

### easy-so

- 同样从java代码找到so文件关键代码分析，两个简单换位加密
- `so分析`，`简单加密`

### easy-jni

- 毫无新意，同样so分析，变表base64加换位

### RememberOther

- 评价就是有病
- 直接jadx查看逻辑发现啥都不填return true，MD5解密+安卓就是flag

### 基础android

- 好题，java层简单加密，以及对广播的一个考察
- `简单逆向`，`Broadcast`，`页面跳转`，`文件流`

### app3

- ab文件–>安卓备份文件
- 先用[abe](https://github.com/nelenkov/android-backup-extractor)提取一下apk
- java -jar abe.jar unpack app3.ab app3.tar
- 破解数据库密码
- [sqlitebrowser](https://github.com/sqlitebrowser/sqlitebrowser) 打开携带的数据库找到flag
- `sqlite`、`安卓备份`

### Ph0en1x-100

- 同样是native层对flag的一个加密，解法很多
- 本质就是一个getflag函数外套了一个getSecret函数，IDA分析getSecret发现就是一个rot1

```python
if(this.getSecret(this.getFlag()).equals(this.getSecret(this.encrypt(sInput)))) {
          Toast.makeText(this, "Success", 1).show();
```

- 1：直接androidkiller篡改代码，打印出执行完getSecret(getflag())函数的返回值，然后进行回转得到flag
- 2：jeb下断点，动调获得返回值

```python
s = "ek`fz@q2^x/t^fn0mF^6/^rb`qanqntfg^E`hq|"
for i in s:
    print(chr(ord(i)+1),end='')
```

- `native`,`hook`

### APK逆向

- java层仿写加密即可，但是md5加密时候给我报错了：
  - Unhandled exception: java.security.NoSuchAlgorithmException
- 所以我直接jeb动调把加密后的串提取出来了
- 逻辑分析也可以，不难

```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;


public class Main {
    public static void main(String[] args) {
        checkSN("Tenshine");
    }
    private static String toHexString(byte[] bytes, String separator) {
        StringBuilder hexString = new StringBuilder();
        for (byte b : bytes) {
            String hex = Integer.toHexString(b & 255);
            if (hex.length() == 1) {
                hexString.append('0');
            }
            hexString.append(hex).append(separator);
        }
        return hexString.toString();
    }

    static void checkSN(String userName) {
        String hexstr = "b9c77224ff234f27ac6badf83b855c76";
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < hexstr.length(); i += 2) {
            sb.append(hexstr.charAt(i));
        }
        String userSN = sb.toString();
        System.out.println(userSN);
    }
}
```

- 难度一般

### 人民的名义-抓捕赵德汉1-200

- 🤡题，生成flag的md5结果裸奔，直接拿去解密即可
- 原意应该是利用hexkey对/ClassEnc文件进行aes解密，得到已经存在的CheckPass代码，然后生成passCheckObject对象

```python
import os
from Crypto.Cipher import AES
filename="ClassEnc"
key = "bb27630cf264f8567d185008c10c3f96"
key_bytes = bytes.fromhex(key)
aes = AES.new((key_bytes), AES.MODE_ECB)
data = bytearray(os.path.getsize(filename))
with open(filename, 'rb') as f:
	f.readinto(data)
	f.close()
decryption_data = aes.decrypt(data)
with open(filename+"_decryption.classs", 'ba') as f:
    f.write(decryption_data)
f.close()
```

### Android2.0

- native层的一个题，加密类似于栅栏密码之后三组异或，分别解开就行

```c++
//分组init
int __fastcall Init(char *a2, char *a3, const char *a4, int a5)
{
  int v5; // r5
  int v6; // r10
  int v7; // r6

  v5 = 0;
  v6 = 0;
  do
  {
    v7 = v5 % 3;
    if (v7== 2){
      a3[v5/3] = a4[v5];
    }
    else if (v7 == 1){
      a2[v5/3] = a4[v5];
    }
    else if (v7 == 0){
      *(v5/3) = a4[v5];
    }
    ++v5;
  }
  while(a5 != v5);
}
s1 = "fgorl"
s2 = "l{sra"
s3 = "asoy}"
for i in range(5):
	print(s1[i]+s2[i]+s3[i],end="")
```

### easy-dex

#### 整体流程

- 这个题蛮有意思，把dex文件放进了native层
- so文件拖进IDA找到android-main函数定位关键代码
- 之后分析解密dump出dex文件，放进原apk中重新识别
- jadx分析发现twofish加密，在线网站解密

#### 一些细节

- dex段的数据，可以看到从0x7004开始，长度为0x34c10。
- 至于dex的加密，他是设定了10秒摇手机90次，然后用dex文件段的数据分10组去和摇晃次数做一个异或，最后释放解密的dex文件，进行正常的程序运行，这里放一个网上的代码

```python
# 提取数据
fLib = open("libnative.so","rb");
fLib.seek(0x6004, 0);
read_buf = fLib.read(0x3CA10)
fDump = open("out.dump", "wb");
fDump.write(read_buf);
fDump.close();
fLib.close();
# 解密
import zlib
with open("./blog/out.dump", 'rb') as f:
    data = f.read()
    data_len = len(data)
    decode_bytes = [0] * data_len
# 分组异或
    for i in range(0, 9):
        per_seg_len = int(len(data) // 10)
        start = i * per_seg_len
        while start < (i + 1) * per_seg_len:
            decode_bytes[start] = data[start] ^ (i * 10 + 9)
            start += 1
# 末端
    per_seg_len = int(len(data) // 10)
    start = 9 * per_seg_len
    while start < data_len:
        decode_bytes[start] = data[start] ^ 89
        start += 1
# 解压写文件
with open("./blog/decode.dex", 'wb') as f:
    f.write(zlib.decompress(bytes(decode_bytes)))
java-->native`，`twofish加密`，`dumpdex
```

### boomshakalaka-3

- 这个题本身不难，但跟鬼打了一样
- 一个老版飞机大战游戏，jadx打开发现两个包plane和lib，lib明显是原本的游戏，plane是出题人加的
- 核心在于`SharedPreferences接口`的数据传输，查资料后了解到这玩意会在/data/data/package name/shared_prefs目录以xml格式接受数据
- 查看Cocos2dxPrefsFile.xml，结合游戏，发现是一个不断追加字符的过程，没在java层找到对应代码，那就去so
- so加载了老半天，发现一个score函数，里面就是不断加字符，然后结合题目要求最高分，所以按分数顺序把字符添上得到终极字符串
- 最后根据题目给出的传参提示串`YmF6aW5nYWFhYQ==`猜测base64，解码得到flag

```
native`,`cocos2d`,``SharedPreferences
```

### Illusion

- 发现关键调用checkflag，同时encflag已知，进入so分析

```java
String encflag = new BufferedReader(new InputStreamReader(MainActivity.this.getAssets().open("Flag"))).readLine();

CheckFlag(flag, encflag))
```

- 直接搜索checkflag函数，是个假函数，未经调用
- 搜索onload函数，找到动态加载的真正checkflag函数进行分析

```c++
int __fastcall sub_DC8(_JNIEnv *a1, int a2, int a3, int a4)
{
  size_t v4; // r0
  char v5; // r1
  size_t i; // [sp+28h] [bp-34h]
  char *v8; // [sp+30h] [bp-2Ch]
  char *v9; // [sp+34h] [bp-28h]
  char *v10; // [sp+38h] [bp-24h]

  v10 = j_j_Jstring2CStr(a1, a3);	//input flag
  v4 = j_strlen(v10);
  v9 = j_calloc(1u, v4 + 1);
  v8 = j_j_Jstring2CStr(a1, a4);	//encflag
  for ( i = 0; i < j_strlen(v10); ++i )
  {
    sub_10C0(v10[i] + aLjavaLangStrin_0[i] - 64, 93);
    v9[i] = v5 + 32;
  }
  if ( !j_strcmp(v9, v8) )
    return j__JNIEnv::NewStringUTF(a1, "Correct!");
  else
    return j__JNIEnv::NewStringUTF(a1, "Try Again!");
}
```

- 分析sub_10C0函数，第一层的if判断必为真，只执行第一个函数，函数目的为`a1%a2`，思路明了，写脚本
- 网友这个脚本还是有东西的

```python
enc_flag = "Ku@'G_V9v(yGS"
data = "(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;"
data = [ord(i) for i in data]

def main():
    enc = [ord(i) for i in enc_flag]
    flag = [0]*len(enc)
    for i in range(len(enc)):
        flag[i] = (enc[i]-32)+64-data[i]
        while flag[i] < 0x32:
            flag[i] += 93
        while flag[i] > 125:
            flag[i] -= 93
    print("".join([chr(i) for i in flag]))


if __name__ == '__main__':
    main()
# CISCN{GJ5728}
```