---
title: Python基操-datetime
layout: post
tags: Python基操
categories: 'Python'
---

Python 中的datetime模块，时间获取、时间转换等常用的用法。

### 第一部分 总体概览

datetime是一个关于时间的库，主要包含的类有：

| 类名          | 功能说明                                                   |
| :------------ | :--------------------------------------------------------- |
| date          | 日期对象,常用的属性有year, month, day                      |
| time          | 时间对象                                                   |
| datetime      | 日期时间对象,常用的属性有hour, minute, second, microsecond |
| datetime_CAPI | 日期时间对象C语言接口                                      |
| timedelta     | 时间间隔，即两个时间点之间的长度                           |
| tzinfo        | 时区信息对象                                               |

### 第二部分 模块常用

### 1. date模块
```python
# 获取当前日期 ，返回值是时间特有的date格式，里面包括了year，month和day,通过点号获取
import datetime
>>> tyday = datetime.date.today()
>>> tyday
datetime.date(2022, 1, 23)

>>> tyday.year
2022

# 日期转换为字符串,这里是通过转换为符合ISO 8601标准 (YYYY-MM-DD) 的日期字符串
>>> tyday.isoformat()
'2022-01-23'

# 拆分日期,方便取用
>>> spday = tyday.isocalendar()
(2022, 3, 7)
>>> spday[0]
2022
```

```python
# 构造需要的时间 [直接输入]
>>> wantday = datetime.date(2017,3,22)
>>> wantday
datetime.date(2017, 3, 22)

# 构造需要的时间 [从时间戳构建]
>>> import time
>>> time.time()
1642932611.3631077
>>> datetime.date.fromtimestamp(time.time())
datetime.date(2022, 1, 23)


# 判断一周的第几天，isoweekday(...): 返回符合ISO标准的指定日期所在的星期数（周一为1，周二为2，....,周日为7) 
>>> wantday.isoweekday()
3
```
实例
```python
now = datetime.date( 2010 , 4 , 6 ) 
tomorrow = now.replace(day = 7 ) 
print  ('now:' , now, ', tomorrow:' , tomorrow )
print  ('isoweekday():' , now.isoweekday() )
#  拆散时间，转换为元祖形式，方便取用
print  ('isocalendar():' , now.isocalendar() )
print  ('isoformat():' , now.isoformat())

# # ---- 结果 ----  
# now: 2010-04-06 , tomorrow: 2010-04-07  
# weekday(): 1  
# isocalendar(): (2010, 14, 2)  
# isoformat(): 2010-04-06
```

### 2. time模块

这个模块个人觉得过于鸡肋，因为`datetime.datetime`包含了里面的内容，所以不总结了，下一个

```sh
# hour的范围为[0, 24)，
# minute的范围为[0, 60)，
# second的范围为[0, 60)，
# microsecond的范围为[0, 1000000)
```

### 3. datatime模块

```python
# 获取当前时间（本地），这个时间包含： 年、月、日、时、分、秒、微秒 共7个量
# 分别对应取用 year、month、day、hour、minute、second、microsecond
>>> from datetime import datetime
>>> now = datetime.now()
>>> now
datetime.datetime(2022, 1, 23, 18, 35, 57, 666123)

# 获取协调世界时,即世界标准时(取代格林威治时间)
>>> datetime.datetime.now()
datetime.datetime(2022, 1, 23, 18, 57, 54, 313232)
>>> datetime.datetime.utcnow()
datetime.datetime(2022, 1, 23, 10, 57, 54, 313232)
```

```python
# 构造需要的时间 [直接输入]
>>> datetime.datetime(2017, 3, 22, 16, 55, 49, 148233)
datetime.datetime(2017, 3, 22, 16, 55, 49, 148233)

# 构造需要的时间 [从时间戳构建，获取时间标准时间；本地时间直接获取就行了]
>>> datetime.datetime.utcfromtimestamp(time.time())
datetime.datetime(2017, 3, 22, 8, 29, 7, 654272)

# 构造需要的时间 [从字符串构建]，     记忆：这个中间是p，strptime，str product time
# 这里需要指定两个参量：  时间字符串、字符串格式
>>> datetime.datetime.strptime('2017-3-22 15:25','%Y-%m-%d %H:%M')
datetime.datetime(2017, 3, 22, 15, 25)
```

