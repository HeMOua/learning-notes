

## `Input/Output`

1、读

```python
df = read_excel(path, index_col='colum')
```

+ `index_col`：将某一列作为索引，如果不添加，默认会在第一列新添加一个由 0 开头的索引



确定时间索引

```python
df = pd.read_csv('600519.csv', index_col='trade_date', parse_dates=['trade_date'])
```

`parse_dates` 用来确定时间索引



2、写

```python
df.to_excel('output.xlsx')
```

## `DataFrame`

```python
data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Ella'],
    'Age': [25, 30, 22, 27, 35],
    'City': ['New York', 'San Francisco', 'London', 'Paris', 'Sydney'],
    'Salary': [50000, 60000, 45000, 55000, 70000]
}
df = pd.DataFrame(data)
```

+ 获取列名

```python
print(df.columns)
# Index(['Name', 'Age', 'City', 'Salary'], dtype='object')
```



## `Resampling`

> **DataFrame.resample**(*rule*, *axis=0*, *closed=None*, *label=None*, *convention='start'*, *kind=None*, *on=None*, *level=None*, *origin='start_day'*, *offset=None*, *group_keys=False*)

`rule` 参数是用于指定重采样的规则，它决定了重采样的频率和间隔。`rule` 参数可以是一个字符串，也可以是一个整数。以下是对 `rule` 参数的具体描述：

1. 字符串形式的 `rule`：
   - `'D'`：按天重采样。
   - `'W'`：按周重采样，每个周的起始时间为星期日。
   - `'M'`：按月重采样，每个月的起始时间为该月的第一天。
   - `'Q'`：按季度重采样，每个季度的起始时间为该季度的第一个月的第一天。
   - `'A'`：按年重采样，每年的起始时间为该年的第一天。

   这些字符串规则还可以与数字结合使用，例如 `'2W'` 表示每两周重采样一次，`'3M'` 表示每三个月重采样一次。

2. 整数形式的 `rule`：
   - 整数形式的 `rule` 表示以指定的周期进行重采样。例如，`rule=5` 表示每5个数据点进行一次重采样。

`rule` 参数可以控制数据重采样的频率，例如按天、周、月或其他周期进行重采样。这在对原始数据进行频率转换或聚合时非常有用。注意，`rule` 参数的选择应该根据数据的特性和需求进行合理的设置。

3、其他规则

1. `MS` 是 pandas 中用于时间序列重采样的一个字符串规则，它代表 "Month Start"，即按月开始时间进行重采样。
2. `M`：Month End，按月结束时间进行重采样。重采样后的数据将对应于每个月的最后一天。
3. `BMS`：Business Month Start，按工作日月份开始时间进行重采样。重采样后的数据将对应于每个月的第一个工作日。
4. `BM`：Business Month End，按工作日月份结束时间进行重采样。重采样后的数据将对应于每个月的最后一个工作日。
5. `Q`：Quarter End，按季度结束时间进行重采样。重采样后的数据将对应于每个季度的最后一天。
6. `BQ`：Business Quarter End，按工作日季度结束时间进行重采样。重采样后的数据将对应于每个季度的最后一个工作日。
7. `A`：Year End，按年结束时间进行重采样。重采样后的数据将对应于每一年的最后一天。
8. `BA`：Business Year End，按工作日年结束时间进行重采样。重采样后的数据将对应于每一年的最后一个工作日。

这些规则可以与不同的聚合函数（如 `sum`、`mean`、`max`、`last`、`first`等）结合使用，从而实现按照不同频率和时间点对时间序列数据进行重采样和聚合。如果您有特定的时间频率需求，可以根据具体情况选择合适的规则来处理数据。



