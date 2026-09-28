#paper

[GCN: Semi-Supervised Classification with Graph Convolutional Networks (Kipf & Welling, ICLR 2017)](https://arxiv.org/abs/1609.02907)

# 簡介
GCN(Graph Convolutional Network)把「卷積」推廣到圖上,用一條極簡的**逐層傳播規則**做**半監督節點分類**:只有少數節點有標籤,靠圖結構把標籤資訊擴散到其餘節點。它的貢獻是把複雜的**譜圖卷積 (spectral graph convolution)** 理論,一路近似成一個又快又好用的線性傳播層。

核心定位:
- **半監督**:只標一小部分節點,其餘靠圖結構推論。
- **Transductive(直推式)**:訓練時要看到**整張圖**,操作在固定的圖上(對比 [[GraphSAGE]] 的 inductive)。
- **結構 + 特徵**:每個節點有特徵向量,GCN 沿邊聚合鄰居特徵。

# 方法

## 逐層傳播規則(核心)
$$
H^{(l+1)} = \sigma\Bigl(\hat{A}\,H^{(l)}\,W^{(l)}\Bigr)
$$
- $H^{(l)}$:第 $l$ 層的節點表徵矩陣($H^{(0)}=X$,輸入特徵)。
- $W^{(l)}$:該層可學權重。
- $\sigma$:非線性(如 ReLU)。
- $\hat{A}$:**正規化後的鄰接矩陣**(下面定義),負責「沿邊聚合鄰居」。

## 正規化鄰接矩陣與 renormalization trick
$$
\hat{A} = \tilde{D}^{-1/2}\,\tilde{A}\,\tilde{D}^{-1/2},\qquad \tilde{A} = A + I_N,\quad \tilde{D}_{ii} = \sum_j \tilde{A}_{ij}
$$
- $\tilde{A}=A+I$:加上**自環 (self-loop)**,讓節點聚合鄰居時也保留自己。
- $\tilde{D}^{-1/2}(\cdot)\tilde{D}^{-1/2}$:**對稱正規化**,依節點 degree 縮放,避免高 degree 節點主導、數值爆炸。

> **直覺**:$\hat{A}H$ 就是「把每個節點換成它(含自己的)鄰居特徵的 degree-加權平均」;再乘 $W$ 做線性轉換、過 $\sigma$ —— 一層 GCN = **一次鄰域平均 + 一層 MLP**。疊 $K$ 層匯集 $K$ 跳資訊。

## 從譜圖卷積推導(為什麼長這樣)
1. 圖上的卷積定義在**圖拉普拉斯** $L = I_N - D^{-1/2}AD^{-1/2}$ 的特徵基底上:$g_\theta \star x = U g_\theta U^\top x$($U$ 為 $L$ 的特徵向量)。直接做要做特徵分解,$O(N^2)$,太貴。
2. **ChebNet**:用 $K$ 階 Chebyshev 多項式逼近 $g_\theta$,免特徵分解、且**局部化**(只看 $K$ 跳)。
3. **GCN 的簡化**:取 $K=1$(一階)、近似 $\lambda_{max}\approx 2$,得到 $g_\theta \star x \approx \theta(I_N + D^{-1/2}AD^{-1/2})x$。
4. **renormalization trick**:$I_N + D^{-1/2}AD^{-1/2}$ 特徵值落在 $[0,2]$,反覆堆疊會數值不穩,改寫成 $\tilde{D}^{-1/2}\tilde{A}\tilde{D}^{-1/2}$(即上面的 $\hat{A}$)穩定訓練。

## 兩層 GCN 做分類
論文主模型就兩層:
$$
Z = \text{softmax}\Bigl(\hat{A}\;\text{ReLU}\bigl(\hat{A}\,X\,W^{(0)}\bigr)\,W^{(1)}\Bigr)
$$
損失:**只對有標籤節點**算 cross-entropy(半監督):
$$
\mathcal{L} = -\sum_{l\in \mathcal{Y}_L}\sum_{f} Y_{lf}\ln Z_{lf}
$$
($\mathcal{Y}_L$ 為有標籤節點集)。$\hat{A}$ 預先算好,前傳就是兩次稀疏矩陣乘法,很快。

# 與其他方法的關係
- **Spectral CNN → ChebNet → GCN**:GCN 是 ChebNet 取一階 + renormalization 的極簡特例。
- **[[GraphSAGE]]**:GraphSAGE 的 mean aggregator ≈ GCN 的 **inductive、mini-batch(鄰居採樣)** 版;GCN 本身是 **full-batch、transductive**。
- **[[node2vec]] / DeepWalk**:那些是無監督、只用拓樸的 embedding;GCN 端到端做半監督分類、且吃節點特徵。
- **Weisfeiler-Lehman**:GCN 的傳播規則可看成 1-WL 圖同構檢驗的可微分、參數化版本。

# 實驗
- **資料集**:Citeseer、Cora、Pubmed(引文網路)、NELL(知識圖)。半監督設定,每類僅少量標籤節點。
- **結果**:準確率與速度都勝過 label propagation、[[DeepWalk]]、ICA、Planetoid 等;兩層淺模型 + 預算很省。

# 限制
- **Transductive**:訓練要整張圖,新增節點得重算 → 催生 [[GraphSAGE]] 的 inductive 做法。
- **Full-batch 記憶體**:$\hat{A}$ 與全圖一起算,難擴展到超大圖 → 後續 FastGCN、Cluster-GCN、[[GraphSAGE]] 用採樣/分批解決。
- **層數淺(通常 2 層)**:疊太多層會 **over-smoothing**,各節點表徵趨同、反而變差。
- **鄰居權重固定**:degree 正規化是固定的、非學習得來,無法區分鄰居重要性 → [[GAT]] 改用 attention 加權。
- **需已知且無向的圖結構**:對有向圖、動態圖、缺邊的情況需額外處理。
