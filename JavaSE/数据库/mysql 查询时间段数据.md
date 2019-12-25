# mysql 查询时间段数据

## 今天

```sql
select * from 表名 where to_days(时间字段名) = to_days(now());
```

## 昨天

```sql
SELECT * FROM 表名 WHERE TO_DAYS( NOW( ) ) - TO_DAYS( 时间字段名) = 1
```

## 近7天

```sql
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(时间字段名)
```

## 近30天

```sql
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(时间字段名)
```

## 本月

```sql
SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' )
```

## 上一月

```sql
SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1
```

## 查询本季度数据

```sql
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(now());
```

## 查询上季度数据

```sql
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));
```

## 查询本年数据

```sql
select * from `ht_invoice_information` where YEAR(create_date)=YEAR(NOW());
```

## 查询上年数据

```sql
select * from `ht_invoice_information` where year(create_date)=year(date_sub(now(),interval 1 year));
```

## 查询当前这周的数据

```sql
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now());
```

## 查询上周的数据

```sql
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now())-1;
```

## 查询上个月的数据

```sql
select name,submittime from enterprise where date_format(submittime,'%Y-%m')=date_format(DATE_SUB(curdate(), INTERVAL 1 MONTH),'%Y-%m')

select * from user where DATE_FORMAT(pudate,'%Y%m') = DATE_FORMAT(CURDATE(),'%Y%m') ; 

select * from user where WEEKOFYEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = WEEKOFYEAR(now()) 

select * from user where MONTH(FROM_UNIXTIME(pudate,'%y-%m-%d')) = MONTH(now()) 

select * from user where YEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = YEAR(now()) and MONTH(FROM_UNIXTIME(pudate,'%y-%m-%d')) = MONTH(now()) 
select * from user where pudate between  上月最后一天  and 下月第一天 
```

## 查询当前月份的数据

```sql
select name,submittime from enterprise   where date_format(submittime,'%Y-%m')=date_format(now(),'%Y-%m')
```

## 查询距离当前现在6个月的数据

```sql
select name,submittime from enterprise where submittime between date_sub(now(),interval 6 month) and now();
```

 

## 原文

[mysql 查询当天、本周，本月，上一个月的数据](https://www.cnblogs.com/benefitworld/p/5832897.html)