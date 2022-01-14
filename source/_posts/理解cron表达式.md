---
title: 理解cron表达式
date: 2018-3-1 11:00:00
updated: 2018-3-1 11:50:00
comments: true
categories:
- Java
tags:
- 教程
- Java
---

## A Cron Expressions

cron表达式形如 `0 0 12 * * ?` ，由 `Seconds` `Minutes` `Hours` `Day of month` `Month` `Day of week` `Year` 字段构成，即`几秒` `几分` `几时` `几号` `几月` `周几` `哪年`。 字段之间使用空格分隔，每个字段可包含任意多个合法的字符。

Table A-1 Cron Expressions Allowed Fields and Values

| Name         | Required | Allowed Values    | Allowed Special Characters |
| ------------ | :------: | :---------------: | :------------------------: |
| Seconds      | Y        | 0-59              | , - * /                    |
| Minutes      | Y        | 0-59              | , - * /                    |
| Hours        | Y        | 0-23              | , - * /                    |
| Day of month | Y        | 1-31              | , - * ? / L W              |
| Month        | Y        | 1-12 or JAN-DEC   | , - * /                    |
| Day of week  | Y        | 1-7 or SUN-SAT    | , - * ? / L #              |
| Year         | N        | empty or 1970-2099| , - * /                    |

<!--more-->

## Special Characters

`*` (“all values”) - 用于表示字段中的所有值。例如，`Minutes`字段中的“*”意味着每一分钟。

`?` (“no specific value”) -  不指定特定的值，举个例子，如果我希望我的任务在某月中特定的某天（如第10天）被触发，并且我不关心是星期几被触发，那么我会设置`day-of-month`字段为`10`，设置`day-of-week`字段为`?`

`-` - 用于指定范围.如`hour`值为`10-12` 表示10时，11时，12时。

`,` - 用于指定附加的值。例如，`day-of-week`的值为`MON,WED,FRI`表示星期天，星期三，星期五

`/` - 用于指定增长。例如，秒字段值为“0/15”表示具体为第0秒，15秒，30秒，45秒的集合，即从第0秒开始，间隔15秒累加。秒字段值为“5/15”表示具体为第5秒，20秒，35秒，50秒的集合，即从第5秒开始，间隔15秒累加。也可以把/写在''(即空串，没有'')后面如'/15',这种情况等价于'0/15'。`day-of-month`等于`1/3` 表示从某月的第一天开始，每隔3天触发。

`L` (“last”) - 指最后，不同的情况有不同的含义，如`day-of-month`等于“L”表示月中的最后一天。`day-of-week`等于“L”仅表示7或者SAT。`day-of-week`等于“6L”表示月中的最后一个周五。也可以从月中的最后一天指定一个偏移量，例如`day-of-month`等于“L-3”表示月中的倒数第三天。当使用“L”选项时，切记不要指定列表或值的范围，因为可能会得到令人困惑意外的结果。

`W` (“weekday”) - 用于指定离指定日最近的工作日（星期一至星期五）。例如，如果您要将“15W”指定为`day-of-month`字段的值，则含义为：“距离本月15日最近的工作日”。因此，如果15日是星期六，触发器将于14日星期五触发。如果15日是星期天，触发器将在16日星期一触发。如果15日是星期二，那么它将在15日星期二触发。 但是，如果您将“1W”指定为`day-of-month`的值，并且1号是星期六，则触发器将在星期一即3号触发，因为它不能跨越当前月的边界。'W'字符只能在`day-of-month`是单日而不是日期范围或日期列表时指定。

L和W字符也能联合使用在`day-of-mouth`字段中，即`LW`，它表示月中的最后一个工作日。

`#` - 用于指定月中的第几天。例如，`day-of-week`等于“6#3”表示月中的第三个星期五。又如，“2#1” 表示月中的第一个星期一，“4#5”表示月中的第五个星期三。注意如果你指定了“#5”，并且该月的星期不存在5个，则该月不会发生任何触发。

合法字符串、月份名称、星期名称不区分大小写，MON等价于mon。

## Examples

下面的例子自己理解一下吧~
```bash
# Fire at 12:00 PM (noon) every day
0 0 12 * * ?
# Fire at 10:15 AM every day
0 15 10 ? * *	
# Fire at 10:15 AM every day
0 15 10 * * ?
# Fire at 10:15 AM every day
0 15 10 * * ? *	
# Fire at 10:15 AM every day during the year 2005
0 15 10 * * ? 2005	
# Fire every minute starting at 2:00 PM and ending at 2:59 PM, every day
0 * 14 * * ?	
# Fire every 5 minutes starting at 2:00 PM and ending at 2:55 PM, every day
0 0/5 14 * * ?	
# Fire every 5 minutes starting at 2:00 PM and ending at 2:55 PM, AND fire every 5 minutes starting at 6:00 PM and ending at 6:55 PM, every day
0 0/5 14,18 * * ?	
# Fire every minute starting at 2:00 PM and ending at 2:05 PM, every day
0 0-5 14 * * ?	
# Fire at 2:10 PM and at 2:44 PM every Wednesday in the month of March
0 10,44 14 ? 3 WED	
# Fire at 10:15 AM every Monday, Tuesday, Wednesday, Thursday and Friday
0 15 10 ? * MON-FRI	
# Fire at 10:15 AM on the 15th day of every month
0 15 10 15 * ?	
# Fire at 10:15 AM on the last day of every month
0 15 10 L * ?	
# Fire at 10:15 AM on the last Friday of every month
0 15 10 ? * 6L	
# Fire at 10:15 AM on the last Friday of every month
0 15 10 ? * 6L	
# Fire at 10:15 AM on every last friday of every month during the years 2002, 2003, 2004, and 2005
0 15 10 ? * 6L 2002-2005	
# Fire at 10:15 AM on the third Friday of every month
0 15 10 ? * 6#3	
# Fire at 12 PM (noon) every 5 days every month, starting on the first day of the month
0 0 12 1/5 * ?	
# Fire every November 11 at 11:11 AM
0 11 11 11 11 ?	
```

参考资料：
1.[Cron Expressions](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm)
2.[Cron Trigger Tutorial](http://www.quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/crontrigger.html)