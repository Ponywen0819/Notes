#note

# 簡介

**The Open Network (TON)** 是由 Telegram 團隊於 2018 年提出的高吞吐量公有鏈，原名 *Telegram Open Network*，後因 SEC 監管問題由社群以 *The Open Network* 名義繼續維護。

![ton_operate_system](image/ton_operating_system.png)

主要特點：
- **Multi-chain（多鏈架構）**：採用層級式分片設計，可水平擴展至理論上 $2^{60}$ 條鏈。
- **同質 又 異質**：
    - **異質 (heterogeneous)**：每條 work-chain 可以採用不同的協定（不同的虛擬機、共識規則、代幣經濟）。
    - **同質 (homogeneous)**：同一條 work-chain 底下分裂出的 shard-chain 必然繼承父鏈協定。
- **Instant Hypercube Routing**：跨分片訊息可在 $O(\log n)$ 跳數內路由完成。
- **PoS + BFT 共識**：驗證者抵押 TON 後參與 Byzantine Fault Tolerant 投票。

# TON 的鏈層級結構

TON 採三層階層式分片：

```
                Master-chain
                    │
     ┌──────────────┼──────────────┐
 Work-chain #0  Work-chain #1   Work-chain #N      ← 不同協定
     │              │
 ┌───┴───┐      ┌───┴───┐
Shard   Shard  Shard   Shard                       ← 動態分裂/合併
  │       │
 acc     acc                                       ← 帳號邏輯鏈（虛擬）
```

## Master-chain
整個網路的根鏈，負責記錄：
- 所有 work-chain 與 shard-chain 的當前狀態雜湊。
- 驗證者集合與 PoS 抵押資訊。
- 全域協定參數。

Master-chain 區塊一旦敲定，所有子鏈狀態就具有最終性。

## Work-chain
獨立的「子網路」，最多可有 $2^{32}$ 條。每條 work-chain 可自訂：
- 虛擬機（TON 預設使用 TVM；其他 work-chain 可換成 EVM 或自製 VM）。
- 帳戶模型、交易格式、手續費機制。

由於彼此獨立，因此稱為「異質」。

## Shard-chain
work-chain 內的水平分片。每條 shard-chain 處理一部分帳戶的交易，分片規則由帳戶位址前綴決定。

特性：
- **同質性**：同一 work-chain 的所有 shard-chain 共用協定，可無縫遷移帳戶。
- **動態分裂 / 合併**：根據負載自動調整分片數量。
    - 負載高 → 一條 shard-chain 分裂為兩條，每條負責一半帳戶位址空間。
    - 負載低 → 兩條相鄰 shard-chain 合併。
- 初始狀態下，每個 work-chain 只有 **1 條** shard-chain，等流量上升才會分裂。

## Account-chain
最底層的「帳號鏈」——只記錄**單一帳號**的交易歷史。

重要：account-chain **並非實體存在**，而是邏輯上的概念——它是從 shard-chain 中濾出該帳號相關交易後虛擬出的鏈。實際儲存與共識仍在 shard-chain 上完成。

# 動態分片 (Dynamic Sharding)

當某條 shard-chain 的交易量超出單組驗證者的處理能力時，協定會觸發分裂：

1. 將該 shard-chain 負責的帳戶位址空間二分（依位址首位 bit）。
2. 為每一半各自指派新的驗證者組。
3. 新區塊起，兩條 shard-chain 並行運作，跨片交易由 hypercube routing 處理。

當負載下降，相鄰 shard-chain 可反向合併，回收驗證者資源。這個機制讓 TON 能根據實際使用量自動調整吞吐量。

# 跨分片訊息

TON 採 **Instant Hypercube Routing**：每條 shard-chain 之間透過 hypercube 拓樸連接，任意兩條鏈間的訊息只需 $O(\log N)$ 跳即可送達。每筆跨片交易帶有目標位址與路由標頭，由中繼分片轉發直到抵達目的分片。

這讓 TON 可避免傳統分片鏈中常見的「跨片交易延遲爆炸」問題。

# 共識協定

TON 使用 **Catchain**——一個基於 BFT 的多輪投票協定：
- 每個 shard-chain 由一組驗證者（validator）負責出塊。
- 驗證者由 master-chain 上的 PoS 抵押與隨機抽選共同決定。
- 每輪選出 leader 提案區塊，其他驗證者投票，達 $\geq 2/3$ 即敲定。
- 拜占庭容錯界 $f < n/3$，與本課程介紹的 [[Week 10 Note|拜占庭傳播下界]]一致。

# 與其他公鏈的比較

| 維度 | TON | Ethereum | Solana |
| --- | --- | --- | --- |
| 分片架構 | 動態分片（多層） | Rollup-centric（外掛 L2） | 單鏈（無分片） |
| 共識 | BFT (Catchain) + PoS | Gasper (Casper FFG + LMD-GHOST) | PoH + Tower BFT |
| TPS（理論） | 百萬級（多分片並行） | 15–30（L1） | 50,000+ |
| 智能合約 VM | TVM（TL-B/Fift/FunC） | EVM（Solidity） | Sealevel（Rust） |
| 最終性 | 數秒（分片） + master-chain 敲定 | ~12 分鐘 | ~13 秒 |

# 參考
- TON Whitepaper: <https://docs.ton.org/learn/overviews/ton-blockchain>
- 課程筆記：[[Week 10 Note]]、[[Blockchain]]
