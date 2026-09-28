#paper

[node2vec: Scalable Feature Learning for Networks (Grover & Leskovec, KDD 2016)](https://arxiv.org/abs/1607.00653)

# 簡介
node2vec 是一種**圖嵌入（graph embedding）** 方法，目標是學到一個映射
$$
f: V \rightarrow \mathbb{R}^{d}
$$
將圖中每個節點 $v \in V$ 轉成 $d$ 維的特徵向量，使得在圖上「相似」的節點在向量空間中也相近。它延續 [[DeepWalk]] 的概念（隨機漫步 + Skip-gram），但設計了**有偏（biased）隨機漫步**，可以同時兼顧：
- **同質性（homophily）**：同一社群、結構緊密相連的節點要相近 → 偏向 DFS 型探索。
- **結構等價性（structural equivalence）**：在圖中扮演相似角色（例如都是橋樑、中心點）的節點要相近，即使距離很遠 → 偏向 BFS 型探索。

兩個目標往往互相衝突，node2vec 透過參數 $p, q$ 平滑地在兩者之間切換。

# 方法
## 隨機漫步
核心方法為**隨機漫步**，定義長度為 $l$ ，  $c_i$ 表示在漫步中第 $i$ 個節點，由 $c_0$ 開始
$$
P(c_i = x| c_{i-1} = v) = \begin{cases}
\frac{\pi_{vx}}{Z} , &\text{if}(v,x)\in E\\
0, &\text{otherwise}
\end{cases}
$$
其中 $\pi_{vx}$ 為從 $v$ 走到 $x$ 的未正規化轉移機率，$Z$ 為正規化常數。

## 有偏隨機漫步（Biased Random Walk）
若僅考慮邊權重 $w_{vx}$ ，則 $\pi_{vx}=w_{vx}$，這就是一般的加權隨機漫步。node2vec 引入**搜索偏好** $\alpha$，將前一步的節點納入考量：
$$
\pi_{vx} = \alpha_{pq}(t, x)\cdot w_{vx}
$$
假設上一步是 $t \rightarrow v$，現在要從 $v$ 跳到 $x$，定義
$$
\alpha_{pq}(t,x) = \begin{cases}
\dfrac{1}{p}, & \text{if } d_{tx} = 0 \quad (\text{即 } x = t,\text{ 走回頭路})\\[4pt]
1, & \text{if } d_{tx} = 1 \quad (x \text{ 與 } t \text{ 相鄰})\\[4pt]
\dfrac{1}{q}, & \text{if } d_{tx} = 2 \quad (x \text{ 離 } t \text{ 較遠})
\end{cases}
$$
其中 $d_{tx}\in\{0,1,2\}$ 為 $t$ 與 $x$ 在圖上的最短距離。

兩個關鍵超參數：
- **Return parameter $p$**：控制走回上一個節點的機率。$p$ 越大越不容易回頭，鼓勵探索新區域；$p$ 越小越容易在原地附近徘徊。
- **In-out parameter $q$**：控制是要繼續往外（$d_{tx}=2$）還是留在 $t$ 附近（$d_{tx}=1$）。
    - $q < 1$：偏向 DFS，往外走、深入探索 → 捕捉**社群／同質性**。
    - $q > 1$：偏向 BFS，留在 $t$ 的鄰居附近 → 捕捉**結構角色／等價性**。

這個設計巧妙之處在於：$\alpha$ 只依賴 $(t, v, x)$，因此可以**預先計算**每個節點的轉移機率表，採樣時 $O(1)$ 完成。

## 目標函數
延用 Skip-gram 的設定。給定節點 $u$ 與其鄰域 $N_S(u)$（由隨機漫步取樣得到），最大化：
$$
\max_{f}\;\sum_{u\in V}\log P\bigl(N_S(u)\,\big|\,f(u)\bigr)
$$
搭配兩個常見假設：
1. **條件獨立**：$P(N_S(u)\mid f(u)) = \prod_{n_i \in N_S(u)} P(n_i \mid f(u))$。
2. **特徵空間對稱**：以 softmax 建模
$$
P(n_i\mid f(u)) = \frac{\exp\bigl(f(n_i)\cdot f(u)\bigr)}{\sum_{v\in V}\exp\bigl(f(v)\cdot f(u)\bigr)}
$$

合併後得到對所有節點的最佳化目標：
$$
\max_{f}\;\sum_{u\in V}\Biggl[-\log Z_u + \sum_{n_i\in N_S(u)} f(n_i)\cdot f(u)\Biggr]
$$
其中 $Z_u = \sum_{v\in V}\exp\bigl(f(u)\cdot f(v)\bigr)$ 計算成本太高，實作上使用 **Negative Sampling** 近似，並以 SGD 更新。

# 演算法流程
$$
\textbf{LearnFeatures}(G=(V,E,W),\, d,\, r,\, l,\, k,\, p,\, q)
$$

1. 依 $p, q$ 預先計算每個節點的轉移機率，得到改寫後的圖 $G' = (V, E, \pi)$。
2. 對每個節點 $u\in V$ 起始 $r$ 條長度為 $l$ 的有偏隨機漫步 → 收集 walks。
3. 將所有 walks 視為「句子」，丟入 **Skip-gram with Negative Sampling** 訓練 $d$ 維向量。

子程序 **node2vecWalk**：

```
walk = [u]
for i = 1 to l-1:
    curr = walk[-1]
    Vcurr = Neighbors(curr)
    s = AliasSample(Vcurr, π)   # 依預計算的轉移機率採樣
    walk.append(s)
return walk
```

實作上的關鍵小技巧：使用 **Alias Method** 將每次採樣的時間複雜度降到 $O(1)$。

# 與其他方法的關係
- 當 $p = q = 1$ 時，$\alpha \equiv 1$，退化為 [[DeepWalk]] 的均勻隨機漫步。
- 與 [[LINE]] 的「first/second-order proximity」相比，node2vec 的鄰域定義更彈性，可由 $p, q$ 連續調控。
- 與純 BFS（如譜聚類）和純 DFS（如 [[DeepWalk]]）相比，node2vec 是兩者的**插值**。

# 應用
論文中在三類任務上驗證：
- **多標籤分類（multi-label classification）**：BlogCatalog、PPI、Wikipedia 等資料集，學到的 embedding 直接餵給 logistic regression。
- **連結預測（link prediction）**：以節點向量做二元運算（Hadamard、L1、L2、average）作為邊的特徵。
- **延伸**：可以推廣到**邊嵌入（edge embedding）**，例如 $g(u,v) = f(u) \odot f(v)$。

# 超參數的實務建議
- $d$：通常 $128$ 左右；資料夠大可以拉高到 $256$ 或更高。
- $l$（walk length）：$80$ 是常見設定。
- $r$（每節點 walk 數）：$10$。
- $k$（Skip-gram window）：$10$。
- $p, q$：通常在 $\{0.25, 0.5, 1, 2, 4\}$ 內 grid search；以下游任務的驗證集分數挑選。

# 限制
- 屬於 **transductive**：新加入的節點需要重新跑整個流程，無法直接推論未見過的節點（這點促成了後來 [[GraphSAGE]] 等 inductive 方法）。
- 只用到圖的拓樸結構，**忽略節點特徵與邊屬性**。
- 對大型圖，預計算所有 $\alpha$ 仍需要 $O(|E|\cdot \bar{d})$ 的空間（$\bar{d}$ 為平均度）。
