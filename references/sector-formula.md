# 板块管理与公式调用 API

## 目录
- [get_stock_list - 获取系统分类成份股](#get_stock_list)
- [get_sector_list - 获取A股板块代码列表](#get_sector_list)
- [get_stock_list_in_sector - 获取板块成份股](#get_stock_list_in_sector)
- [get_user_sector - 获取自定义板块列表](#get_user_sector)
- [send_user_block - 添加自定义板块成份股](#send_user_block)
- [create_sector - 创建自定义板块](#create_sector)
- [delete_sector - 删除自定义板块](#delete_sector)
- [rename_sector - 重命名自定义板块](#rename_sector)
- [clear_sector - 清空自定义板块成份股](#clear_sector)
- [formula_format_data - 格式化K线数据](#formula_format_data)
- [formula_set_data - 设置公式数据](#formula_set_data)
- [formula_set_data_info - 设置公式数据信息](#formula_set_data_info)
- [formula_get_data - 获取公式设置数据](#formula_get_data)
- [formula_zb/xg/exp - 调用通达信公式](#formula_zb)
- [formula_process_mul_xg/zb - 批量调用公式](#formula_process_mul)

## get_stock_list

获取系统分类成份股。

```python
get_stock_list(market=None, list_type: int = 0) -> List
```

list_type: 0只返回代码, 1返回代码和名称。

market 常用值：
- 5:所有A股, 7:上证主板, 8:深证主板, 50:沪深A股, 51:创业板, 52:科创板, 53:北交所
- 10:所有板块指数, 16:研究行业一级, 12:概念板块, 14:地区板块
- 23:沪深300, 24:中证500, 25:中证1000, 28:中证A500
- 31:ETF基金, 32:可转债, 101:国内期货, 102:港股, 103:美股

## get_sector_list

获取A股全部板块代码列表（等同于 `get_stock_list('10')`）。

```python
get_sector_list(list_type: int = 0) -> List
```

## get_stock_list_in_sector

获取板块成份股。

```python
get_stock_list_in_sector(block_code: str, block_type: int = 0, list_type: int = 0) -> List
```

### block_type 参数

- `block_type=0`（默认）：传入板块指数代码或板块名称
  - 板块名称：如 `'通达信88'`、`'煤炭开采'`、`'钛金属'`
  - 板块代码：如 `'880301'`
- `block_type=1`：传入自定义板块简称
  - 用户自建板块：如 `'重点关注'`
  - 自选股：`'ZXG'`
  - 临时条件股：`'TJG'`

### list_type 参数

- `list_type=0`（默认）：只返回代码列表 → `['000545.SZ', '000629.SZ']`
- `list_type=1`：返回代码和名称 → `[{'Code': '000545.SZ', 'Name': '金浦钛业'}, ...]`

### 示例

```python
# 系统板块（按名称）
codes = tq.get_stock_list_in_sector('煤炭开采')

# 系统板块（按代码）
codes = tq.get_stock_list_in_sector('880301')

# 自定义板块
codes = tq.get_stock_list_in_sector('重点关注', block_type=1)

# 自选股
codes = tq.get_stock_list_in_sector('ZXG', block_type=1)

# 需要股票名称时
codes = tq.get_stock_list_in_sector('通达信88', list_type=1)
# → [{'Code': '000545.SZ', 'Name': '金浦钛业'}, ...]
```

## get_user_sector

获取自定义板块列表。

```python
get_user_sector() -> List  # 返回 [{'Code': 'CSBK', 'Name': '测试板块'}]
```

## send_user_block

添加成份股到自定义板块。

```python
send_user_block(block_code: str = '', stocks: List[str] = [], show: bool = False) -> Dict
```

- block_code 为空则添加到临时条件股
- stocks 为空列表则清空该板块
- 自选股的 block_code 为 'ZXG'

## create_sector / delete_sector / rename_sector / clear_sector

```python
create_sector(block_code: str = '', block_name: str = '')
delete_sector(block_code: str = '')
rename_sector(block_code: str = '', block_name: str = '')
clear_sector(block_code: str = '')
```

## formula_format_data

格式化 get_market_data 的K线数据为公式可用格式。

```python
formula_format_data(data_dict: Dict = {})
```

返回 `{stock_code: [{'Date':..., 'Amount':..., 'Volume':..., 'Close':..., 'Open':..., 'High':..., 'Low':...}]}`

## formula_set_data

手动设置公式参数（K线数据）。

```python
formula_set_data(
    stock_code: str = '',
    stock_period: str = '1d',
    stock_data: List = [],       # formula_format_data 的输出
    count: int = 1,              # 有效K线数量，最大24000
    dividend_type: int = 0       # 0不复权 1前复权 2后复权
)
```

## formula_set_data_info

通过参数自动设置公式数据（推荐）。

```python
formula_set_data_info(
    stock_code: str = '',
    stock_period: str = '1d',
    start_time: str = '',
    end_time: str = '',
    count: int = -1,             # -1全部, -2无序列, 0用时间段, >0截取最新N条
    dividend_type: int = 0
)
```

## formula_get_data

获取当前公式设置中的K线数据。

```python
formula_get_data()  # 返回 {'Code': '...', 'Data': [...]}
```

## formula_zb / formula_xg / formula_exp

调用通达信公式（需先 set_data）。

```python
formula_zb(formula_name: str = '', formula_arg: str = '', xsflag: int = -1)   # 技术指标
formula_xg(formula_name: str = '', formula_arg: str = '')                      # 条件选股
formula_exp(formula_name: str = '', formula_arg: str = '')                     # 专家系统
```

- formula_arg 格式 "arg1,arg2,..."，纯数字，最多16个
- xsflag < 0 返回默认精度，最大8位小数
- formula_zb 返回 `{'Data': {'指标名1': [值列表], '指标名2': [值列表], ...}}`，如 MACD 返回 DIF/DEA/MACD 三组数据
- formula_xg 返回 `{'Data': {'条件名': [值列表]}}`，值为 '1'（满足）或 '0'（不满足）
- formula_exp 返回 `{'Data': {'ENTERLONG': [值列表], ...}}`，值为 '1' 或 '0'

## formula_process_mul_xg / formula_process_mul_zb

批量调用公式（无需提前 set_data）。

```python
formula_process_mul_xg(
    formula_name: str = '',
    formula_arg: str = '',
    return_count: int = 1,       # 每个返回值的返回数
    return_date: bool = False,   # 是否返回日期
    stock_list: List[str] = [],
    stock_period: str = '1d',
    start_time: str = '',
    end_time: str = '',
    count: int = 0,              # K线数量
    dividend_type: int = 0
)

formula_process_mul_zb(
    # 同上，额外参数：
    xsflag: int = -1             # 数据精度
)
```

### count 与 end_time 互斥（重要）

当 `count > 0` 时，`end_time` 参数会被忽略，实际返回最新 N 根 K 线的计算结果。
- 需要指定时间范围：设 `count=0`，同时传 `start_time` + `end_time`
- 需要最新 N 条：设 `count=N`，`start_time`/`end_time` 可忽略

### 返回结构（与单只调用不同）

批量调用返回结构的外层 key 是股票代码，而非 `'Data'`：

```python
# 批量 formula_process_mul_zb 返回：
{
    '600519.SH': {'DIF': [值列表], 'DEA': [值列表], 'MACD': [值列表]},
    '000858.SZ': {'DIF': [值列表], 'DEA': [值列表], 'MACD': [值列表]}
}

# 对比：单只 formula_zb 返回：
{'Data': {'DIF': [值列表], 'DEA': [值列表], 'MACD': [值列表]}}

# 批量 formula_process_mul_xg 返回：
{
    '600519.SH': {'条件名': ['1', '0', '1']},
    '000858.SZ': {'条件名': ['0', '0', '1']}
}
```

### SMA 收敛注意

含 SMA 递归计算的指标（KDJ、RSI、WR 等），count 设置过小会导致初始值不准确。
经验参考：日线 count ≥ 250，周线 count ≥ 120。高精度需求可设 `count=-1` 从头计算（耗时较长）。

## 市场后缀常量

详见 [constants.md](../references/constants.md)。
