# 行情与基础数据 API 详细参考

## get_market_data — 获取K线行情

```python
get_market_data(
    field_list: List[str] = [],      # 字段筛选，空返回全部
    stock_list: List[str] = [],      # 证券代码列表
    period: str = '',                # 1m/5m/15m/30m/60m/1d/1w/1mon/1q/1hy/tick
    start_time: str = '',            # YYYYMMDD
    end_time: str = '',              # YYYYMMDD
    count: int = -1,                 # 每只股票返回条数
    dividend_type: str = None,       # none/front/back
    fill_data: bool = True           # 是否向后填充空缺数据（如停牌时用前值填充）
) -> Dict  # {field: DataFrame(index=stock_list, columns=time_list)}
```

默认返回字段：Date, Time, Open, High, Low, Close, Volume, Amount, ForwardFactor。
ForwardFactor 仅 dividend_type='none' 时有效。单次最多 24000 条。

示例：
```python
df = tq.get_market_data(stock_list=['688318.SH'], start_time='20251220', count=1, dividend_type='none', period='1d')
```

## get_market_snapshot — 获取最新快照

```python
get_market_snapshot(stock_code: str, field_list: List = []) -> Dict
```

返回字段：ItemNum, LastClose, Open, Max, Min, Now, Volume, NowVol, Amount, Inside, Outside,
TickDiff, InOutFlag(0Buy/1Sell/2Unknown), Jjjz(基金净值),
Buyp[5], Buyv[5], Sellp[5], Sellv[5](五档盘口),
UpHome, DownHome(指数涨跌家数), Before5MinNow, Average, XsFlag, Zangsu, ZAFPre3。

## get_stock_info — 证券基本信息

```python
get_stock_info(stock_code: str, field_list: List = []) -> Dict
```

基础字段：Name, Unit, VolBase, MinPrice, XsFlag, Fz[8](开收市时间), DelayMin。
属性标志：BelongHS300, BelongRZRQ, BelongHSGT, IsHKGP, IsQH, IsQQ, IsSTGP, IsQuitGP, TodayDRFlag。
品种类型 HSStockKind：0指数,1A股主板,2北证,3创业板,4科创板,5B股,6债券,7基金,8权证。
股本：ActiveCapital(流通万股), J_zgb(总股本万股), J_bg(B股), J_hg(H股)。
财务：J_zzc(总资产), J_jzc(净资产), J_yysy(营业收入), J_jly(净利润), J_mgsy(每股收益),
J_mgjzc(每股净资产), J_jyl(净资产收益率), J_wfply(未分配利润), J_gdrs(股东人数)。
其他：J_start(上市日期), tdx_dyname(地域), rs_hyname(行业), blockzscode(行业板块指数代码)。

## get_more_info — 股票更多信息

```python
get_more_info(stock_code: str = '', field_list: List = []) -> Dict
```

实时指标：ZTPrice(涨停价), DTPrice(跌停价), fHSL(换手率), Zsz(总市值亿), Ltsz(流通市值亿)。
涨幅系列：ZAF(当日), ZAFYesterday, ZAFPre5/10/20/30/60, ZAFYear, ZAFPreOneYear。
资金流：Zjl(主买净额万), Zjl_HB(主力净流入万), TotalBVol/TotalSVol, FCAmo(封单额万)。
估值：DynaPE(动态PE), StaticPE_TTM, PB_MRQ, DYRatio(股息率)。
涨停：EverZTCount(连板天), YearZTDay(年涨停天数), ConZAFDateNum(连涨天数)。
其他：MA5Value, HisHigh/HisLow(52周), IPO_Price, FreeLtgb(自由流通万股), IsKzz(是否可转债)。

## get_divid_factors — 分红配送

```python
get_divid_factors(stock_code: str, start_time: str, end_time: str) -> pd.DataFrame
```

列：Type(1除权除息/11扩缩股/15重新调整), Bonus, AllotPrice, ShareBonus, Allotment。

## get_ipo_info — 新股申购

```python
get_ipo_info(ipo_type: int = 0, ipo_date: int = 0)
```

ipo_type: 0新股, 1新发债, 2全部。ipo_date: 0今天, 1今天及以后。

## get_gb_info — 股本数据

```python
get_gb_info(stock_code: str = '', date_list: List[str] = [], count: int = 1)
```

返回 `[{Date, Zgb(总股本), Ltgb(流通股本)}]`。date_list 须从小到大。

## get_cb_info — 可转债信息

```python
get_cb_info(stock_code: str = '', field_list: List[str] = [])
```

字段：HSCode(正股代码), ZGPrice(转股价), ZGRate(转股比例), CurRate(票面利率),
EndDate(到期日), EndPrice(到期赎回价), ForceRedeem(强赎价), PutBack(回售价),
RestScope(剩余规模万元), HSScore/KZZScore(评级)。

## get_trading_dates — 交易日列表

```python
get_trading_dates(market: str, start_time: str, end_time: str, count: int = -1) -> List
```

market 暂固定 'SH'。需先下载上证指数盘后数据。

## refresh_cache — 刷新行情缓存

```python
refresh_cache(market: str = 'AG', force: bool = False)
```

market: 'AG'A股, 'HK'港股, 'US'美股, 'QH'期货, 'QQ'期权, 'NQ'新三板, 'ZZ'中证国证。
force=False 时距上次不足10分钟不刷新。

## refresh_kline — 刷新历史K线

```python
refresh_kline(stock_list: List[str] = [], period: str = '')
```

period 仅支持 '1d', '1m', '5m'。盘中分钟线只到上个交易日。

## download_file — 下载数据文件

```python
download_file(stock_code: str = '', down_time: str = '', down_type: int = 1)
```

down_type: 1十大股东(年份), 2ETF申赎清单(日期), 3最近舆情, 4综合信息。

## price_df — 辅助函数：转换为宽表

```python
tq.price_df(data_dict: Dict, field: str, column_names: List[str] = []) -> pd.DataFrame
```

将 `get_market_data` 返回的 dict 中指定字段转为 DataFrame（index=日期, columns=股票代码）。
常用于后续信号计算。

示例：
```python
df = tq.get_market_data(field_list=['Close'], stock_list=codes, period='1d', ...)
close_df = tq.price_df(df, 'Close', column_names=codes)
```

## get_full_tick — 获取完整 Tick 数据

```python
tq.get_full_tick(stock_code: str) -> Dict
```

获取指定股票的完整 tick 级别数据。返回字段与 `get_market_snapshot` 类似，包含 Now（最新价）、LastClose（昨收）、Volume（成交量）等。具体字段以实际返回为准，常用于订阅回调中获取最新行情。

与 `get_market_snapshot` 的区别：`get_full_tick` 在订阅回调场景下更常用，返回数据更贴近实时 tick 级别。
