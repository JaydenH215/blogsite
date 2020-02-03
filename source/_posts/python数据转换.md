---
title: python数据转换
date: 2019-12-10 13:09:01
tags:
- python

categories:
- 术业专攻

---

整理一下写python小工具的数据转换常用方法。
<!-- more --> 

前言
===

优先使用py标准库的方法

目录
===

<!-- TOC -->

- [1. Built-in Types](#1-built-in-types)
    - [1.1. int](#11-int)
        - [1.1.1. int.to_bytes(length, byteorder, *, signed=False)](#111-intto_byteslength-byteorder--signedfalse)
        - [1.1.2. int.from_bytes(bytes, byteorder, *, signed=False)](#112-intfrom_bytesbytes-byteorder--signedfalse)
    - [1.2. str](#12-str)
        - [1.2.1. str.encode(encoding="utf-8", errors="strict")](#121-strencodeencodingutf-8-errorsstrict)
        - [1.2.2. str.isascii()](#122-strisascii)
        - [1.2.3. str.isdecimal()](#123-strisdecimal)
    - [1.3. bytes](#13-bytes)
        - [1.3.1. bytes.fromhex(string)](#131-bytesfromhexstring)
        - [1.3.2. hex()](#132-hex)
        - [1.3.3. bytes.decode(encoding="utf-8", errors="strict")](#133-bytesdecodeencodingutf-8-errorsstrict)
        - [1.3.4. bytes.isascii()](#134-bytesisascii)
- [2. Other Std Libs](#2-other-std-libs)
    - [2.1. struct](#21-struct)
        - [2.1.1. format](#211-format)
        - [2.1.2. struct.pack(format, v1, v2, ...)](#212-structpackformat-v1-v2-)
        - [2.1.3. struct.pack_into(format, buffer, offset, v1, v2, ...)](#213-structpack_intoformat-buffer-offset-v1-v2-)
        - [2.1.4. struct.unpack(format, buffer)](#214-structunpackformat-buffer)
        - [2.1.5. struct.unpack_from(format, buffer, offset=0)](#215-structunpack_fromformat-buffer-offset0)
    - [2.2. binascii](#22-binascii)
        - [2.2.1. binascii.hexlify(data[, sep[, bytes_per_sep=1]])](#221-binasciihexlifydata-sep-bytes_per_sep1)
        - [2.2.2. binascii.unhexlify(hexstr)](#222-binasciiunhexlifyhexstr)
- [3. 参考资料](#3-参考资料)

<!-- /TOC -->

# 1. Built-in Types

## 1.1. int

### 1.1.1. int.to_bytes(length, byteorder, *, signed=False)

Return an array of bytes representing an integer.

```python
>>> (1024).to_bytes(2, byteorder='big')
b'\x04\x00'
>>> (1024).to_bytes(10, byteorder='big')
b'\x00\x00\x00\x00\x00\x00\x00\x00\x04\x00'
>>> (-1024).to_bytes(10, byteorder='big', signed=True)
b'\xff\xff\xff\xff\xff\xff\xff\xff\xfc\x00'
>>> x = 1000
>>> x.to_bytes((x.bit_length() + 7) // 8, byteorder='little')
b'\xe8\x03'
```
### 1.1.2. int.from_bytes(bytes, byteorder, *, signed=False)

Return the integer represented by the given array of bytes.

```python
>>> int.from_bytes(b'\x00\x10', byteorder='big')
16
>>> int.from_bytes(b'\x00\x10', byteorder='little')
4096
>>> int.from_bytes(b'\xfc\x00', byteorder='big', signed=True)
-1024
>>> int.from_bytes(b'\xfc\x00', byteorder='big', signed=False)
64512
>>> int.from_bytes([255, 0, 0], byteorder='big')
16711680
```

## 1.2. str

### 1.2.1. str.encode(encoding="utf-8", errors="strict")

Return an encoded version of the string as a bytes object.

```python
>>> 'w435g'.encode()
b'w435g'
>>> type('w435g'.encode())
<class 'bytes'>
```

### 1.2.2. str.isascii()

Return True if the string is empty or all characters in the string are ASCII, False otherwise. ASCII characters have code points in the range U+0000-U+007F.

### 1.2.3. str.isdecimal()
Return True if all characters in the string are decimal characters and there is at least one character, False otherwise. Decimal characters are those that can be used to form numbers in base 10, e.g. U+0660, ARABIC-INDIC DIGIT ZERO. Formally a decimal character is a character in the Unicode General Category “Nd”.

## 1.3. bytes

### 1.3.1. bytes.fromhex(string)

This bytes class method returns a bytes object, decoding the given string object. The string must contain two hexadecimal digits per byte, with ASCII whitespace being ignored.

```python
>>> bytes.fromhex('2Ef0 F1f2  ')
b'.\xf0\xf1\xf2'

```

### 1.3.2. hex()

Return a string object containing two hexadecimal digits for each byte in the instance.

```python
>>> b'\xf0\xf1\xf2'.hex()
'f0f1f2'
```

If you want to make the hex string easier to read, you can specify a single character separator sep parameter to include in the output. By default between each byte. A second optional bytes_per_sep parameter controls the spacing. Positive values calculate the separator position from the right, negative values from the left.

```python
>>> value = b'\xf0\xf1\xf2'
>>> value.hex('-')
'f0-f1-f2'
>>> value.hex('_', 2)
'f0_f1f2'
>>> b'UUDDLRLRAB'.hex(' ', -4)
'55554444 4c524c52 4142'
```

### 1.3.3. bytes.decode(encoding="utf-8", errors="strict")

Return a string decoded from the given bytes.

```python
>>> value = b'\xf0\xf1\xf2'
>>> value.decode()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf0 in position 0: invalid continuation byte

>>> value = b'\x30\x31\x32'
>>> value.decode()
'012'
```

### 1.3.4. bytes.isascii()

Return True if the sequence is empty or all bytes in the sequence are ASCII, False otherwise. ASCII bytes are in the range 0-0x7F.


# 2. Other Std Libs

## 2.1. struct

[标准库介绍链接](https://docs.python.org/3/library/struct.html)

This module performs conversions between Python values and C structs represented as Python bytes objects. This can be used in handling binary data stored in files or from network connections, among other sources. 

### 2.1.1. format

| **Format** | **C Type**  | **Python Type** | **Standard size** |
| :-- | :--: | :--: | :--: | :-- |
| x | pad byte  | no value |  |
| c | char      | bytes of length 1 | 1 |
| b | signed char | integer | 1 |
| B | unsigned char | integer | 1 |
| h | short | integer | 2 |
| H | unsigned short | integer | 2 |
| i | int | integer | 4 |
| I | unsigned int | integer | 4 |
| s | char [] | bytes

### 2.1.2. struct.pack(format, v1, v2, ...)

Return a bytes object containing the values v1, v2, … packed according to the format string format. The arguments must match the values required by the format exactly.

### 2.1.3. struct.pack_into(format, buffer, offset, v1, v2, ...)

Pack the values v1, v2, … according to the format string format and write the packed bytes into the writable buffer buffer starting at position offset. Note that offset is a required argument.

### 2.1.4. struct.unpack(format, buffer)

Unpack from the buffer buffer (presumably packed by pack(format, ...)) according to the format string format. The result is a tuple even if it contains exactly one item. The buffer’s size in bytes must match the size required by the format, as reflected by calcsize().

### 2.1.5. struct.unpack_from(format, buffer, offset=0)

Unpack from buffer starting at position offset, according to the format string format. The result is a tuple even if it contains exactly one item. The buffer’s size in bytes, starting at position offset, must be at least the size required by the format, as reflected by calcsize().


## 2.2. binascii

### 2.2.1. binascii.hexlify(data[, sep[, bytes_per_sep=1]])
Return the hexadecimal representation of the binary data. Every byte of data is converted into the corresponding 2-digit hex representation. The returned bytes object is therefore twice as long as the length of data.

Similar functionality (but returning a text string) is also conveniently accessible using the bytes.hex() method.

If sep is specified, it must be a single character str or bytes object. It will be inserted in the output after every bytes_per_sep input bytes. Separator placement is counted from the right end of the output by default, if you wish to count from the left, supply a negative bytes_per_sep value.

```python
>>> import binascii
>>> binascii.hexlify(b'\xb9\x01\xef', '-')
b'b9-01-ef'
```

### 2.2.2. binascii.unhexlify(hexstr)
Return the binary data represented by the hexadecimal string hexstr. This function is the inverse of hexlify(). hexstr must contain an even number of hexadecimal digits (which can be upper or lower case), otherwise an Error exception is raised.

Similar functionality (accepting only text string arguments, but more liberal towards whitespace) is also accessible using the bytes.fromhex() class method.

```python
>>> binascii.unhexlify(b'3094')
b'0\x94'
```

# 3. 参考资料
- [Python 3中bytes/string的区别](https://www.cnblogs.com/abclife/p/7445222.html)
- [Python 3官方文档](https://docs.python.org/3/index.html)
- [python3 byte,int,str转换](https://www.cnblogs.com/DirWang/p/11426517.html)
- [python 字符串，bytes和hex字符串之间的相互转换](https://www.cnblogs.com/LoveMusicGuy/p/10616430.html)