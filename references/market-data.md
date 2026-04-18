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
) -> Dict  # {field: DataFrame(行=时间, 列=股票代码)}
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
品种类型 HSStockKind：0指数,1A股主板,2北证,3创业板,4科创板,5B股,6债券,7基金,8权证,9其它,10非沪深京。
股本：ActiveCapital(流通万股), J_zgb(总股本万股), J_bg(B股), J_hg(H股)。
财务（万元）：J_zzc(总资产), J_ldzc(流动资产), J_gdzc(固定资产), J_wxzc(无形资产),
J_ldfz(流动负债), J_cqfz(少数股东权益), J_zbgjj(资本公积金), J_jzc(净资产),
J_yysy(营业收入), J_yycb(营业成本), J_yszk(应收账款), J_yyly(营业利润),
J_tzsy(投资收益), J_jyxjl(经营现金净流量), J_zxjl(总现金净流量),
J_ch(存货), J_lyze(利润总额), J_shly(税后利润), J_jly(净利润), J_wfply(未分配利润)。
每股指标：J_jyl(净资产收益率), J_mgwfp(每股未分配), J_mgsy(每股收益/折算全年),
J_mgsy2(季报每股收益), J_mggjj(每股公积金), J_mgjzc(每股净资产), J_gdqyb(股东权益比),
J_gdrs(股东人数), J_HalfYearFlag(报告期月份3/6/9/12), J_start(上市日期)。
行业地域：tdx_dycode/tdx_dyname(地域), rs_hycode_sim/rs_hyname(行业),
blockzscode(行业板块指数代码), underly_setcode/underly_code(标的代码，如ETF跟踪的指数)。

**注意：`field_list` 不能为空，必须明确指定需要的字段。**

## get_more_info — 股票更多信息

```python
get_more_info(stock_code: str = '', field_list: List = []) -> Dict
```

实时指标：ZTPrice(涨停价), DTPrice(跌停价), fHSL(换手率), fLianB(量比), Wtb(委比),
Zsz(总市值亿), Ltsz(流通市值亿), vzangsu(量涨速), Fzhsl(分钟换手率), FzAmo(2分钟金额万)。
涨幅系列：ZAF(当日), ZAFYesterday, ZAFPre2D(前天), ZAFPre5/10/20/30/60,
ZAFYear(年初至今), ZAFPreMyMonth(本月来), ZAFPreOneYear(一年来)。
资金流：Zjl(主买净额万), Zjl_HB(主力净流入万), TotalBVol/TotalSVol(总买卖量),
BCancel/SCancel(总撤买卖量), FCAmo(封单额万), FCb(封成比)。
**注意：资金流字段（Zjl、Zjl_HB、TotalBVol、TotalSVol 等）仅盘中实时有效，盘后返回值全为 0。目前 API 无历史资金流数据接口。**
**涨停跌停判断：用 FCAmo 字段，大于 0 是涨停，小于 0 是跌停。**
封单与竞价：OpenAmo(开盘金额万), OpenZTBuy(竞价涨停买入万), OpenFDE(开盘封单额万),
OpenAmoPre1(昨开盘金额万), CJJEPre1(昨成交额万), FDEPre1/FDEPre2(昨/前封单额万)。
涨停相关：ZTGPNum(板块涨停家数), EverZTCount(连板天), ConZAFDateNum(连涨天数),
YearZTDay(年涨停天数), LastStartZT(几天), LastZTHzNum(几板)。
估值：DynaPE(动态PE), MorePE(港股动态/其他静态), StaticPE_TTM, PB_MRQ, DYRatio(股息率)。
价格：MA5Value, HisHigh/HisLow(52周), IPO_Price, BetaValue(贝塔系数), More_YJL(ETF溢价率)。
股票属性：IsT0Fund, IsZCZGP(注册制A股), IsKzz(可转债), Kzz_HSCode(可转债正股代码)。
财务扩展：FreeLtgb(自由流通万股), KfEarnMoney(扣非净利润万), RDInputFee(研发费用万),
CashZJ(货币资金万), PreReceiveZJ(合同负债万), StaffNum(员工人数)。
近期事件日期：RecentGGJYDate(北上大额交易), RecentHGDate(回购预案), RecentIncentDate(股权激励),
NoticeDate_Recent(业绩预告), RecentReleaseDate(解禁), RecentDZDate(定增),
ReportDate(财报公告), ZTDate_Recent/DTDate_Recent(近2年涨停/跌停),
TopDate_Recent(龙虎榜), StopJYDate_Recent(停牌)。

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

## get_relation — 获取股票所属板块

```python
get_relation(stock_code: str = '') -> List[Dict]
```

获取指定股票所属的全部板块信息。

返回字段：

| 字段 | 说明 |
|------|------|
| BlockCode | 板块代码（无板块代码的板块返回 `"0"`） |
| BlockName | 板块名称 |
| BlockType | 板块类型（行业、地区、概念、风格、指数等） |
| GPNume | 成份股数量 |

示例：
```python
blocks = tq.get_relation(stock_code='688318.SH')
# [{'BlockCode': '881355.SH', 'BlockName': '软件服务', 'BlockType': '行业', 'GPNume': '234'},
#  {'BlockCode': '880592.SH', 'BlockName': '互联金融', 'BlockType': '概念', 'GPNume': '211'},
#  {'BlockName': '沪股通标的', 'BlockType': '风格', 'GPNume': '1763'},
#  {'BlockName': '中证500', 'BlockType': '指数', 'GPNume': '500'}]
```

## get_trackzs_etf_info — 获取跟踪指数的 ETF 信息

```python
get_trackzs_etf_info(zs_code: str = '') -> List[Dict]
```

获取跟踪某一指数的全部 ETF 基金信息。

- `zs_code`：指数代码，如 `'000300.CSI'`（沪深300）

返回字段：

| 字段 | 说明 |
|------|------|
| Code | ETF 代码 |
| Name | ETF 名称 |
| NowPrice | 现价 |
| PreClose | 昨收 |
| IOPV | 净值 |
| Zgb | 净额（万份） |
| Sz | 规模（亿元） |

示例：
```python
etfs = tq.get_trackzs_etf_info(zs_code='000300.CSI')
# [{'Code': '510300.SH', 'Name': '沪深300ETF', 'NowPrice': '4.123', ...}]
```

## get_kzz_info — 获取可转债信息

```python
get_kzz_info(stock_code: str = '', field_list: List[str] = []) -> Dict
```

> 注：官方新版 API 名为 `get_kzz_info`，旧版为 `get_cb_info`，功能相同。

返回字段：KZZCode(可转债代码), HSCode(正股代码), ZGPrice(转股价), ZGRate(转股比率%),
CurRate(当期利率), RestScope(剩余规模万), PutBack(回售触发价), ForceRedeem(强赎触发价),
ZGDate(转股日), EndDate(到期日), EndPrice(到期价), RealValue(纯债价值),
ExpireYield(到期收益率%), KZZScore(可转债评级), HSScore(主体评级),
AGPrice(正股当前价), KZZPrice(可转债当前价), KZZYj(溢价率), ZGValue(转股价值)。

## exec_to_tdx — 调用客户端功能

```python
exec_to_tdx(url: str = '') -> Dict
```

让通达信客户端根据传入的 URL 或功能串执行指定操作。功能串以 `http://www.treeid` 开头。

返回字段：ErrorId, Msg, run_id。

示例：
```python
# 跳转到国内期货主界面
tq.exec_to_tdx(url='http://www.treeid/MAINQH')

# 打开网页链接
tq.exec_to_tdx(url='http://www.treeid/dlghttp://www.tdx.com.cn')
```
