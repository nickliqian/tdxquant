---
name: tdxquant
description: >
  TdxQuant（通达信量化）全能开发助手。基于 tqcenter Python API 帮助用户完成量化策略开发全流程：
  数据获取、选股策略、回测验证、实时监控、条件预警、公式调用、板块管理。
  触发场景：用户提到"量化策略"、"选股"、"回测"、"TdxQuant"、"通达信量化"、"tqcenter"、
  "K线"、"因子"、"板块"、"金叉死叉"、"实时监控"、"行情订阅"、"预警"、"subscribe"、
  "盯盘"、"涨停"、"tq.get_"、"tq.send_"、"tq.formula_"、"FN字段"、"GP字段"、
  "price_df"、"get_full_tick"等关键词时使用。
---

# TdxQuant 全能开发助手

## 环境前提

- Python 3.7+（建议 3.13），64位
- 需预先启动通达信金融终端（支持 TQ 策略功能）
- 所有策略必须先初始化：`from tqcenter import tq; tq.initialize(__file__)`

## 策略开发流程

1. 初始化连接
2. 确认股票池来源（生成选股/回测代码前，先确认用户要从哪个板块选股，明确板块名称和 block_type 参数；如果用户没指定，推荐沪深300或中证500作为起点）
3. 获取数据（K线/快照/财务/板块成份股）
4. 信号计算（均线/因子/公式）
5. 执行操作（回测/预警/写入板块/下单）
6. 结果输出（print_to_tdx/send_message）
7. 如果是回测策略，询问用户是否需要生成配套的可视化脚本（收益曲线、回撤图等）

## 常用策略模式

### 选股入板块

```python
from tqcenter import tq
tq.initialize(__file__)

codes = tq.get_stock_list_in_sector('通达信88')
df = tq.get_market_data(
    field_list=['Close'], stock_list=codes,
    start_time='20250101', end_time='',
    dividend_type='front', period='1d', fill_data=True
)
close_df = tq.price_df(df, 'Close', column_names=codes)
# ... 信号逻辑 ...
tq.send_user_block(block_code='MYBK', stocks=selected_list, show=True)
```

### 回测策略（配合 vectorbt）

```python
import vectorbt as vbt
from tqcenter import tq
tq.initialize(__file__)

df = tq.get_market_data(
    field_list=['Close', 'Open'], stock_list=['688318.SH'],
    start_time='20240101', end_time='20250101',
    dividend_type='front', period='1d', fill_data=True
)
close_df = tq.price_df(df, 'Close', column_names=['688318.SH'])
open_df = tq.price_df(df, 'Open', column_names=['688318.SH'])

ma = vbt.MA.run(close_df, window=5).ma
entries = close_df.vbt.crossed_above(ma).shift(1).fillna(False).astype(bool)
exits = close_df.vbt.crossed_below(ma).shift(1).fillna(False).astype(bool)

pf = vbt.Portfolio.from_signals(
    close=close_df, entries=entries, exits=exits,
    price=open_df, init_cash=100000, fees=0.0003,
    freq='D', size_granularity=100
)
print(pf.stats())
```

### 调用通达信公式

```python
# 单只股票
tq.formula_set_data_info(stock_code='688318.SH', stock_period='1d', count=100, dividend_type=1)
result = tq.formula_zb(formula_name='MACD', formula_arg='12,26,9')

# 批量选股
xg = tq.formula_process_mul_xg(
    formula_name='UPN', formula_arg='3',
    stock_list=['688318.SH','600519.SH'],
    stock_period='1d', count=5, dividend_type=1
)
```

### 实时监控与预警

```python
import json, time
from tqcenter import tq
tq.initialize(__file__)

triggered = set()

def on_update(data_str):
    try:
        code = json.loads(data_str).get('Code')
        if code in triggered:
            return
        snap = tq.get_market_snapshot(stock_code=code)
        now, pre = float(snap['Now']), float(snap['LastClose'])
        if pre <= 0: return
        rise = (now - pre) / pre * 100
        if rise > 5.0:
            triggered.add(code)
            tq.unsubscribe_hq(stock_list=[code])
            from datetime import datetime
            tq.send_warn(
                stock_list=[code],
                time_list=[datetime.now().strftime("%Y%m%d%H%M%S")],
                price_list=[str(now)], close_list=[str(pre)],
                volum_list=[snap.get('Volume','0')],
                bs_flag_list=['0'], warn_type_list=['0'],
                reason_list=[f'涨幅{rise:.2f}%突破5%'], count=1
            )
    except Exception as e:
        print(f"回调异常: {e}")

codes = tq.get_stock_list_in_sector('通达信88')[:100]
tq.subscribe_hq(stock_list=codes, callback=on_update)
while True:
    time.sleep(1)
```

