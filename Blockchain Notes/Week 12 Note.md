# 金融交易 101

假如你要買入，那必定要有人賣出，反之相同。

[[Order book]] 

根據交易所的種類 ( 中心化 / 分散式 ) 分別有以下特點

中心化交易所，有以下特點
- 較快的處理速度
- 取消交易不需要手續費
- No censorship resistance
- Exchange front running

分散式系統建構於 ( [[Blockchain]] )，有以下特點
- Censorship resistance
- Robust
- 較慢的處理速度
- 交易操作需要手續費
	由於大部分是使用智能合約建構，改動智能合約的狀態需要手續費
- Miner/trader front running

分散式交易所有幾個優點
- No KYC / AML
- 進行交易本身時不需給予交易所手續費
- implement loss

# Automated Market Maker
### Liquidity Pool
使用者可以將自己的資產投入該池子中

## Constant Product AMM
$$ x \times y = c $$
該池子中資產 $x$ 與資產 $y$ 的數量之乘積恆等於常數 $c$ 。

當池子中資產 $x$ 的數量越來越大時，一個單位的資產 $x$ 可以換到的資產 $y$ 的數量就越少

> HW Design AMM for USD / NTD

## AMM 的優點跟缺點
- 不需要維護 [[Order book]]
- 可以使用較簡單的實作方法 ( eg. constant product AMM )
- 容易陷入低流動的困境
- 可能會遭受 [[Samwitch Attack]]

### Stablecoins
### 優點跟缺點



## 問題
- 若 AMM 中的資產脫鉤時該怎麼辦
- 若虛擬貨幣資產被封禁時會怎樣


## AMM Arbitrage

## AMM Aggregator

### Bellman Ford Algorithm
### Theorem Solver ( SMT )