```python
# 格式化日期，字符串输出，记忆：这个中间是f，strftime， str from time
>>> td = datetime.datetime.now()
>>> print(td.strftime("%Y-%m-%d"))
2022-01-23

# 格式化时间,字符串输出
>>> td = datetime.datetime.now()
>>> print(td.strftime("%H:%M:%S"))
19:09:48
        
```

```python
datetime.min、datetime.max：datetime所能表示的最小值与最大值；
datetime.resolution：datetime最小单位；
datetime.today()：返回一个表示当前本地时间的datetime对象；
datetime.now([tz])：返回一个表示当前本地时间的datetime对象，如果提供了参数tz，则获取tz参数所指时区的本地时间；
datetime.utcnow()：返回一个当前utc时间的datetime对象；
datetime.fromtimestamp(timestamp[, tz])：根据时间戮创建一个datetime对象，参数tz指定时区信息；
datetime.utcfromtimestamp(timestamp)：根据时间戮创建一个datetime对象；
datetime.combine(date, time)：根据date和time，创建一个datetime对象；
datetime.strptime(date_string, format)：将格式字符串转换为datetime对象；
```

实例

```python
dt = datetime.now() 
print  ((%Y-%m-%d %H:%M:%S %f): ' , dt.strftime( '%Y-%m-%d %H:%M:%S %f' ))
print  ((%Y-%m-%d %H:%M:%S %p): ' , dt.strftime( '%y-%m-%d %I:%M:%S %p' )) 
print  （%%a: %s ' % dt.strftime( '%a' ) ）
print  (%%A: %s ' % dt.strftime( '%A' ) )
print  (%%b: %s ' % dt.strftime( '%b' )) 
print  (%%B: %s ' % dt.strftime( '%B' ) )
print  (日期时间%%c: %s ' % dt.strftime( '%c' ) )
print  (日期%%x：%s ' % dt.strftime( '%x' ) )
print  (时间%%X：%s ' % dt.strftime( '%X' ) )
print (今天是这周的第%s天 ' % dt.strftime( '%w' ) )
print  (今天是今年的第%s天 ' % dt.strftime( '%j' ) )
print  (今周是今年的第%s周 ' % dt.strftime( '%U' )) 
  
# # ---- 结果 ----  
# (%Y-%m-%d %H:%M:%S %f): 2010-04-07 10:52:18 937000  
# (%Y-%m-%d %H:%M:%S %p): 10-04-07 10:52:18 AM  
# %a: Wed  
# %A: Wednesday  
# %b: Apr  
# %B: April  
# 日期时间%c: 04/07/10 10:52:18  
# 日期%x：04/07/10  
# 时间%X：10:52:18  
# 今天是这周的第3天  
# 今天是今年的第097天  
# 今周是今年的第14周   

```
### 第三部分 格式化字符的意义
```
%a   星期的简写。如 星期三为Web|
%A   星期的全写。如 星期三为Wednesday|
%b   月份的简写。如4月份为Apr
%B   月份的全写。如4月份为April
%c:  日期时间的字符串表示。（如： 04/07/10 10:43:39）
%d:  日在这个月中的天数（是这个月的第几天）
%f:  微秒（范围[0,999999]）
%H:  小时（24小时制，[0, 23]）
%I:  小时（12小时制，[0, 11]）
%j:  日在年中的天数 [001,366]（是当年的第几天）
%m:  月份（[01,12]）
%M:  分钟（[00,59]）
%p:  AM或者PM
%S:  秒（范围为[00,61]，为什么不是[00, 59]，参考python手册~_~）
%U:  周在当年的周数当年的第几周），星期天作为周的第一天
%w:  今天在这周的天数，范围为[0, 6]，6表示星期天
%W:  周在当年的周数（是当年的第几周），星期一作为周的第一天
%x:  日期字符串（如：04/07/10）
%X:  时间字符串（如：10:43:39）
%y:  2个数字表示的年份
%Y:  4个数字表示的年份
%z:  与utc时间的间隔 （如果是本地时间，返回空字符串）
%Z:  时区名称（如果是本地时间，返回空字符串）
%%:  %% => %
```

