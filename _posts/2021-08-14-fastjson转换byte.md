---
layout: post
title: "fastjson对byte数组的处理"
tags: ["fastjson", "java"]
---

fastjson对byte数组的处理以及Base64编码的部分基础知识.

## fastjson转换byte[]

将byte[]通过fastjson转换时，fastjson会自动对byte[]进行base64编码转为字符串，因此需要对结果进行base64解码恢复原始数据。

```java
java.util.Base64.getMimeDecoder.decode(String s);
```

特别注意要用getMimeDecoder而不是getDecoder。

## fastjson @JSONField
fastjson可以使用@JSONField注解来修改json和实体类的字段映射，例如
```java
class DTO{
    @JSONField(name="src_ip_addr")
    private String srcIp;
}
```
可以将json
```json
{
  "src_ip_addr": "1.1.1.1"
}
```
转为对应的DTO，字段映射为srcIp。这个注解会同时影响序列化和反序列化，即我们通过toJSONString方法将实体类写出的json字段名会自动变为src_ip_addr。
有些情况下我们会同时序列化和反序列化，但是我们只想要在反序列化时使用该注解，例如我们从第三方接口获取了一个json，将其转为我们自己的实体类，之后又需要
把这个实体类转成json发送出去，此时我们不需要再对字段进行映射，但是使用fastjson会自动转换，此时可以如下处理：
> 1、另辟蹊径，序列化时使用其他的库（gson或jackson），避开fastjson的处理
> 2、修改注解位置。实际上@JSONField时可以加在方法上的，而fastjson序列化和反序列化时访问的就是字段的get和set方法，如果只想反序列化是处理，可以
 如下
 ```java
class DTO{
    private String srcIp;

    @JSONField(name="src_ip_addr")
    public void setSrcIp(String srcIp) {
        this.srcIp = srcIp;
    }

    public String getSrcIp() {
       return srcIp; 
    }
}
```
这样序列化时字段名还是srcIp。


### Base64

base64主要用于网络传输二进制，将不可读的二进制转换为可打印字符传输。具体就是将原始数据按照3字节长度进行分割，分割后的三字节数据用4个字节进行编码，最后不足三个字节的也会补全之后转换，所以结果为四字节的整数倍，因此转码后的结果比原始数据要大三分之一。

 这样原本每个字节8位，分割后每个字节6位，需要2^6也就是64个字符即可表示。最基本（basic）的base64使用a-z,A-Z,0-9再加上+和/两个符号这64个来编码，因为在url中+和/都有特殊含义，因此变种的url安全编码的base64使用-和_来代替原来的+和/

java中编码解码方法在java.util.Base64中
```java
        String originalInput = "test input";
        String encodedString = Base64.getEncoder().encodeToString(originalInput.getBytes());

        byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
        String decodedString = new String(decodedBytes);
```


### MIME

MIME的全称是"Multi-purpose Internet Mail Extensions"，用于邮件中发送非asc编码的文字或者其他二进制类型的文件。因为最开始的邮件规范规定了邮件只能使用ascii编码，这导致只支持英文文本。

通常情况下，Base64输出一个不包含换行符的字符串，如果使用mime编码，输出结果会确保每行不超过76个字符，即没76个字符就会添加一个换行(\r\n)，java中mime编码方法如下

```java
StringBuilder buffer = getMimeBuffer();
byte[] encodedAsBytes = buffer.toString().getBytes();
String encodedMime = Base64.getMimeEncoder().encodeToString(encodedAsBytes);

byte[] decodedBytes = Base64.getMimeDecoder().decode(encodedMime);
String decodedMime = new String(decodedBytes);
```