# 交易下单 API 详细参考

## stock_account — 获取资金账户句柄

```python
stock_account(account: str = '', account_type: str = 'STOCK') -> int
```

| 参数 | 类型 | 说明 |
|------|------|------|
| account | str | 资金账号，空字符串时默认当前登录账户 |
| account_type | str | 账号类型：`'STOCK'` 股票交易、`'CREDIT'` 信用交易、`'FUTURE'` 期货交易、`'OPTION'` 期权交易 |

- 返回值 ≥ 0 为有效句柄（0 也是有效的），< 0 为无效句柄
- **所有交易函数调用前，必须先调用此函数获取 account_id**
- 获取句柄前须在通达信客户端登录对应账号

示例：
```python
from tqcenter import tq
tq.initialize(__file__)
myAccount = tq.stock_account(account="1190008847", account_type="STOCK")
print(myAccount)  # 返回 int，>0 有效
```

## order_stock — 交易执行下单

```python
order_stock(
    account_id: int = -1,
    stock_code: str = '',
    order_type: int = 0,
    order_volume: int = 0,
    price_type: int = 0,
    price: float = 0.0
) -> Dict
```

| 参数 | 类型 | 说明 |
|------|------|------|
| account_id | int | 资金账号句柄（由 stock_account 获取） |
| stock_code | str | 证券代码，如 `'688318.SH'` |
| order_type | int | 委托类型，使用 tqconst 常量（见下方常量表） |
| order_volume | int | 委托数量（股） |
| price_type | int | 报价类型，使用 tqconst 常量（见下方常量表） |
| price | float | 委托价格（price_type 为市价时此参数被忽略） |

### 返回数据

| 字段 | 类型 | 说明 |
|------|------|------|
| Value | int | 成功标志：0 失败，1 待用户确认（实盘），2 成功（模拟盘） |
| Wtbh | str | 委托编号，仅成功时返回 |
| Msg | str | 返回提示信息 |
| ErrorId | str | 错误码，`'0'` 表示无错误 |

### Value 返回值说明（重要）

- **Value=0**：下单失败，检查 Msg 获取原因
- **Value=1**：实盘账户 — 信号已发送到客户端，等待用户手动确认（默认行为）
- **Value=2**：模拟盘 / 已开通 TQ 自动下单的实盘 — 直接成功

> 实盘自动下单（无需手动确认）需联系开户券商开通支持 TQ 的版本。

### 市价委托说明

当 price_type 为 `tqconst.PRICE_SJ`（市价）时，具体使用哪种市价方式由通达信客户端设置决定：
客户端 → 系统设置 → 参数 中配置。

示例：
```python
from tqcenter import tq
from tqcenter import tqconst
tq.initialize(__file__)

myAccount = tq.stock_account(account="1190008847", account_type="STOCK")

# 限价买入
order_res = tq.order_stock(
    account_id=myAccount,
    stock_code="688318.SH",
    order_type=tqconst.STOCK_BUY,
    order_volume=200,
    price_type=tqconst.PRICE_MY,
    price=160.0
)
print(order_res)
# 实盘返回: {'ErrorId': '0', 'Msg': '已发送信号至客户端，待用户确认！', 'Value': 1}
# 模拟盘返回: {'ErrorId': '0', 'Msg': '...', 'Value': 2, 'Wtbh': '12345'}
```

## query_stock_asset — 查询账户资产

```python
query_stock_asset(account_id: int = -1) -> Dict
```

| 返回字段 | 类型 | 说明 |
|---------|------|------|
| Currency | str | 币种 |
| Balance | str | 余额 |
| Cash | str | 可用余额 |
| Asset | str | 总资产 |
| MarketValue | str | 总市值 |
| TotalFreeze | str | 期货冻结资金 |
| CloseProfit | str | 期货平仓盈亏 |
| CurrentEquity | str | 期货动态权益 |
| PreviousEquity | str | 期货静态权益 |
| ProfitLoss | str | 期货持仓盈亏 |
| TotalMargin | str | 期货持仓保证金 |
| ErrorId | str | 错误码 |

> 股票账户主要关注 Balance（余额）、Cash（可用余额）、Asset（总资产）、MarketValue（总市值）。
> 期货相关字段仅期货账户有效。

示例：
```python
myAccount = tq.stock_account(account="1190008847", account_type="STOCK")
zc_res = tq.query_stock_asset(account_id=myAccount)
print(zc_res)
# {'Currency': '人民币', 'Balance': '30234.070', 'Cash': '30234.070',
#  'Asset': '1233041.070', 'MarketValue': '1201690.000', 'ErrorId': '0'}
```

