# 财务与交易数据 API

## 目录
- [get_financial_data - 获取专业财务数据](#get_financial_data)
- [get_financial_data_by_date - 获取指定日期财务数据](#get_financial_data_by_date)
- [get_gpjy_value - 获取股票交易数据](#get_gpjy_value)
- [get_gpjy_value_by_date - 获取指定日期股票交易数据](#get_gpjy_value_by_date)
- [get_bkjy_value - 获取板块交易数据](#get_bkjy_value)
- [get_bkjy_value_by_date - 获取指定日期板块交易数据](#get_bkjy_value_by_date)
- [get_scjy_value - 获取市场交易数据](#get_scjy_value)
- [get_scjy_value_by_date - 获取指定日期市场交易数据](#get_scjy_value_by_date)
- [get_gp_one_data - 获取股票单个财务数据](#get_gp_one_data)

## get_financial_data

获取专业财务数据（需先在客户端下载专业财务数据）。

```python
get_financial_data(
    stock_list: List[str] = [],      # 证券代码列表
    field_list: List[str] = [],      # 字段如 'Fn193','Fn194'（不能为空）
    start_time: str = '',            # YYYYMMDD
    end_time: str = '',              # 为空表示无结束限制
    report_type: str = 'report_time' # 'announce_time'按公告日 / 'tag_time'按报告期 / 'report_time'默认
) -> Dict
```

返回 `{stock_code: DataFrame(columns=[字段, announce_time, tag_time])}`。

### 常用财务字段速查

| 字段 | 含义 | 字段 | 含义 |
|------|------|------|------|
| FN1 | 基本每股收益 | FN193 | 成本费用利润率(%) |
| FN4 | 每股净资产 | FN194 | 营业利润率 |
| FN6 | 净资产收益率 | FN197 | 净资产收益率 |
| FN7 | 每股经营现金流 | FN199 | 销售净利率(%) |
| FN40 | 资产总计 | FN200 | 总资产净利率 |
| FN63 | 负债合计 | FN202 | 销售毛利率(%) |
| FN68 | 未分配利润 | FN210 | 资产负债率(%) |
| FN72 | 股东权益合计 | FN230 | 营业收入 |
| FN107 | 经营活动现金流净额 | FN232 | 归母净利润 |
| FN131 | 现金净增加额 | FN183 | 营业收入增长率(%) |
| FN134 | 净利润 | FN184 | 净利润增长率(%) |
| FN159 | 流动比率 | FN206 | 扣非净利润 |
| FN160 | 速动比率 | FN238 | 总股本 |
| FN172 | 应收账款周转率 | FN242 | 股东人数 |
| FN173 | 存货周转率 | FN276 | 近一年净利润 |
| FN175 | 总资产周转率 | FN304 | 研发费用 |

完整字段列表 FN1-FN584，涵盖资产负债表、利润表、现金流量表、财务指标、机构持股等。

## get_financial_data_by_date

获取指定日期的专业财务数据。

```python
get_financial_data_by_date(
    stock_list: List[str] = [],
    field_list: List[str] = [],      # 不能为空
    year: int = 0,                   # 0 表示最新
    mmdd: int = 0                    # 0 表示最新
) -> Dict
```

## get_gpjy_value

获取股票交易数据（需先下载股票数据包）。

```python
get_gpjy_value(
    stock_list: List[str] = [],
    field_list: List[str] = [],      # 如 'GP1','GP3'
    start_time: str = '',
    end_time: str = ''
) -> Dict
```

### 常用股票交易数据字段

| 字段 | 含义 |
|------|------|
| GP01 | 股东人数(户) |
| GP02 | 龙虎榜买入/卖出总计(万元) |
| GP03 | 融资余额(万元) / 融券余量(股) |
| GP04 | 大宗交易成交均价/成交额 |
| GP05 | 增减持成交均价/变动股数 |
| GP06 | 陆股通持股数量(股) |
| GP14 | 涨停金额(万元) / 开板次数 |
| GP15 | 涨跌停状态 / 封单金额 |
| GP16 | 总市值(万元) |
| GP21 | 股息率(%) |
| GP27 | 市场人气排名 / 行业人气排名 |
| GP44 | 股票评分综合评分 |

## get_gpjy_value_by_date

获取指定日期股票交易数据。参数同上，year/mmdd 默认 0 返回最近一条。

## get_bkjy_value

获取板块交易数据。

```python
get_bkjy_value(
    stock_list: List[str] = [],      # 板块代码列表
    field_list: List[str] = [],
    start_time: str = '',
    end_time: str = ''
) -> Dict
```

字段：BK5(市盈率TTM), BK6(市净率), BK9(涨跌家数), BK10(总市值亿), BK12(涨停家数),
BK15(融资融券), BK16(陆股通流入) 等。

## get_bkjy_value_by_date

获取指定日期板块交易数据。year/mmdd 默认 0 返回最近。

## get_scjy_value

获取市场交易数据。

```python
get_scjy_value(field_list: List[str] = [], start_time: str = '', end_time: str = '') -> Dict
```

字段：SC01(融资融券), SC02(陆股通流入), SC03(涨停股个数), SC04(跌停股个数),
SC05-07(股指期货净持仓), SC08(ETF基金规模), SC10(增减持统计), SC16(龙虎榜),
SC30(市场高度/连板) 等。

## get_scjy_value_by_date

获取指定日期市场交易数据。year/mmdd 默认 0 返回最近。

## get_gp_one_data

获取股票单个财务数据（非序列）。

```python
get_gp_one_data(stock_list: List[str] = [], field_list: List[str] = []) -> Dict
```

字段：GO1(发行价), GO3(一致预期目标价), GO5-7(一致预期T/T+1/T+2年EPS),
GO23-25(一致预期PE), GO33(总股本万股), GO36-39(业绩预告归母净利润上下限及同比),
GO42-43(派现/募资总额) 等。
