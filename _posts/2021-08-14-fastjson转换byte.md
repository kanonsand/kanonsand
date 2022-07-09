---
layout: post
title: "fastjson对byte数组的处理"
tags: ["fastjson", "java"]
---

fastjson对byte数组的处理以及Base64编码的部分知识
## fastjson转换byte[]

将byte[]通过fastjson转换时，fastjson会自动对byte[]进行base64编码转为字符串，因此需要对结果进行base64解码恢复原始数据。

```java
java.util.Base64.getMimeDecoder.decode(String s);
```

特别注意要用getMimeDecoder而不是getDecoder。



### Base64

base64主要用于网络传输二进制，将不可读的二进制转换为可打印字符传输。具体就是将原始数据按照3字节长度进行分割，分割后的三字节数据用4个字节进行编码，最后不足三个字节的也会补全之后转换，所以结果为四字节的整数倍，因此转码后的结果比原始数据要大三分之一。

 这样原本每个字节8位，分割后每个字节6位，需要2^6也就是64个字符即可表示。最基本（basic）的base64使用a-z,A-Z,0-9再加上+和/两个符号这64个来编码，因为在url中+和/都有特殊含义，因此变种的url安全编码的base64使用-和_来代替原来的+和/



### MIME

MIME的全称是"Multi-purpose Internet Mail Extensions"，用于邮件中发送非asc编码的文字或者其他二进制类型的文件。因为最开始的邮件规范规定了邮件只能使用ascii编码，这导致只支持英文文本。