### 财务选股（基本面筛选）

```python
from tqcenter import tq
tq.initialize(__file__)

# 获取沪深300成份股
codes = tq.get_stock_list('23')

# 获取最新财务数据：ROE(FN197)、营收增长率(FN183)、资产负债率(FN210)
fd = tq.get_financial_data_by_date(
    stock_list=codes,
    field_list=['Fn197', 'Fn183', 'Fn210'],
    year=0, mmdd=0  # 0表示最新
)

# 筛选 ROE > 15% 且 营收增长 > 10% 且 资产负债率 < 60%
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

print(f"筛选出 {len(selected)} 只股票")
tq.send_user_block(block_code='CWXG', stocks=selected, show=True)
```

### 行业轮动选股

```python
from tqcenter import tq
tq.initialize(__file__)

# 获取所有行业板块
sectors = tq.get_stock_list('16', list_type=1)  # 研究行业一级

# 获取各板块近5日涨幅，筛选前3
import pandas as pd
sector_perf = {}
for s in sectors:
    code = s['Code']
    try:
        bk = tq.get_bkjy_value(stock_list=[code], field_list=['BK1'], start_time='', end_time='')
        if code in bk and not bk[code].empty:
            sector_perf[code] = float(bk[code].iloc[-1].get('BK1', 0))
    except Exception:
        continue

top3 = sorted(sector_perf, key=sector_perf.get, reverse=True)[:3]

# 从前3行业中获取成份股
all_codes = []
for sec in top3:
    all_codes.extend(tq.get_stock_list_in_sector(sec))

# ... 对 all_codes 做进一步选股/回测 ...
```

## 常用辅助函数

详见 [market-data.md](references/market-data.md) 中的 price_df 和 get_full_tick 章节。

- `tq.price_df(df, 'Close', column_names=codes)` — 将 get_market_data 返回值转为宽表（行=日期, 列=股票代码）
- `tq.get_full_tick(stock_code)` — 获取完整 tick 数据，返回字段与 get_market_snapshot 类似，常用于订阅回调

## 关键约束与常见陷阱

- `get_market_data` 单次最多 24000 条，分钟线需分批获取
- `get_market_data` 返回的 DataFrame 是 index=stock_list、columns=time_list，与 pandas 常见的"行=时间、列=股票"相反。用 `tq.price_df()` 转置为常规格式
- `subscribe_hq` 最多订阅 100 条
- `send_warn` 的 reason_list 每个元素最多 25 汉字；注意参数名是 `volum_list`（非 volume_list）
- `formula_set_data` 的 count 最大 24000
- 批量公式（formula_process_mul_xg/zb）无需提前 set_data
- 复权类型有两套写法：行情 API 用字符串 `'none'`/`'front'`/`'back'`；公式 API 用整数 `0`/`1`/`2`
- 周期：`1m` `5m` `15m` `30m` `60m` `1d` `1w` `1mon` `1q` `1hy` `tick`

## API 参考（按需加载）

查询具体接口参数和字段时，参见对应文件：

- [行情与基础数据](references/market-data.md) — get_market_data, get_market_snapshot, get_stock_info, get_more_info 等
- [财务与交易数据](references/financial-data.md) — get_financial_data(FN1-584), get_gpjy_value(GP01-46), get_bkjy_value, get_scjy_value, get_gp_one_data 等
- [板块管理与公式调用](references/sector-formula.md) — get_stock_list, get_stock_list_in_sector, send_user_block, formula_zb/xg/exp, formula_process_mul 等
- [订阅与通知](references/subscribe-notify.md) — subscribe_hq, send_warn, send_message, send_file, print_to_tdx 等
- [常量枚举](references/constants.md) — 市场后缀, 复权类型, K线周期, market参数值
