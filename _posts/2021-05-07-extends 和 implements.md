---
layout: post
title: "Java中extends和implements"
tags: ["java"]
---
简单比较java中extends和implements关键字的使用场景及限制。

## extends 和 implements

| extends                                                      | implements                             |
| ------------------------------------------------------------ | -------------------------------------- |
| 通过extends，类可以继承另外一个类，接口可以继承其他接口（可以多个） | 通过implements关键字类可以实现一个接口 |
| 子类不强制重写父类的所有方法                                 | 类实现接口时强制实现接口中的所有方法   |
| 一个类只能extends一个父类                                    | 一个类可以implements任意数量接口       |
| 一个接口可以extends任意数量接口                              | 接口无法implements另外的接口           |

