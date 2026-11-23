# GitHub 加密交易资源清单 + ccxt / tradoor 实测报告

整理与实测：Muse，2026-11-23（Asia/Shanghai）。星数为 2026-11-23 经 GitHub 搜索实测值。
背景：用户想达到币安带单员「熬鹰资本（allincrypto 熬鹰者）」的做单水平。先行结论：他在 GitHub 上无任何策略/代码（已搜 thankUcrypto / allincrypto，只有粉丝自制 0 星监控工具）；他的 edge 是盘感+仓位管理，不可下载。可复制的是数据完整度、统计纪律、仓位分级——以下资源就是搭这套系统的零件。

## 一、资源清单（按层）

### 1. 数据层
- ccxt/ccxt（44,961★）https://github.com/ccxt/ccxt — 100+ 交易所统一 API：K线/OI/资金费率/多空比/盘口/逐笔，一个库全拿。自动化「12 项数据清单」的骨架。
- 币安官方历史数据（data.binance.vision）：合约 K线/逐笔/资金费率全历史免费下载，回测原料仓。GitHub 上的下载器仓库普遍很小（最高 15★），用官方源即可。
- 币安强平推送 forceOrder（WebSocket 免费原始流）：CoinGlass 清算数据的上游，清算层自建靠它。

### 2. 订单流层
- tradoor-terminal/tradoor（320★）https://github.com/tradoor-terminal/tradoor — Footprint/TPO/DOM/真实清算一体的开源终端，有官方在线版 tradoorterminal.com。详见下方实测。
- DegenSugarBoo/OpenBook（195★）— 深度热力图 + 订单流（Rust）。
- zadescoxp/order-flow-screener（276★）— 现货/永续订单流筛选器。
- ianfigueroa/TapeFlow（21★）— tape/footprint/DOM/CVD/清算终端（C++）。
- 注意：清算热力图（预测价位模型）在 GitHub 无成熟开源替代，搜到的均为 0–1 星个人项目。

### 3. 回测与执行框架
- freqtrade/freqtrade（57,172★）+ freqtrade/freqtrade-strategies（5,675★）— 币圈回测事实标准。
- jesse-ai/jesse（8,828★）；hummingbot/hummingbot（21,357★）；nkaz001/hftbacktest（5,128★，订单簿级回测）；kernc/backtesting.py（9,281★）；Superalgos/Superalgos（5,816★）。

### 4. 资源索引
- paperswithbacktest/awesome-systematic-trading（15,518★）
- je-suis-tm/quant-trading（11,336★）
- thuquant/awesome-quant（5,741★，中文）

## 二、ccxt 实测（2026-11-23）

安装：pip 一次成功，版本 4.5.93。

数据完整度（以 OKX 公开 API 实拉验证 — ccxt 封装的正是这些端点）：
- ✅ 永续价格 61,256.7（BTC-USDT-SWAP，与当日行情一致）
- ✅ 资金费率 0.0029%
- ✅ OI 31,297.99 BTC
- ✅ 多空账户比（1H）2.13；主动买卖量（taker buy/sell）有数
- ✅ K线、盘口、逐笔成交全部正常

环境坑（必须如实记录）：
1. 币安合约 API（fapi.binance.com）从本机出口被地域限制：HTTP 451。Bybit 403。OKX 200 可用；币安现货公开数据域（data-api.binance.vision）200 可用。
2. ccxt 的 HTTP 层在本机沙箱与出口代理有 TLS 兼容问题（SSL WRONG_VERSION_NUMBER），同样的端点用 requests 直连全部成功。这是本机环境问题，不是 ccxt 本身的缺陷（在普通 VPS/本地机器上 ccxt 是行业标准）。结论：库可用；若在本环境落地，需绕过其默认 HTTP 层或换运行环境。

对照 12 项清单（ccxt/公开 API 路线）：OI✅ 资金费率✅ 多空比✅ 合约成交量✅ CVD 原料（逐笔）✅ 盘口原料✅；清算热力图❌（只有原始强平流，需自建模型）币安聪明钱❌ 现货 ETF 资金流❌（需另接数据源）Footprint❌（需自算，见 tradoor）。

## 三、tradoor 实测（2026-11-23，源码通读 + 克隆；启动被环境安全检查拦截，未实际运行，有官方在线版可直接体验）

- 技术栈很重：Rust + PostgreSQL + SolidJS/WASM；自托管门槛高，建议先用在线版 tradoorterminal.com。
- 数据源：浏览器直连 15 家交易所公开流，不需要 API key；**没有实盘下单功能**（只有纸面训练账户和交易日志）。
- 对照 12 项：有 4 项（OI、真实清算量、合约成交量、Footprint——且是强项）；部分 2 项（资金费率无独立面板、CVD 有 delta/flow 但无独立累积曲线）；没有 6 项（清算热力图、多空比、订单簿热力图、币安聪明钱、CoinGlass 综合、ETF 资金流）。
- 定位：补「微观结构/订单流」这一半，与 CoinGlass 的聚合统计互补，不能替代现有 12 项清单。
- 坑：Footprint 历史只覆盖 BTC/ETH × Binance/Bybit × 现货/永续；清算记录默认只跟 BTC/ETH/SOL；许可证 AGPL-3.0（自用无碍）。

## 四、建议路线（复制熬鹰可复制的部分）

1. 数据：ccxt（或直连 API）把 12 项清单自动化，告别手动点网站。
2. 验证：freqtrade/jesse 把自己的打法回测成统计（胜率/盈亏比/最大回撤），对标他的公开口径（胜率 67%、183 笔、最大回撤 68%——注意他的回撤，学纪律不学杠杆）。
3. 复盘：交易日志按 setup 分类统计，像他那样笔笔有账。
4. 订单流手感：tradoor 在线版直接上手，零部署。
