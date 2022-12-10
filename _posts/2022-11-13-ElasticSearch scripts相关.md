---
layout: post
title: "ElasticSearch scripts相关"
tags: ["elasticsearch","scripts","painless"]
---

elasticsearch支持使用脚本语言动态生成或者处理数据。

## 脚本语言支持历史
从目前查到的资料，elasticsearch支持的脚本发展历史大概为mvel->groovy->painless。
groovy在5.0版本时被启用，groovy很灵活，但是速度慢并且不够安全，历史上出现过多次漏洞，
最新的默认脚本语言是painless。painless 是专为elasticsearch开发的脚本语言，性能更优秀，
出现漏洞后es开发团队可以及时修复。同时es当前还支持expression和mustache两种脚本语言，具体可见
[链接](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)，
这里我们只讨论painless。

## 简单使用
最简单的方式

```json
{
  "script_fields": {
    "my_field_double": {
      "script": {
        "lang": "painless",
        "source": "doc['my_field'].value * params['power']",
        "params": {
          "power": 2
        }
      }
    }
  }
}
```
上面的查询，通过将已有的字段**my_field**乘以2生成一个新的字段**my_field_double**并返回。
查询包装在**script_fields**中，**script**包含了脚本的执行逻辑：
- lang： 脚本语言，上面提到最新的es支持三种：painless、expression和mustache，默认是painless。
- source： 脚本代码，例子中是painless的语法
- params： 参数，可以在script中提取params中的参数使用

script_fields看名字是可以包含多个脚本字段，事实也是如此，可以添加多个字段。

因为lang字段默认为painless，如果而参数可以直接指定在source中，所以有一种更简化的表示方式
```json
{
  "script_fields": {
    "my_field_double": {
      "script": "doc['my_field'].value * 2"
    }
  }
}
```
直接使用script字段编写painless脚本，如果查询包含双引号的话，可以使用"""script_code"""的语法，
即高版本java的字符块语法，用三个引号将字符串包起来，这样不用转义，可读性也更好。