---
layout: wiki
title: BASE 64 编码
categories: encode
description: BASE 64 编码
keywords: encode, base64
---

## Base 64 编码

- 说明

Base 64 编码是使用 64 个可打印 ASCII 字符「A-Z、a-z、0-9、+、/」将任意字节序列数据编码成ASCII字符串，另有`=`符号用作后缀用途。

Base 64 将输入字符串按字节切分，取得每个字节对应的二进制值「若不足 8 比特则高位补 0」，然后将这些二进制数值串联起来，再按照 6 比特一组进行切分「因为 2^6=64」，最后一组若不足 6 比特则末尾补 0 。将每组二进制值转换成十进制，然后在上述表格中找到对应的符号并串联起来就是 Base 64编码结果。

由于二进制数据是按照 8 比特一组进行传输，因此 Base 64 按照 6 比特一组切分的二进制数据必须是 24 比特的倍数「6 和 8 的最小公倍数」。24 比特就是 3 个字节，若原字节序列数据长度不是 3 的倍数时且剩下 1 个输入数据，则在编码结果后加 2 个 `=` ；若剩下2个输入数据，则在编码结果后加 1 个 `=` 。

Base 64 的编码表是由「A-Z、a-z、0-9、+、/」64 个可见字符构成，`=` 符号用作后缀填充。

- 位数

4 的倍数

- 特征

「大写字母」「小写字母」「阿拉伯数字」「`+`」「`/`」「末位 0 / 1 / 2 个 `=` 」

- 编码表

| 数值 | 字符 | 数值 | 字符 | 数值 | 字符 | 数值 | 字符 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0    | A    | 16   | Q    | 32   | g    | 48   | w    |
| 1    | B    | 17   | R    | 33   | h    | 49   | x    |
| 2    | C    | 18   | S    | 34   | i    | 50   | y    |
| 3    | D    | 19   | T    | 35   | j    | 51   | z    |
| 4    | E    | 20   | U    | 36   | k    | 52   | 0    |
| 5    | F    | 21   | V    | 37   | l    | 53   | 1    |
| 6    | G    | 22   | W    | 38   | m    | 54   | 2    |
| 7    | H    | 23   | X    | 39   | n    | 55   | 3    |
| 8    | I    | 24   | Y    | 40   | o    | 56   | 4    |
| 9    | J    | 25   | Z    | 41   | p    | 57   | 5    |
| 10   | K    | 26   | a    | 42   | q    | 58   | 6    |
| 11   | L    | 27   | b    | 43   | r    | 59   | 7    |
| 12   | M    | 28   | c    | 44   | s    | 60   | 8    |
| 13   | N    | 29   | d    | 45   | t    | 61   | 9    |
| 14   | O    | 30   | e    | 46   | u    | 62   | +    |
| 15   | P    | 31   | f    | 47   | v    | 63   | /    |





-   过程

1.  查询ASCII码，变换位8位二进制
2.  变换为6位二进制，用0补全24「6和8最小公倍数」倍数位。
3.  对照 Base 64 编码，前面补了N个8位二进制0，最后结尾增加N个`=`

-   举例「含过程」

1.  编码内容为：`12345`

2.  查找ASCII码：

    ​	 `1`的ASCII码`49`，二进制为`0011 0001`

    ​	 `2`的ASCII码`50`，二进制为`0011 0010`

    ​	 `3`的ASCII码`51`，二进制为`0011 0011`

    ​	 `4`的ASCII码`52`，二进制为`0011 0100`

    ​	 `5`的ASCII码`53`，二进制为`0011 0101`

3. 二进制分组，因为 5 * 8 = 40 ，不是 24 「 6 和 8 的最小公倍数」的倍数，所以需要补 1 个 8 位二进制 0 到末位。

    ​	原「8位一组二进制」：00110001 00110010 00110011 00110100 00110101

    ​	补「补止24的倍数位」：00110001 00110010 00110011 00110100 00110101 00000000

    ​	分「分为6位一组」：001100 010011 001000 110011 001101 000011 010100 000000
    
    ​	转「二进制转换十进制」：12 19 8 51 13 3 20 0
    
    ​	替「根据补位数量替换`=`」：12 19 8 51 13 3 20 =
    
    ​	编「根据Base 64编码表编码」：MTIzNDU=
    
4. 结果

    ​	Base 64 ( 12345 ) = `MTIzNDU=`