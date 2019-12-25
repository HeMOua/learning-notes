# sqlserver 查询时间段数据

## 当天 

```sql
select * from sysLog where DateDiff(dd,creatDate,getdate())=0
```

## 本周

由于西方国家认为每周的周日是一周的第一天，所以按照中国人的习惯，应该修改成现在这种写法

```sql
SELECT * FROM sysLog where creatDate between dateadd(week, datediff(week, 0, GETDATE()-1), 0) and GETDATE() 
```

## 本月

```sql
SELECT * FROM sysLog WHERE DATEDIFF(month, [creatDate], GETDATE()) = 0
```

## 本季度

```sql
select * from sysLog where DATEPART(qq, creatDate) = DATEPART(qq, GETDATE()) and DATEPART(yy, creatDate) = DATEPART(yy, GETDATE())
```

## 本年

```sql
select *from sysLog where year(creatDate)=year(GETDATE()) 
```

​    

