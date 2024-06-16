# Exchange Trading Fee

##### VIP 0

| Exchange | Spot Maker | Spot Taker | Perp Maker | Perp Taker |
| :------: | :--------: | :--------: | :--------: | :--------: |
|   OKX    |   0.08%    |    0.1%    |   0.02%    |   0.05%    |
|  Bybit   |    0.1%    |    0.1%    |   0.02%    |   0.055%   |
|   HTX    |    0.2%    |    0.2%    |   0.02%    |   0.04%    |
|  Biance  |    0.1%    |    0.1%    |   0.02%    |   0.04%    |





# 保证金

### 逐仓 isolated vs 全仓 cross

- 全仓模式下，用户账户里所有的可用资金都视作可用保证金。
- 在逐仓模式下，各个仓位单独核算保证金，盈亏互不影响

### 币本位 vs U本位

- 币本位合约：盈亏和保证金等都以交易的数字货币作为计价单位

  初始保证金 = 面值 * |张数| * 合约乘数 / (开仓均价* 杠杆倍数)，初始保证金固定不变。

  > 假设当前 BTC 价格为$10000，用户希望使用 10 倍杠杆开多 1 BTC 等值的永续合约，用户开多张数=开多 BTC 数量\*BTC 价格 / 面值 = 1\*10,000/100 = 100 张。则 初始保证金=面值\*张数 / (BTC 价格\*杠杆倍数) = 100\*100/（10,000\*10）=0.1 BTC

- U本位合约：盈亏和保证金等都以USDT为单位。

  开仓保证金= 面值 * |张数| * 合约乘数*开仓均价／杠杆倍数，初始保证金固定不变。

  > 假设当前 BTC 价格为 10,000 USDT/BTC，用户希望使用 10 倍杠杆开多 1 BTC 等值的永续合约，用户开多张数 = 开多 BTC 数量 / 面值=1/0.0001=10,000张，则 初始保证金 = 面值\*张数\*BTC 价格/杠杆倍数=0.0001\*10,000\*10,000/10=1,000USDT

比较：

一般都使用 U本位，因为好结算 (如ETH-USDT永续，BTC-USDT永续等)

币本位场景：由于保证金也为现货，相当于自带 1 倍杠杆，上涨行情时可以多赚一倍的钱。

### 可用保证金 vs 占用保证金

- 可用保证金+占用保证金=净值
- 占用保证金：仓位占用的保证金
- 可用保证金：其余的保证金

- 初始保证金率：1/杠杆倍数
- 维持保证金率：也叫最低保证金率，是用户维持当前仓位所需的最低保证金率，是指融资或融券交易完成后，证券行情变化时，为了维护交易双方的利益平衡所确定的保证金
- 
- 保证金率：
  - 全仓单币种保证金率 = (该币种全仓余额 + 全仓收益 - 该币种挂单卖出数量 - 期权买单所需要的该币种数量 - 逐仓开仓所需要的该币种数量 - 所有挂单手续费) / (维持保证金 + 爆仓手续费)。
  - 全仓跨币种保证金率 = 有效保证金 / (维持保证金 + 减仓手续费)
  - 逐仓单币种/跨币种/投资组合保证金 ：
    - 币本位保证金率 = (保证金余额 + 收益) / (面值 * |张数| / 标记价格 * (维持保证金率 + 手续费率))
    - U本位保证金率 = (保证金余额 + 收益) / (面值 * |张数| * 标记价格 * (维持保证金率 + 手续费率))



### 永续期货合约 perpetual swap

也称为永续合约，BitMEX 首创，不同于普通期货合约，永续合约没有到期日。

永续期货合约单位为张，一般10/100/1000张等同一个现货标的物，最小交易单位一般为0.1张

永续合约稳定价格的做法是每8小时让多空双方之间支付资金费。资金费用并不是由交易所收取，而是在多空双方之间转移。若永续合约价格高于现货价格，资金费率为正数，多头将向空头支付资金费用；反之资金费率为负数，空头向多头支付资金费。

资金费按照持仓数量计算。If you close your position prior to the funding exchange then you will not pay or receive funding.

