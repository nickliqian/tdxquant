# 订阅与通知 API 详细参考

## subscribe_hq — 订阅行情更新

```python
subscribe_hq(stock_list: List[str] = [], callback=None)
```

- 最多订阅 100 条
- callback 格式：`def on_data(data_str)` → data_str 为 JSON `{"Code":"XXXXXX.XX","ErrorId":"0"}`
- 策略需保持运行（while True + sleep）才能持续接收回调

## unsubscribe_hq — 取消订阅

```python
unsubscribe_hq(stock_list: List[str] = [])
```

## get_subscribe_hq_stock_list — 获取订阅列表

```python
get_subscribe_hq_stock_list() -> List  # 如 ['600519.SH']
```

## send_warn — 发送预警信号

```python
send_warn(
    stock_list: List[str] = [],        # 证券代码列表
    time_list: List[str] = [],         # 时间 YYYYMMDDHHMMSS
    price_list: List[str] = [],        # 现价（纯数字字符串）
    close_list: List[str] = [],        # 收盘价
    volum_list: List[str] = [],        # 成交额（注意：参数名为 volum_list，非 volume_list）
    bs_flag_list: List[str] = [],      # 0买 1卖 2未知（不足count自动补2）
    warn_type_list: List[str] = [],    # 0常规预警
    reason_list: List[str] = [],       # 预警原因（每个最多25汉字/50英文）
    count: int = 1                     # 有效数据个数
) -> Dict
```

所有 list 元素一一对应。同一只股票多次预警需在 stock_list 中重复添加。

## send_message — 发送消息到客户端

```python
send_message(msg_str: str) -> Dict
```

用 `|` 分行显示，`\n` 也可换行。

## send_file — 发送文件到客户端

```python
send_file(file: str) -> Dict
```

- 文件在 `.\PYPlugins\file\` 下可只传文件名
- 其他位置需传绝对路径
- 支持 txt, pdf, html

## send_bt_data — 发送回测数据

将回测结果发送到通达信客户端的"TQ策略数据浏览"中展示，用于可视化回测的买卖点、收益曲线等自定义数据。

```python
send_bt_data(
    stock_code: str = '',              # 单只股票代码
    time_list: List[str] = [],         # YYYYMMDDHHMMSS
    data_list: List[List[str]] = [],   # 二维List，每个子List最多16个纯数字
    count: int = 1                     # 有效数据个数
) -> Dict
```

示例：
```python
tq.send_bt_data(
    stock_code='688318.SH',
    time_list=['20251215141115'],
    data_list=[['11']],
    count=1
)
```

## print_to_tdx — 导出数据到客户端

```python
print_to_tdx(
    df_list: list[pd.DataFrame] = [],  # 每个DataFrame第一列为日期
    sp_name: str = "",                 # .sp文件名前缀
    xml_filename: str = "",            # .xml文件名（建议必填）
    jsn_filenames: list[str] = None,   # 每组对应的.jsn文件名
    vertical: int = None,              # 纵向排列组数
    horizontal: int = None,            # 横向排列组数（优先级高于vertical）
    height: list[str|float] = None,    # 每组高度占比(0-1)，支持 float 如 0.3 或 str 如 "0.3"，两种写法等效
    table_names: list[str] = None      # 每组面板标题
) -> None
```

df_list 和 jsn_filenames 长度必须与 vertical/horizontal 指定的组数一致。

## SIGNALS_TQ 公式系统

通过 `send_bt_data` 发送序列数据后，可在通达信公式管理器中创建技术指标公式，使用 `SIGNALS_TQ(ID, TYPE)` 函数引用数据并在 K 线图上展示。

### 函数语法

```
SIGNALS_TQ(ID, TYPE)
```

| 参数 | 说明 |
|------|------|
| ID | data_list 子列表中的位置序号（1~16） |
| TYPE | 0=不平滑处理；1=平滑（无数据周期返回上一周期值）；2=无数据则为 0 |

### 通达信公式示例

在通达信公式管理器中新建公式：

```
{技术指标}
MA5:SIGNALS_TQ(1,0);
MA10:SIGNALS_TQ(2,0);

{交易信号}
BUY_SIGNAL:=SIGNALS_TQ(3,0);
SELL_SIGNAL:=SIGNALS_TQ(4,0);

{绘制交易信号图标}
DRAWICON(BUY_SIGNAL, LOW, 1);
DRAWICON(SELL_SIGNAL, HIGH, 2);
```

### 配合 send_bt_data 使用

```python
# 发送 MA5、MA10 和买卖信号
tq.send_bt_data(
    stock_code='688318.SH',
    time_list=['20260120', '20260121'],
    data_list=[
        ['136.60', '131.74', '1', '0'],   # 位置1=MA5, 2=MA10, 3=买信号, 4=卖信号
        ['135.30', '131.48', '0', '1']
    ],
    count=2
)
# 在通达信中用 SIGNALS_TQ(1,0)~SIGNALS_TQ(4,0) 引用上述数据
```
