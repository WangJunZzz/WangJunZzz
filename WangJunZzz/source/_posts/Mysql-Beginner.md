---
title: Mysql-Beginner
date: 2019-07-29 00:15:04
tags:
- Mysql
categories:
- Mysql
---
# 函数
- 字符转日期
``` sql
    # 日期2019-08-08 -> 20190808  
    CASE(CURRENT_DATE() AS SIGNED)
    # 时间 00:08:15 -> 815
    CASE(CURRENT_TIME() AS SIGEND)
    # 时间戳 2019-08-08 00:08:15
    CASE(NOW() AS SIGNED)
    # 日期 2019-08-08 -> 2020
    YEAR(CURRENT_DATE())
    # 00:08:15 -> 15
    SECOND(CURRENT_DATE())
```
