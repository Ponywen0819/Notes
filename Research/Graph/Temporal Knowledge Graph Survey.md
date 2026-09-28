#paper

[A Survey on Temporal Knowledge Graph: Representation Learning and Applications (Cai et al., 2024)](https://arxiv.org/abs/2403.04782)

> arXiv:2403.04782 ｜ 2024-03 ｜ cs.CL / cs.AI
> 作者:Li Cai、Xin Mao、Yuhao Zhou、Zhaoguang Long、Changxu Wu、Man Lan
> 類型:**綜述 (Survey)** —— 整理 Temporal KG 表示學習方法與應用

---

# 一、簡介與定位 (Positioning)

多數知識圖譜研究只處理**靜態 KG**(事實不隨時間變),忽略了現實中大量知識**只在特定時段成立**(某人某段期間任某職、某事件發生在某天)。**Temporal Knowledge Graph (TKG)** 把時間維度納入,本綜述系統性整理「**TKG 表示學習**」—— 為實體與關係學低維向量、並建模其**隨時間演化**的動態。

本篇貢獻:① 統一定義、資料集、評估指標;② 提出以**核心技術**為軸的方法**分類體系 (taxonomy)**;③ 盤點下游應用與未來方向。

---

# 二、基本定義 (Definitions)

- **靜態 KG**:事實是三元組 (triple) `(head, relation, tail)`。
- **TKG**:擴充為**四元組 (quadruple)** `(head, relation, tail, timestamp)`,標記事實**何時**成立。
- **事實型態**:
  - **Event-based**:不規則時間戳上發生的事件(如外交衝突)。
  - **Time-interval based**:在某段區間內有效的事實(如 2008–2016 任總統)。
- **兩種任務設定(重要)**:
  - **Interpolation(內插)**:補全**已觀測時間範圍內**缺失的事實。
  - **Extrapolation(外推)**:**預測未來**、超出歷史範圍的事實(即 forecasting,難度更高)。

---

# 三、資料集 (Datasets)

| 資料集 | 規模 | 時間特性 | 主要用途 |
| --- | --- | --- | --- |
| **ICEWS**(14 / 05-15 / 18) | 7K–23K 實體 | 1–4 年,危機/外交事件 | 知識推理 |
| **GDELT** | 7.7K 實體 | 2,751 時間戳 | 事件預測 |
| **Wikidata** | 12.5K 實體 | 232 時間戳 | 推理 |
| **YAGO** | 10.6K 實體 | 189 時間戳 | 對齊任務 |

> ICEWS 是最常用的 benchmark 家族(危機事件),GDELT 時間粒度最細。

---

# 四、評估指標 (Metrics)

- **MRR(Mean Reciprocal Rank)**:正確答案排名倒數的平均。
- **Hits@k**:正確答案落在 top-k 的比例($k=1,3,5,10$)。
- **Time-aware filtering**:評估時排除「在該時間戳本就成立」的其他正確候選,避免低估排名(靜態 KG filtered 設定的時間版)。

---

# 五、方法分類體系 (Taxonomy)— 全篇核心

以**核心技術**分成十類:

| 類別 | 核心技術 | 代表方法 |
| --- | --- | --- |
| **A. Transformation 變換** | 嵌入空間的幾何變換 | 平移:TTransE、TA-TransE、**HyTE**(投影到時間超平面);旋轉:TeRo、ChronoR、RotateQVS(四元數) |
| **B. Decomposition 張量分解** | CP / Tucker 分解 | **TComplEx**、DE-SimplE、TuckERT、TLT-KGE |
| **C. GNN** | 帶時間感知的鄰域聚合 | TeA-GNN、TREA(時間關係注意力)、DEGAT(動態 + [[GAT]])、T²TKG |
| **D. Capsule Network** | routing 偵測階層樣式 | TempCaps(時間窗鄰居)、BiQCap、DuCape |
| **E. Autoregression 自迴歸** | 快照序列 + RNN/GRU | **RE-NET**(R-GCN+GRU)、**RE-GCN**(遞迴演化)、TiRGN、CEN |
| **F. Temporal Point Process** | 連續時間強度函數建模 | **Know-Evolve**(Rayleigh+RNN)、GHNN(Hawkes+cLSTM)、EvoKG |
| **G. Interpretability 可解釋** | 顯式推理路徑 | 子圖:**xERTE**(迭代剪枝);強化學習:CluSTeR、**TITer**(時間路徑 RL) |
| **H. Language Model** | 預訓練 LLM 做預測 | in-context:zrLLM(zero-shot 關係);微調:ECOLA、**GenTKG**、Chain-of-History(RAG) |
| **I. Few-Shot** | meta-learning 處理新實體/關係 | 少實體:MetaTKG、MetaTKGR;少關係:TR-Match、MTKGE |
| **J. 其他** | — | CyGNet(copy-generation,利用重複樣式)、**TANGO**(Neural ODE 連續演化)、DyERNIE(黎曼流形)、BoxTE |

> **兩條主線值得記**:
> - **Interpolation 派**:多用 A/B(變換、分解)把時間當成嵌入的一個維度,做圖補全。
> - **Extrapolation 派**:多用 E/F(自迴歸、point process)建模事件序列的時間動態,做未來預測 —— 這支與序列/動態圖學習關係最近。

---

# 六、下游應用 (Applications)

1. **TKG 推理**:鏈結預測(interpolation)與未來事件預測(extrapolation)。
2. **實體對齊 (Entity Alignment)**:跨多個 TKG 比對同一實體。
3. **問答 (Temporal QA)**:對 TKG 做時間感知檢索回答。

---

# 七、未來方向 (Future Directions)

- **可擴展性**:十億級 TKG 的高效訓練/推理。
- **可解釋性**:更透明的預測解釋路徑。
- **資訊融合**:整合多模態(文字、影像)與外部知識。
- **LLM 整合**:更好的預訓練與檢索增強策略(對應 H 類正快速成長)。

---

# 八、重點摘要 (Takeaways)

- **TKG = 四元組**`(h, r, t, time)`,核心是把「時間」納入表示學習,建模知識的**動態演化**。
- **任務分 interpolation(補過去)vs extrapolation(測未來)**,是讀任何 TKG 論文先要分清的設定。
- **方法十類**,但實務上常落在「變換/分解(補全)」與「自迴歸/point process(預測)」兩大陣營。
- **GNN 類**(TeA-GNN、RE-GCN)把 [[GCN]] / [[GraphSAGE]] 的鄰域聚合擴展到帶時間的快照圖;**LM 類**是近年最熱的新支。
- ICEWS / GDELT 是主流 benchmark,MRR / Hits@k + time-aware filtering 是標準評估。
