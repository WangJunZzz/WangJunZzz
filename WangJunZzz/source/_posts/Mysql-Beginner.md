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
    CAST(CURRENT_DATE() AS SIGNED)
    # 时间 00:08:15 -> 815
    CAST(CURRENT_TIME() AS SIGEND)
    # 时间戳 2019-08-08 00:08:15
    CAST(NOW() AS SIGNED)
    # 日期 2019-08-08 -> 2020
    YEAR(CURRENT_DATE())
    # 00:08:15 -> 15
    SECOND(CURRENT_DATE())
    # 时间转unix时间错
    UNIX_TIMESTAMP(NOW())
    # 字符串转时间
    CAST('2019-08-08' AS DATE)
    # 字符串转日期
    DATE_FROMAT(CURRENT_DATE(),'%Y-%m-%d')
    # 获取日期
    DAYNAME(CURRENT_DATE())
    MONTH(CURRENT_DATE())
```
- 聚合函数(常用于GROUP BY从句的SELECT查询中)
1. AVG(col)　　　　　　　　返回指定列的平均值
2. COUNT(col)　　　　　　　返回指定列中非NULL值的个数
3. MIN(col)　　　　　　　　返回指定列的最小值
4. MAX(col)　　　　　　　　返回指定列的最大值
5. SUM(col)　　　　　　　　返回指定列的所有值之和
6. GROUP_CONCAT(col)　    返回由属于一组的列值连接组合而成的结果

- 字符串函数
1. CONCAT(s1,s2...,sn)　　       将s1,s2...,sn连接成字符串
2. CONCAT_WS(sep,s1,s2...,sn)　　将s1,s2...,sn连接成字符串，并用sep字符间隔
3. LEFT(str,x)　　　　　　　　    返回字符串str中最左边的x个字符
4. RIGHT(str,x)　　　　　　　     返回字符串str中最右边的x个字符

- 控制流函数
1. CASE WHEN[test1] THEN [result1]...ELSE [default] END 如果testN是真，则返回resultN，否则返回default
2. CASE [test] WHEN[val1] THEN [result]...ELSE [default] END 如果test和valN相等，则返回resultN，否则返回default
3. IF(test,t,f) 如果test是真，返回t；否则返回 f
4. IFNULL(arg1,arg2) 如果arg1不是空，返回arg1，否则返回arg2
5. NULLIF(arg1,arg2) 如果arg1=arg2返回NULL；否则返回arg1