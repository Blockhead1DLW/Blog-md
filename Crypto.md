---
title: Crypyo
tags: Crypto
categories: Crypto
top: false
---

## Base家族

#### base16

- 16进制，只包含（[0-9], [A-F]）

#### base32

- （[A-Z],[2-7]）,一般来讲密文字母多于数字，而且末尾有多个等号

#### base58

- 结尾无等号

#### base64

- （[A-Z],[a-z],[0-9]），末尾有1到2个等号
- 8位变6位重组

#### base85

- 等号可能出现在密文中间

#### base91

- （[0-9]，[a-z]，[A-Z],[!#$%&()*+,./:;<=>?@[]^_`{|}~”]）

#### base100

- 密文为emjio

## 幂数加密

### 加密方法

- 由于每一个数都可以变成形式为”2a+2b+2c….”的表达式
  所以我们用对应”abc…”数字串来代表这个数
- 例如
  5 = 20+22，即可表示为02
  19 = 20+21+24，即014

### 题目

> 8842101220480224404014224202480122

- 是8位大写字母
- 采用幂数加密

### 题解

- 联想8位字母和8个0的字符串，猜测以0为分隔符
  拆分后可以得到：

  > 88421 122 48 2244 4 142242 248 122

- 观察发现每个数字都是2的倍数，推断单个数字表示的就是”2a“，所以将每一段数字直接求和

- 上脚本：

  ```
  #include <iostream>
  using namespace std;
  int main(void)
  {
  	string s = "88421 122 48 2244 4 142242 248 122 "; 
  	int sum = 0;
  	for(int i = 0; i < s.length(); i++){
  		if(s[i] == ' '){
  			cout <<char(sum+'A'-1);
  			sum = 0;
  		}
  		else{
  			sum += (s[i]-'0');
  		}
  		
  	}
  	return 0;
  }
  ```

- 得到flag：

  > cyberpace{WELLDONE}

## 猪圈密码

- 特征：

  ![Crypto_Pig.jpg](https://s2.loli.net/2022/11/10/NcQI8HopdEwVzy9.jpg)

- 解密网站：

  - [猪圈密码-在线解密加密工具 (xiao84.com)](https://xiao84.com/tools/103177.html)

## emjio-AES

- 加密过程就是将一段字符串用key和rotation进行AES加密，然后将输出的base64码用表情包替换，特征只能靠猜和hint了
- 解密网站：
  - [emoji-aes (aghorler.github.io)](https://aghorler.github.io/emoji-aes/)
- 涉及题目
  - ISCTF2022 可爱的emjio

## 佛曰加密

- 特征：“諸隸毘僧降毘吽諸陀毘摩毘隸僧毘缽薩毘願心毘婆毘嚴彌毘念毘如毘囑毘” 一堆古怪文字
- 解密网站：
  - [新约佛论禅/佛曰加密 - PcMoe!](http://hi.pcmoe.net/Buddha.html)

## 天干地支加密

- 特征：利用纪年法对应加密数字
- 解密：查表
- 涉及题目：
  - [MRCTF]天干地支+甲子

## 社会主义核心价值观加密

- 特征：顾名思义
- 解密网站：
  - [社会主义核心价值观加密/解密 - 一个工具箱 - 好用的在线工具都在这里！ (atoolbox.net)](http://www.atoolbox.net/Tool.php?Id=850)

## jsfuck编码

- 特征：由()+ []! 六个字符组成编码

- 举例：

  ```
  false	==>	![]
  true	=>	!![]
  undefined	=>	[][[]]
  NaN	=>	+[![]]
  0	=>	+[]
  1	=>	+!+[]
  2	=>	!+[]+!+[]
  10	=>	[+!+[]]+[+[]]
  Array	=>	[]
  Number	=>	+[]
  String	=>	[]+[]
  Boolean	=>	![]
  Function	=>	[]["filter"]
  eval	=>	[]["filter"]["constructor"]( CODE )()
  window	=>	[]["filter"]["constructor"]("return this")()
  ```

- 解密网站：

  - https://www.bugku.com/tools/jsfuck
  - [jsfuck加密/解密 - dejs.vip](https://www.dejs.vip/encry_decry/jsfuck.html)

- 涉及题目：

  - buuctf - 这是什么