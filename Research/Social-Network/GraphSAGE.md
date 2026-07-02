#paper

[GraphSAGE: Inductive Representation Learning on Large Graphs (Hamilton, Ying & Leskovec, NeurIPS 2017)](https://arxiv.org/abs/1706.02216)

# 簡介
GraphSAGE(**SA**mple and aggre**G**at**E**)是一種圖節點嵌入方法,核心突破是**inductive(歸納式)**:不再像 [[node2vec]]、[[DeepWalk]] 那樣為每個節點各學一個固定向量,而是學一組**聚合函數(aggregator)**,在推論時**即時**由節點的「特徵 + 局部鄰域」生成嵌入。

兩個關鍵差異:
- **Transductive → Inductive**:[[node2vec]] 等方法一旦圖加入新節點就得整個重訓;GraphSAGE 學到的是「**怎麼從鄰居聚合**」的函數,對**沒見過的節點、甚至整張沒見過的圖**都能直接生成嵌入。
- **善用節點特徵**:[[node2vec]] 只看拓樸結構;GraphSAGE 把節點本身的特徵(屬性、文字、degree 等)當輸入,結構與特徵一起學。

> 一句話:**學的不是「每個節點的向量」,而是「從鄰域生成向量的函數」** —— 這就是它能推廣到未見節點的根本原因。

# 方法

## 核心思想:取樣 + 聚合
每個節點的嵌入,由它的鄰居嵌入「聚合」而來;鄰居的嵌入又由鄰居的鄰居聚合而來。疊 $K$ 層,就能匯集到 $K$ 跳 (hop) 範圍內的資訊。

## 前向傳播(Embedding Generation,Algorithm 1)
輸入:圖 $G$、節點特徵 $\{x_v\}$、層數 $K$、各層聚合函數與權重 $W^k$。初始化 $h_v^0 = x_v$。

對每一層 $k = 1,\dots,K$,每個節點 $v$:
$$
h_{N(v)}^{k} = \text{AGGREGATE}_k\bigl(\{\,h_u^{k-1},\ \forall u \in N(v)\,\}\bigr)
$$
$$
h_v^{k} = \sigma\Bigl(W^{k}\cdot \text{CONCAT}\bigl(h_v^{k-1},\ h_{N(v)}^{k}\bigr)\Bigr)
$$
$$
h_v^{k} \leftarrow h_v^{k} \,/\, \lVert h_v^{k}\rVert_2 \quad(\text{L2 正規化})
$$
最終嵌入 $z_v = h_v^{K}$。

> 直覺:每層做兩件事 —— ①把鄰居的上一層表徵**聚合**成一個向量,②把它和**自己**的上一層表徵接起來,過一層 $W^k + \sigma$ 非線性。L2 正規化穩定訓練。

## 鄰居取樣(Neighborhood Sampling)
若用上**全部**鄰居,計算量會隨高 degree 節點爆炸、且每個 batch 不固定。GraphSAGE 在每一層**均勻採樣固定大小** $S_k$ 的鄰居子集 $N(v)$:
- 讓每個 mini-batch 的計算與記憶體footprint**固定**為 $O\bigl(\prod_{k=1}^{K} S_k\bigr)$。
- 論文常用 $K=2$、$S_1=25$、$S_2=10$(兩跳、合計 $25\times10=250$ 鄰居)。
- 不同層、不同 epoch 重新採樣,帶來類似 dropout 的正則效果。

## 聚合函數(Aggregators)
鄰居是**無序集合**,所以聚合函數必須**對稱(置換不變)**。論文比較三種:

| Aggregator | 公式 / 作法 | 特性 |
| --- | --- | --- |
| **Mean** | 對鄰居向量逐元素取平均 | 最簡單;其變體近似 [[GCN]] 的 inductive 版 |
| **LSTM** | 把鄰居丟進 LSTM | 表達力強,但**非置換不變** → 對鄰居做**隨機排列**後再輸入 |
| **Pooling** | 每個鄰居過一層 MLP,再逐元素 **max-pool**:$\sigma\bigl(\max(\{W_{pool}h_u + b\})\bigr)$ | 兼顧對稱與表達力,**整體表現最佳** |

# 訓練目標
聚合函數與權重 $W^k$ 可用兩種損失端到端學:

## 無監督(graph-based loss)
讓「圖上鄰近」的節點嵌入相似、隨機採樣的負例 (negative sample) 嵌入相異:
$$
J(z_u) = -\log\bigl(\sigma(z_u^{\top} z_v)\bigr) \;-\; Q\cdot \mathbb{E}_{v_n \sim P_n}\bigl[\log\bigl(\sigma(-z_u^{\top} z_{v_n})\bigr)\bigr]
$$
其中 $v$ 是與 $u$ 在固定長度隨機漫步上**共現**的節點,$P_n$ 為負採樣分布,$Q$ 為負例數。形式上與 [[node2vec]] 的 Skip-gram + Negative Sampling 同源,差別在這裡優化的是**聚合函數**而非各節點向量。

## 監督
直接接下游任務(如節點分類)的 cross-entropy,聚合器參數隨任務一起學。

# 與其他方法的關係
- **GCN**:mean aggregator 的變體可視為 [[GCN]] 的 **inductive、mini-batch 化**版本(GraphSAGE-GCN);GCN 原版是 full-batch、transductive。
- **node2vec / DeepWalk**:[[node2vec]] 是 transductive、忽略節點特徵;GraphSAGE 是 inductive、利用特徵 —— 正是 [[node2vec]] 限制一節點名的後繼。
- **Weisfeiler-Lehman**:論文證明 GraphSAGE 與 WL 同構檢驗有理論連結(在特定條件下能逼近節點的 clustering coefficient),說明其結構表達力。

# 實驗
三個 **inductive** 設定的資料集:
- **Citation(Web of Science)**:用部分年份的論文訓練,預測**未來、未見過**的論文節點。
- **Reddit**:貼文社群分類,以較早的貼文訓練、預測後來的貼文。
- **PPI(蛋白質交互作用)**:**multi-graph** 設定 —— 在一批圖上訓練,泛化到**完全沒見過的整張圖**。

結果:**Pooling / LSTM aggregator 最好**;在 inductive 設定下大幅勝過 raw features 與 DeepWalk;且因鄰居採樣,比 transductive 方法**快很多**。

# 限制
- **鄰居一視同仁**:均勻採樣 + 對稱聚合,沒有區分鄰居重要性 → 促成後來 [[GAT]] 用 **attention** 加權鄰居。
- **採樣的隨機性**:固定大小採樣會丟資訊;高 degree 節點只看 $S_k$ 個鄰居。
- **層數淺**:通常 $K=2$,聚合太多層會 over-smoothing(各節點嵌入趨同)。
- **依賴有意義的節點特徵**:inductive 泛化到新節點/新圖時,若特徵品質差,效果會打折。
