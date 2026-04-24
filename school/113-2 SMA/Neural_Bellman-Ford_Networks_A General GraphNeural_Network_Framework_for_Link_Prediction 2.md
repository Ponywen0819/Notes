# Relate work
Link prediction 任務的方法可以分為三種
- path-based
- embedding
- graph neural networks

**Path based**

只能用於 homogeneous graph
方法為早期的主流，利用基本圖學概念去做指標的推倒，並預測可能的連線

**Embedding**

可以用於 homogeneous graph 也可以用於 knowledge graph，方法的主流概念是將 graph 的結構映射到較為低維度的表示中，但同時保留 graph 的結構。

優點是可以被scale up ，但沒辦被使用在 Inductive 的 link prediction 中。

 - **Transductive（传导式）**：模型训练和测试在同一张大图上。测试时所有节点都已在训练集中出现，仅预测**边的存在与否**。
    
- **Inductive（归纳式）**：模型训练与测试**在不同图**或包含**新节点**的同一张图上进行。测试阶段遇到的节点／子图不在训练数据中，模型需凭借学到的规则或函数，**推广到未知结构**。

**GNN**
可以分成兩種
- Auto‑Encoder
- Subgraph Encoding

**Auto-Encoder**
利用編碼器 Encoder 將節點映射到表示向量，並透過 Decoder 解析 edge 存在的機率。

若節點已有自帶特徵，例如相關文本與屬性，則可以用於歸納式的任務中，否則只能使用於傳導式任務中。

**Subgraph Encoding**
针对每个候选连结 (u,v)(u,v)，提取其局部子图（如 1–2 跳邻居），用专门的 GNN（或子图级别的编码器）将整个子图编码为一个向量，再判别 (u,v)(u,v) 是否应连边。



# 方法
