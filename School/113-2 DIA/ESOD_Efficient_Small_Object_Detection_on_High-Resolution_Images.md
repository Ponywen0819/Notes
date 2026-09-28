#paper

# Method
## Revisiting Object Detector
作者針對傳統物件偵測流程進行數學建模與模組化分析。給定一張高為 $H$、寬為 $W$ 的彩色影像，其可表示為：

$$$I \in \mathbb{R}^{H \times W \times 3}$$

### 1. Backbone Network

影像首先輸入骨幹網路進行特徵提取，產生金字塔結構的多層次特徵表示：

$$P = F(I) = \{ P_l \in \mathbb{R}^{\frac{H}{l} \times \frac{W}{l} \times C} \}$$

其中，$l$ 代表下採樣倍數，$C$ 為通道數，$P_l$ 為第 $l$ 層特徵圖。

為統一表示不同架構，無論是基於 CNN 或 ViT 的骨幹網路，皆可進一步拆解為 `Stem` 與 `Neck` 模組：

$$P = F(I) = F^n(F^s(I))$$

### 2. Detection Head

接續骨幹網路後，偵測頭 (Detection Head) 負責將特徵 $P$ 映射至 $n$ 個預測框：

$$B = H(P) = \{ (x^i_c, y^i_c, w^i, h^i, c^i) \}_{i=1}^n$$​

其中，$(x^i_c, y^i_c)$ 為預測框中心座標，$(w^i, h^i)$ 表示寬高，$c^i$ 為類別機率。

綜合以上流程，物件偵測可表述為以下函數組合：

$$B = H(F^n(F^s(I)))$$

### 3. 前處理機制（Conventional Filtering）

傳統架構中，為降低主幹網路負載，常額外加入輕量化前處理模組以篩除背景，篩選包含潛在物體的切片，再送入主網路。然而此類方法在小物體偵測場景下，雖計算開銷較低，實際效果卻有限。

![[Pasted image 20250602145054.png]]


## Efficient Object Seeker (ObjSeeker)
ObjSeeker 是部署於骨幹網路 Stem 後的輕量模組，其目標為預測一張 **實值遮罩圖 $\hat{M}$**，表示輸入影像中各位置「包含物體的可能性」，數值範圍為：

$$\hat{M} = O(F^s(I)) \in [0, 1]^{\frac{H}{8} \times \frac{W}{8}}$$
因此，$\hat{M}$ 並非二值，而是連續值遮罩，用於衡量每個位置屬於物體的置信程度（objectness score）。
### 結構設計
[偷米黃的文章](https://chih-sheng-huang821.medium.com/%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92-mobilenet-depthwise-separable-convolution-f1ed016b3467)
ObjSeeker 採用 MobileNet 提出的 **Depth-wise Separable Convolution** 結合 $1 \times 1$ **Point-wise Convolution**。其優勢如下：
- **計算效率高**：僅需 1.2 GFLOPs，可忽略不計。
- **感受野提升**：使用大型 kernel ($13 \times 13$) 提升單通道的有效感受野。

### Label 訓練策略
為訓練該模組，需產生與物體區域對應的 ground truth mask：
- 傳統：以物體中心進行高斯模糊。
- 本文：為更精準捕捉物體邊界，引入 Segment Anything Model（SAM）分割物體區域。
- 混合標註策略：$$M = \begin{cases} M_S \odot M_G & \text{if } \|M_S\|_1 > 0 \\ M_G & \text{otherwise} \end{cases}$$​

其中 $M_S$ 來自 SAM，$M_G$ 來自高斯生成，$\odot$ 表示逐元素乘法。

使用該架構的最主要目的就是為了減少計算量。

其次是因為想要每個通道可以擁有自己的**有效感受野（receptive field）**，使用較大的 kernel size ($13 \times 13$) 可以盡可能的擴展有效感受野。
## Adaptive Feature Slicer
透過已預測出的 mask $\hat{M}$，系統可進行針對性切片，排除背景，僅保留可能含有物體的區域。

### 原始演算法（不可並行）
1. 使用 Max Pooling 找出最大值作為物體中心位置。
2. 使用 Avg Pooling 估算對應區域大小。
3. 根據物體大小排序，優先處理大物體。
4. 切出固定大小區塊並置中對齊物體。
5. 找出切片中最靠近邊緣的物體並對齊，提升覆蓋率。
6. 移除已處理物體並重複流程。

### 簡化演算法（可並行）
考量 GPU 加速限制，提出簡化版切片演算法：

1. 使用滑動視窗對整張 mask 切片。
2. 對每片執行 local max 檢查是否含物體。
3. 移除無物體切片。
4. 對齊切片中最小座標以減少重複。  

此版本雖可提升併行效率，但在處理大物體時，可能發生切片錯誤劃分。

![[Pasted image 20250602163532.png]]

![[Pasted image 20250602164555.png]]

## Sparse Detection Head
在傳統架構中，偵測頭需對整張特徵圖進行預測。

而在本方法中，經過 ObjSeeker 與 Adaptive Feature Slicer 的處理後，僅需對切片區域進行局部預測，顯著降低計算量與延遲，提升效能。







# EXPERIMENT
## DataSets
該研究主要使用三個資料集
- **VisDrone**
	包含 10209 張靜態圖片，影像大小由 $960 \times 540$ 至 $2000 \times 1500$ ，總共包含 10 種物體。
	若將圖片切分成 $8 \times 8$ 的切片，大約 $70\%$ 的切片內沒有物件，可以證明物件稀疏分布的特性。 
- **UAVDT**
	包含 30 個影片 ( 38327幀 )，大小固定在 $1024 \times 540$ ，總共有 3 種物體。
	
- **TinyPerson**
	主要特點為物件 ( 人 ) 大小極小，通常不大於 20 像素。
	包含 1610 張靜態圖片，大小固定在 $1920\times1080$ 。

