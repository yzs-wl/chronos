# Chronos · 时策 A股量化交易模拟系统

> Master Time, Master Trade.

Chronos（代号"时策"）是一个面向 A 股的**量化交易模拟系统**，覆盖行情中心、策略库、回测引擎、参数优化、模拟交易、实时模拟、推理日志、数据导入、标的管理、每日精选、信号扫描、回测对比等完整链路。后端基于 Spring Boot + JPA + H2，前端基于 Vue 3 + Vite + Pinia + Tailwind + ECharts，前端构建产物直接托管在 Spring Boot 静态资源目录，可单体部署。

---

模块以希腊神祇命名，职责如下：

| 包（模块） | 职责 |
|---|---|
| **mnemosyne**（记忆女神） | 行情中心、标的管理、K 线、交易日历、股息、数据导入与联网同步 |
| **athena**（智慧女神） | 策略接口、策略注册、技术指标、信号扫描 |
| **prometheus**（普罗米修斯） | 回测引擎、撮合、组合、推理日志（Clio）、保存的回测 |
| **aegis**（神盾） | 风控预检：仓位 / 回撤 / 集中度（T+1、卖空由账户层强制） |
| **apollo**（阿波罗） | 参数优化（网格 / 贝叶斯）+ 绩效分析 |
| **hermes**（赫尔墨斯） | 实时模拟引擎、券商适配器（模拟 / 桩） |
| **hemera**（赫墨拉） | 每日精选：自动选股模拟、因子配置、盈亏复盘 |

---

## 功能模块

| 功能 | 前端页面 | 后端接口 |
|---|---|---|
| 概览 | `DashboardView` `/` | 综合 |
| 行情中心 | `MarketView` `/market` | `/api/market`（行情/K线/实时报价/日历/股息） |
| 策略库 | `StrategiesView` `/strategies` | `/api/strategies` |
| 回测引擎 | `BacktestView` `/backtest` | `/api/backtest`（run/runs/trades） |
| 参数优化 | `OptimizationView` `/optimize` | `/api/optimize`（网格/贝叶斯） |
| 模拟交易 | `PaperTradingView` `/paper` | `/api/live/paper/*`（手动模拟下单） |
| 实时模拟 | `LiveView` `/live` | `/api/live` + WebSocket（逐 tick 推演策略） |
| 推理日志 | `TradeLogView` `/tradelog` | `/api/backtest/trades`（按条件查询 / 清空） |
| 数据导入 | `DataImportView` `/data-import` | `/api/market/import｜generate｜sync｜clear` |
| 标的管理 | `SymbolView` `/symbols` | `/api/market/symbols*` |
| 每日精选 | `DailyPickView` `/daily-pick` | `/api/daily-pick`（WebSocket `/topic/daily-pick`） |
| 信号扫描 | `SignalScanView` `/signal-signal-scan` | `/api/signal-scan`（下交易日买卖建议） |
| 回测对比 | `CompareView` `/compare` | `/api/backtest/saved` |

### 内置策略（`IStrategy` 实现，共 5 个）
- **双均线交叉** `dual_ma`：快慢均线金叉买入、死叉卖出
- **MACD 趋势** `macd`：DIF 上穿/下穿 DEA 金叉/死叉
- **RSI 均值回归** `rsi_reversion`：RSI 超卖买入、超买卖出
- **MA+RSI 组合** `ma_rsi`：均线多头 + RSI 未超买买入；死叉/超买卖出
- **布林带均值回归** `bollinger`：触下轨买入、触上轨/中轨卖出

指标工具 `athena/indicators/Indicators` 提供 MA / MACD / RSI / BOLL 等。
---

## 备注
- A 股特色约束（T+1、涨跌停、禁止卖空、ST 处理）在 `common/AShareValidator` 与 `aegis/RiskManager` 中实现。
- 实时模拟的行情为**合成随机游走**（用于验证系统管线与策略信号逻辑），不代表真实行情；判断策略适配性与买卖信号请使用**信号扫描**或**历史回测**。
- 代理前缀：前端以相对路径 `BASE=''` 经 Vite 代理转发 `/api` 与 `/ws` 到后端 8080。
