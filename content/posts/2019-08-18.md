---
title: 使用 openssl 进行 AES 加密
date: 2019-08-18T08:00:00+08:00
draft: false
toc:
comments: true
---


aes 加密需要一个字符串作为密钥，自己写一个，或者用完成的工具生成一随机的，假设密钥为 `passphrase` ，加密一个文件 file：

```
openssl enc -aes-128-cbc -in file -out file.enc -k passphrase
```

也可以把密钥存放在一个文本文件，然后用 `-kfile` 选项指定：

```
openssl enc -aes-128-cbc -in file -out file.enc -kfile passphrase.txt
```

