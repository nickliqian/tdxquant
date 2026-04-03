# TdxQuant Claude Code Skill

TdxQuant（通达信量化）全能开发助手。基于 tqcenter Python API 帮助用户完成量化策略开发全流程。

## 功能特性

- 📊 **数据获取** - K线、快照、财务、板块成份股
- 🎯 **选股策略** - 技术指标、基本面筛选、公式调用
- 📈 **回测验证** - 配合 vectorbt 进行策略回测
- 🔔 **实时监控** - 行情订阅、条件预警
- 📦 **板块管理** - 自定义板块、批量操作
- 🧮 **公式调用** - 调用通达信内置公式

## 环境要求

- Python 3.7+（建议 3.13），64位
- 通达信金融终端（支持 TQ 策略功能）
- 依赖包：`tqcenter`, `vectorbt`（可选）

## 快速开始

### 1. 初始化连接

```python
from tqcenter import tq
tq.initialize(__file__)
```

### 2. 获取行情数据

```python
# 获取K线数据
df = tq.get_market_data(
    field_list=['Close', 'Open', 'High', 'Low', 'Volume'],
    stock_list=['688318.SH', '600519.SH'],
    start_time='20240101',
    end_time='20250101',
    dividend_type='front',  # 前复权
    period='1d',
    fill_data=True
)

# 转换为宽表格式（行=日期，列=股票）
close_df = tq.price_df(df, 'Close', column_names=['688318.SH', '600519.SH'])
```

### 3. 选股入板块

```python
codes = tq.get_stock_list_in_sector('通达信88')
selected = []

for code in codes:
    snap = tq.get_market_snapshot(stock_code=code)
    # 筛选逻辑...
    if condition:
        selected.append(code)

# 写入自定义板块
tq.send_user_block(block_code='MYBK', stocks=selected, show=True)
```

### 4. 实时监控预警

```python
def on_update(data_str):
    code = json.loads(data_str).get('Code')
    snap = tq.get_market_snapshot(stock_code=code)
    # 预警逻辑...
    if trigger_condition:
        tq.send_warn(
            stock_list=[code],
            reason_list=['触发条件'],
            # ... 其他参数
        )

codes = tq.get_stock_list_in_sector('通达信88')[:100]
tq.subscribe_hq(stock_list=codes, callback=on_update)
```

## 常用策略模式

### 均线金叉选股

```python
from tqcenter import tq
import pandas as pd

tq.initialize(__file__)

codes = tq.get_stock_list_in_sector('通达信88')
df = tq.get_market_data(
    field_list=['Close'],
    stock_list=codes,
    start_time='20240101',
    end_time='',
    dividend_type='front',
    period='1d',
    fill_data=True
)

close_df = tq.price_df(df, 'Close', column_names=codes)
ma5 = close_df.rolling(5).mean()
ma20 = close_df.rolling(20).mean()

# 今日金叉
golden_cross = (ma5.iloc[-1] > ma20.iloc[-1]) & (ma5.iloc[-2] <= ma20.iloc[-2])
selected = golden_cross[golden_cross].index.tolist()

print(f"金叉股票: {len(selected)} 只")
tq.send_user_block(block_code='JCBK', stocks=selected, show=True)
```

### 财务基本面筛选

```python
from tqcenter import tq

tq.initialize(__file__)

codes = tq.get_stock_list('23')  # 沪深300

fd = tq.get_financial_data_by_date(
    stock_list=codes,
    field_list=['Fn197', 'Fn183', 'Fn210'],  # ROE、营收增长率、资产负债率
    year=0, mmdd=0
)

selected = []
for code, data in fd.items():
    try:
        roe = float(data.get('FN197', 0))
        rev_growth = float(data.get('FN183', 0))
        debt_ratio = float(data.get('FN210', 100))

        if roe > 15 and rev_growth > 10 and debt_ratio < 60:
            selected.append(code)
    except (ValueError, TypeError):
        continue

print(f"筛选出 {len(selected)} 只优质股")
tq.send_user_block(block_code='CWXG', stocks=selected, show=True)
```

### 回测策略（vectorbt）

```python
import vectorbt as vbt
from tqcenter import tq

tq.initialize(__file__)

df = tq.get_market_data(
    field_list=['Close', 'Open'],
    stock_list=['688318.SH'],
    start_time='20240101',
    end_time='20250101',
    dividend_type='front',
    period='1d',
    fill_data=True
)

close_df = tq.price_df(df, 'Close', column_names=['688318.SH'])
open_df = tq.price_df(df, 'Open', column_names=['688318.SH'])

# 5日均线策略
ma = vbt.MA.run(close_df, window=5).ma
entries = close_df.vbt.crossed_above(ma).shift(1).fillna(False).astype(bool)
exits = close_df.vbt.crossed_below(ma).shift(1).fillna(False).astype(bool)

pf = vbt.Portfolio.from_signals(
    close=close_df,
    entries=entries,
    exits=exits,
    price=open_df,
    init_cash=100000,
    fees=0.0003,
    freq='D',
    size_granularity=100
)

print(pf.stats())
```

## API 参考文档

详细 API 文档请查看 `references/` 目录：

- [market-data.md](references/market-data.md) - 行情与基础数据
- [financial-data.md](references/financial-data.md) - 财务与交易数据
- [sector-formula.md](references/sector-formula.md) - 板块管理与公式调用
- [subscribe-notify.md](references/subscribe-notify.md) - 订阅与通知
- [constants.md](references/constants.md) - 常量枚举

## 关键约束

- `get_market_data` 单次最多 24000 条
- `subscribe_hq` 最多订阅 100 只股票
- `send_warn` 的 reason_list 每个元素最多 25 汉字
- 复权类型：行情 API 用 `'none'`/`'front'`/`'back'`，公式 API 用 `0`/`1`/`2`
- 周期：`1m` `5m` `15m` `30m` `60m` `1d` `1w` `1mon` `1q` `1hy` `tick`

## 许可证

MIT License