## query_stock_positions — 查询账户持仓

```python
query_stock_positions(account_id: int = -1) -> List[Dict]
```

| 返回字段 | 类型 | 说明 |
|---------|------|------|
| Code | str | 股票代码 |
| Cbj | str | 成本价 |
| TotalVol | str | 总持仓量 |
| CanUseVol | str | 可用数量（T+1 限制下，今日买入的不可用） |

示例：
```python
myAccount = tq.stock_account(account="1190008847", account_type="STOCK")
positions = tq.query_stock_positions(account_id=myAccount)
print(positions)
# [{'Code': '000001.SZ', 'Cbj': '10.693', 'TotalVol': '100', 'CanUseVol': '100'},
#  {'Code': '688318.SH', 'Cbj': '117.425', 'TotalVol': '4000', 'CanUseVol': '4000'}]
```

## query_stock_orders — 查询账户委托

```python
query_stock_orders(
    account_id: int = -1,
    stock_code: str = '',
    cancelable_only: bool = False
) -> List[Dict]
```

| 参数 | 类型 | 说明 |
|------|------|------|
| account_id | int | 资金账号句柄 |
| stock_code | str | 证券代码，空字符串查全部 |
| cancelable_only | bool | 是否仅查询可撤委托（暂未生效） |

### 返回数据

| 字段 | 类型 | 说明 |
|------|------|------|
| Wtbh | str | 委托编号 |
| Code | str | 股票代码 |
| Time | str | 时间，HHMMSS |
| BSFlag | int | 买卖标志：0 买、1 卖、-1 撤单 |
| KPFlag | int | 开平标志：0 开仓、1 平仓、2 平今 |
| WTFS | int | 市价方式（沪深不同） |
| Status | int | 委托状态（见下方状态码） |
| WtPrice | str | 委托价 |
| WtVol | str | 委托数量（撤单时为负值） |
| CjPrice | str | 成交价 |
| CjVol | str | 成交数量（撤单时为负值） |
| WtDate | int | 撤单标志：1 已撤、2 夜盘单 |

### 委托状态码

| 常量 | 值 | 含义 |
|------|---|------|
| WTSTATUS_NULL | 0 | 无效单 |
| WTSTATUS_NOCJ | 1 | 未成交 |
| WTSTATUS_PARTCJ | 2 | 部分成交 |
| WTSTATUS_ALLCJ | 3 | 全部成交 |
| WTSTATUS_BCBC | 4 | 部分成交部分撤单 |
| WTSTATUS_ALLCD | 5 | 全部撤单 |

> 委托查询只能查询当日委托。

示例：
```python
myAccount = tq.stock_account(account="1190008847", account_type="STOCK")
orders = tq.query_stock_orders(account_id=myAccount, stock_code="")
print(orders)
# [{'Wtbh': '48957', 'Code': '688318.SH', 'Time': '93605', 'BSFlag': -1,
#   'KPFlag': 0, 'WTFS': 0, 'Status': 0, 'WtPrice': '125.000',
#   'CjPrice': '0.000', 'CjVol': '0', 'WtVol': '1000'}]
```

## cancel_order_stock — 撤单

```python
cancel_order_stock(
    account_id: int = -1,
    stock_code: str = '',
    order_id: str = ''
) -> Dict
```

| 参数 | 类型 | 说明 |
|------|------|------|
| account_id | int | 资金账号句柄 |
| stock_code | str | 证券代码 |
| order_id | str | 委托编号（从 query_stock_orders 的 Wtbh 字段获取） |

### 返回数据

| 字段 | 类型 | 说明 |
|------|------|------|
| Value | int | 成功标志：0 失败、1 成功 |
| Msg | str | 返回提示信息 |
| ErrorId | str | 错误码 |

撤单成功后，对应委托的 Status 会变为：WTSTATUS_NULL(0)、WTSTATUS_BCBC(4) 或 WTSTATUS_ALLCD(5)。

示例：
```python
from tqcenter import tq
from tqcenter import tqconst
tq.initialize(__file__)

myAccount = tq.stock_account(account="1190008847", account_type="STOCK")

# 查询当前委托
orders = tq.query_stock_orders(account_id=myAccount, stock_code="")

# 撤销第一笔委托
if orders:
    cancel_res = tq.cancel_order_stock(
        account_id=myAccount,
        stock_code=orders[0]['Code'],
        order_id=orders[0]['Wtbh']
    )
    print(cancel_res)
    # {'Value': 1, 'ErrorId': '0', 'Msg': '提交撤单成功！'}
```