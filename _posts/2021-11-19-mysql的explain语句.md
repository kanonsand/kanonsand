---
layout: post
title: "mysql的explain语句"
tags: ["linux", "mysql"]
---

explain语句用于分析mysql的执行计划，帮助优化和排查慢查询。


# mysql select和select XXX lock for update结果可能不同
select语句不会阻塞，未提交的事务结果不会返回，而select for update会等待所有写事务提交后返回最新的数据
