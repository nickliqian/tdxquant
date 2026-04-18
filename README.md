# TdxQuant Skill

TdxQuant（通达信量化）全能开发助手。基于 tqcenter Python API 帮助用户完成量化策略开发全流程。

**简单的策略脚本基本可以一次成功，如果生成的有问题，可以联系我帮忙优化skills。**

## 功能特性

- 📊 **数据获取** — K线、快照、财务、板块成份股、可转债、ETF
- 🎯 **选股策略** — 技术指标、基本面筛选、行业轮动、公式调用
- 📈 **回测验证** — 配合 vectorbt 进行策略回测，支持 SIGNALS_TQ 公式展示
- 🔔 **实时监控** — 行情订阅、条件预警、分批订阅
- 📦 **板块管理** — 自定义板块、批量操作
- 🧮 **公式调用** — 调用通达信内置公式（技术指标/条件选股/专家系统）
- 💰 **交易下单** — 账户查询、买卖下单、委托查询、撤单（实盘/模拟盘）

## 环境要求

- Python 3.7+（建议 3.13），64位
- 通达信金融终端（支持 TQ 策略功能）
- 依赖包：
```bash
pip install numpy pandas -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install vectorbt -i https://pypi.tuna.tsinghua.edu.cn/simple  # 回测用，可选
```

## 快速开始

### 1. 安装 Skill

**Kiro IDE（推荐）**
```bash
# 将 skill 目录放到项目的 .kiro/skills/ 下
cp -r tdxquant /your-project/.kiro/skills/
```

**全局安装**
```bash
# Kiro 全局
cp -r tdxquant ~/.kiro/skills/

# Claude Code 全局
cp -r tdxquant ~/.claude/skills/
```

**从 Gitee 克隆**
```bash
git clone git@gitee.com:nickliqian/tdxquant.git
```

验证安装：在对话中提到"量化策略"、"选股"、"回测"、"下单"等关键词时，skill 会自动激活。

### 2. 初始化连接

```python
import sys, os, winreg

# 自动获取通达信安装目录
key_path = r"SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\通达信金融终端64"
with winreg.OpenKey(winreg.HKEY_LOCAL_MACHINE, key_path) as key:
    tdx_root, _ = winreg.QueryValueEx(key, "InstallLocation")

sys.path.insert(0, os.path.join(tdx_root, 'PYPlugins', 'user'))

from tqcenter import tq
tq.initialize(__file__)
```

### 3. 编写策略

![金叉策略](tdxpng/1.png)
![涨停选股](tdxpng/2.png)

## 文件结构

```
tdxquant/
├── SKILL.md                          # 主文件（策略模式 + 陷阱 + 开发流程）
├── README.md                         # 本文件
└── references/
    ├── market-data.md                # 行情与基础数据 API
    ├── financial-data.md             # 财务与交易数据 API（FN/GP/BK/SC/GO）
    ├── sector-formula.md             # 板块管理与公式调用 API
    ├── trading-order.md              # 交易下单 API
    ├── subscribe-notify.md           # 订阅、通知与 SIGNALS_TQ 公式系统
    └── constants.md                  # 常量枚举（市场后缀/复权/周期/交易常量）
```

## 策略模式速览

SKILL.md 中包含 8 个端到端策略模式：

| 模式 | 说明 |
|------|------|
| 选股入板块 | 技术指标选股 → 写入自定义板块 |
| 回测策略 | 配合 vectorbt 回测，支持可视化 |
| 调用通达信公式 | 单只/批量调用技术指标和条件选股公式 |
| 实时监控与预警 | 订阅行情 + 条件触发 + 发送预警信号 |
| 财务选股 | ROE/营收增长/资产负债率等基本面筛选 |
| 行业轮动 | 板块涨幅排名 → 成份股选股 |
| 交易下单 | 完整六步流程：句柄→资产→持仓→下单→委托→撤单 |
| 选股+自动下单 | 信号选股 → 资金分配 → 批量下单 |

## 关键约束

- `get_market_data` 单次最多 24000 条
- `subscribe_hq` 最多订阅 100 只股票
- `send_warn` 的 reason_list 每个元素最多 25 汉字；参数名是 `volum_list`（非 volume_list）
- 复权类型：行情 API 用 `'none'`/`'front'`/`'back'`，公式 API 用 `0`/`1`/`2`
- 周期：`1m` `5m` `15m` `30m` `1h`/`60m` `1d` `1w` `1mon` `1q` `1hy` `1y` `tick`
- 交易下单：`stock_account()` 返回值 ≥ 0 有效；实盘 Value=1（待确认），模拟盘 Value=2（直接成功）
- 批量公式 count > 0 时 end_time 被忽略；批量返回外层 key 是股票代码而非 `'Data'`

## 更新记录

| 版本 | 日期 | 主要变更 |
|------|------|---------|
| v1.6 | 2026-04 | 对标官方文档补充：初始化路径配置、故障排除、4个缺失API、SIGNALS_TQ公式系统、字段完整分类 |
| v1.5 | 2026-04 | 新增交易下单模块（6个API + 2个策略模式 + 交易陷阱） |
| v1.4 | 2026-04 | 公式调用实战踩坑优化（count/end_time互斥、返回结构差异、SMA收敛） |
| v1.3 | 2026-04 | 补充板块参数说明、行业轮动模式、可视化引导 |
| v1.2 | 2025-04 | 合并三个子skill为一个，增加财务选股示例 |
| v1.1 | 2025-04 | 用户审查反馈修复（10+项） |
| v1.0 | 2025-04 | 首次创建 |

## 许可证

MIT License
