---
layout: post
title: "java重载方法匹配优先级"
tags: ["语法", "java"]
---

java重载方法匹配优先级.


## the priority in java overload is :  
**exact match** > **widening** > **boxing/unboxing** > **varargs**

## some rule of overloading
- method signature
> The method signature include ***the type of arguements*** , ***the number of arguements*** , ***the order of arguements(when their types are different)***  
> Overload in java is to change its signature.
- 