资金费率 Funding Rate (F) 的计算由交易所确定，不同交易所可能略有差别，BitMEX 给出的公式如下：
$$
F = P + \text{Clamp}(I - P, a, b)
$$
- Interest Rate (I) 为币种利率，是一个固定值：
  $$
  I = \frac{\text{Interest Quote Index} - \text{Interest Base Index}}{\text{Funding Interval}}
  $$

  - Funding Interval = 3 (Since funding occurs every 8 hours)
  - Interest Base Index = interest rate for borrowing the Base currency
  - Interest Quote Index = interest rate for borrowing the Quote currency
  - For example, on XBTUSD, the Base currency is XBT while the quote currency is USD.

  > - BitMEX将BTC利率定为年化3%
  > - 有一些交易所，比如 gate.io，将分子统一设定为 0.03% 每日

- Premium Index (P) 为溢价指数：
  $$
  P = \frac{\max(0, \text{Impact Bid Px} - \text{Mark Px}) - \max(0, \text{Mark Px} - \text{Impact Ask Px}) }{\text{Spot Px} + \text{Fair Basis used in Mark Px}}
  $$

  - Impact Bid/Ask Px: 深度加权买卖价，The average filled price to execute the Impact Margin Notional (i.e. 10000USD for perpetual contract) on the Bid/Ask side.
  - Impact Mid Price: average of Impact Bid Price and Impact Ask Price
  - Mark Px: 标记价格，标的物现货在某时间点的市价
  - Index Px: 指数价格，各大交易所上标的物现货交易价格的加权平均。有时也使用该价格替代 mark px
  - Spot Px: 现货价格，似乎等同于 Mark Px
  
- a, b: 资金费变动上下限，一般为 $\pm 0.05\%$. 若 $(I - P) $ 在 $\pm 0.05\%$ 范围内，则 $F = P + (I - P) = I$. 因此长期来看总是多头向空头支付资金费用。

> 不同交易所计算方式不尽相同，如 okx 采用 $F=\text{Clamp}[\text{MA}_{8h}(P - I), a, b]$，这种费率对多空双方都是公平的。





# 质押

"质押"数字货币是指将您的数字货币存入一个特定的钱包或智能合约中，以帮助保护区块链网络，并有可能基于您所质押的数字货币获得奖励。这样做既有助于支持网络的安全性和稳定性，也可以为您带来一定的回报。



# 转账

TRC20是由波场TRON与泰达公司Tether合作发行的稳定币通道，其中的TRC20-USDT相比于传统的Omni-USDT和ERC20-USDT在转账费用和交易确认速度方面都有显著的改进。

收款方若持有 usdt，手续费也会相对较低





# OKX

### websocket address

实盘API交易地址：

- REST：`https://www.okx.com/`
- WebSocket公共频道：`wss://ws.okx.com:8443/ws/v5/public`
- WebSocket私有频道：`wss://ws.okx.com:8443/ws/v5/private`
- WebSocket业务频道：`wss://ws.okx.com:8443/ws/v5/business`

实盘 AWS 地址需要将最低级域名 `ws` 改为 `wsaws`；模拟盘地址则将 `ws` 改为 `wspap`，且模拟盘的请求 header 中需要添加 `"x-simulated-trading: 1"`。

### websocket channel

- 行情频道 tickers（公共）

  - 内容：最新成交价、买一价、卖一价和24小时交易量等
  - 最小间隔： 100ms
  - 触发推送的事件：成交、买一卖一发生变动，不触发不推送

- K线频道 candleXX（公共）

  - 内容：K线数据（1s/1m/1d/...），开盘收盘价格，最高最低价，成交量，……
  - 最小间隔：1s

- 交易频道 trades（公共）

  - 内容：最近的成交数据，同一价格成交的多个订单使用 count 字段聚合
  - 最小间隔：无
  - 触发推送的事件：有成交数据就推送，每次推送可能聚合多条成交数据。

- 全部交易频道 all-trades（业务）

  - 内容：最近的成交数据，每次推送仅包含一条成交数据
  - 最小间隔：无
  - 触发推送的事件：有成交数据就推送，每次推送仅包含一条成交数据。

- 深度频道 booksXX（公共）

  - 内容：深度数据。每一档包含价格，数量，订单数。除一档/五档全量推送外，其余都是增量推送

    > bbo-tbt: best bid offer 频道，非大户只能订阅这个
    >
    > books50-l2-tbt: 50档深度频道，只允许交易手续费等级 VIP4及以上的 API 用户订阅.
    >
    > books-l2-tbt: 400档深度频道，只允许交易手续费等级 VIP5及以上的 API 用户订阅。

  - 最小间隔：tbt 频道 10ms，其他频道 100ms

  - 触发事件：当频道对应的深度快照有变化时推送，不触发不推送

