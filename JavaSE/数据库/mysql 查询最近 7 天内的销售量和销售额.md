查询最近 7 天内的销售量和销售额

```sql
select a.clickDate, ifnull(b.salesSum,0) as salesSum, ifnull(b.orderCount,0) as orderCount
from (
		SELECT curdate() as clickDate
		union all
		SELECT date_sub(curdate(), interval 1 day) as clickDate
		union all
		SELECT date_sub(curdate(), interval 2 day) as clickDate
		union all
		SELECT date_sub(curdate(), interval 3 day) as clickDate
		union all
		SELECT date_sub(curdate(), interval 4 day) as clickDate
		union all
		SELECT date_sub(curdate(), interval 5 day) as clickDate
		union all
		SELECT date_sub(curdate(), interval 6 day) as clickDate
) a left join (
		select date(PAY_TIME) datetime, sum(PAYMENT) salesSum , count(id) orderCount
		from sys_bill
		group by date(PAY_TIME)
) b on a.clickDate = b.datetime;
```

