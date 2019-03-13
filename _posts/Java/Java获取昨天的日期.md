---
date: 2015-04-03 17:46
status: public
title: Java获取昨天的日期
categories: Java
---

###Java中获取昨天的日期的几种方式：

1， 通过今天的long型数据减去一天的long型值
```
Date as = new Date(new Date().getTime()-24*60*60*1000);
SimpleDateFormat matter1 = new SimpleDateFormat("yyyy-MM-dd");
String time = matter1.format(as);
System.out.println(time);
```
`取出数字型的时间  再减去24*60*60*1000，就得到昨天的时间了`
2， 通过Calendar类进行计算(方便快捷，推荐使用)，年月日均可通过此方式获得
```
Calendar cal = Calendar.getInstance();
cal.add(Calendar.DATE,-1); //第一个参数表示需要计算的字段；第二个相加的值
String yesterday = new SimpleDateFormat("yyyy-MM-dd ").format(cal.getTime());
System.out.println(yesterday);
```
`这个方法很方便，年月日都可以随心所欲的变！`
3，使用第三方库

apache的DateUtils（需要import org.apache.commons.lang.time.DateUtils）
```
Date currentTime = new Date();
//获取昨天时间
Date backupTime=DateUtils.addDays(currentTime, -1);
```

4, sql 查询条件包含时间的处理方法：
```
select * from TBIMC1 where CREATE_DATE_<to_date( to_char(sysdate-1,'yyyy-mm-dd'),'yyyy-mm-dd')
```