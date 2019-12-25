## `Exchange`封装

```python
from enum import Enum
# 打印错误
import traceback
import json
import time

PLACEHOLDER = '---'


def init_attr(obj, *attr, value=PLACEHOLDER):
    assert len(attr) > 0, 'attr must not be empty'
    for a in attr:
        setattr(obj, a, value)


class Status(Enum):
    SUCCESS = 0
    FAILED = 1
    ERROR = 2


class Result:
    def __init__(self, code: Status, msg: str, data: object):
        self.code = code
        self.msg = msg
        self.data = data

    @classmethod
    def success(cls, data=None, msg=''):
        return cls(Status.SUCCESS, msg, data)
    
    @classmethod
    def failed(cls, data=None, msg=''):
        return cls(Status.FAILED, msg, data)
    
    @classmethod
    def error(cls, data=None, msg=''):
        return cls(Status.ERROR, msg, data)
    
    def __str__(self):
        return self.tostr(filter_empty=True)
    
    def tostr(self, filter_empty=True):
        def is_filte(x):
            if x is None or x == '':
                return True
            return False
        result_dict = {attr: getattr(self, attr).value if isinstance(getattr(self, attr), Enum) else getattr(self, attr)
                       for attr in vars(self) if not (filter_empty and is_filte(getattr(self, attr)))}
        return json.dumps(result_dict, sort_keys=True, ensure_ascii=False)


class Trader:

    def __init__(self, exchange):
        self.exchange = exchange
        # 交易所名称
        self.name = exchange.GetName()
        # 交易所货币对
        self.currency = exchange.GetCurrency()

    def get_account(self):
        init_attr(self, 'raw_account_info', 'balance', 'frozen_balance', 'stocks', 'frozen_stocks')
        try:
            account = self.exchange.GetAccount()
            self.raw_account = account.get('Info')
            self.balance = account['Balance']
            self.frozen_balance = account['FrozenBalance']
            self.stocks = account['Stocks']
            self.frozen_stocks = account['FrozenStocks']
            return Result.success()
        except Exception as e:
            # 如果获取账户信息失败，会影响交易决策
            Log(traceback.format_exc())
            return Result.error(msg='获取账户信息失败')

    def get_ticker(self):
        init_attr(self, 'raw_ticker', 'buy', 'sell', 'last', 'high', 'low', 'volume')
        try:
            ticker = self.exchange.GetTicker()
            self.raw_ticker = ticker.get('Info')
            self.buy = ticker['Buy']
            self.sell = ticker['Sell']
            self.last = ticker['Last']
            self.high = ticker['High']
            self.low = ticker['Low']
            self.volume = ticker['Volume']
            return Result.success()
        except:
            # 如果获取行情失败，就不交易
            Log(traceback.format_exc())
            return Result.error(msg='获取行情失败' + str(self))

    def get_depth(self):
        init_attr(self, 'asks', 'bids')
        try:
            depth = self.exchange.GetDepth()
            self.asks = depth['Asks']
            self.bids = depth['Bids']
            return Result.success()
        except:
            return Result.error(msg='获取深度失败')

    def get_record(self, period, tail=-1):
        init_attr(self, 'record')
        try:
            records = self.exchange.GetRecords(period)
            if tail > 0:
                records = records[-tail:]
            self.record = records
            return Result.success()
        except:
            return Result.error(msg='获取K线失败')

    def create_order(self, type, price, amount):
        assert type in ['buy', 'sell'], 'type must be buy or sell'

        try:
            order_id = self.exchange.Buy(price, amount) if type == 'buy' else self.exchange.Sell(price, amount)
            return Result.success(data={'order_id': order_id, 'time': int(time.time())})
        except:
            return Result.error(msg='下单失败')

    def cancel_order(self, order_id):
        try:
            order_id = self.exchange.CancelOrder(order_id)
            return Result.success(data={'order_id': order_id, 'time': int(time.time())})
        except:
            return Result.error(msg='撤单失败')

    def refresh(self):
        reuslt = self.get_account()
        if reuslt.code != Status.SUCCESS:
            return reuslt
        reuslt = self.get_ticker()
        if reuslt.code != Status.SUCCESS:
            return reuslt
        reuslt = self.get_depth()
        if reuslt.code != Status.SUCCESS:
            return reuslt
        return Result.success(msg='刷新成功')
```



## 邮箱封装

```python
from fmz import *
from enum import Enum
task = VCtx(__doc__) # initialize backtest engine from __doc__

class MailHelper:

    class Type(Enum):
        NETEASY = ('网易163', 'smtp.163.com', 25)
        QQ = ('QQ邮箱', 'smtp.qq.com', 25)
        GMAIL = ('Gmail', 'smtp.gmail.com', 587)

        def name(self):
            return self.value[0]
        
        def host(self):
            return self.value[1]
        
        def port(self):
            return self.value[2]

    def __init__(self, type, user, password):
        self.type = type
        self.user = user
        self.password = password

    def send(self, to, title, content, need_port=False, asyn=False):
        host = f'{self.type.host()}:{self.type.port()}' if need_port else self.type.host()

        if asyn:
            pass
            # Mail_Go(host, self.user, self.password, to, title, content)
        else:
            Mail(host, self.user, self.password, to, title, content)
```

本地测试了，但发不了

