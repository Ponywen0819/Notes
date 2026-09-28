#paper

[Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks (Ren et al., NeurIPS 2015)](https://arxiv.org/pdf/1506.01497)

publish year: 2015

# 簡介
Faster R-CNN 是 R-CNN 系列的第三代，最關鍵的貢獻是用一個**全卷積的 Region Proposal Network (RPN)** 取代了傳統 [[Selective Search for Object Recognition|Selective Search]] 提案演算法，並讓 RPN 與後段偵測網路**共享卷積特徵**。這讓「proposal 生成」幾乎變成免費，將整個物件偵測流程拉到接近即時（5 fps with VGG-16 / 17 fps with ZF on Titan X）。

## R-CNN 系列演進
| 模型 | proposal 方法 | 特徵抽取 | 速度瓶頸 |
| --- | --- | --- | --- |
| R-CNN (2014) | Selective Search | 對每個 proposal 各跑一次 CNN | CNN 重複計算 |
| Fast R-CNN (2015) | Selective Search | 全圖跑一次 CNN + RoI Pooling | proposal 生成（~2 s/img） |
| **Faster R-CNN (2015)** | **RPN（學出來的）** | 全圖跑一次 CNN，與 RPN 共享 | – |

# 整體架構
$$
\text{Image} \rightarrow \text{Backbone CNN} \rightarrow \underbrace{\text{Feature Map}}_{\text{shared}} \begin{cases}\rightarrow \text{RPN} \rightarrow \text{Proposals}\\\rightarrow \text{RoI Pooling} \rightarrow \text{Classifier + BBox Regressor}\end{cases}
$$

整體可以視為兩個模組：
1. **RPN**：在共享特徵圖上，輸出一組可能含有物件的矩形 proposal 與其 objectness score。
2. **Fast R-CNN head**：對每個 proposal 做 RoI Pooling，得到固定大小的特徵，再接全連接層輸出**類別**與**精修後的 bounding box**。

兩個模組看起來像是「Attention 機制」——RPN 告訴後段網路「該看哪裡」。

# Region Proposal Network (RPN)
RPN 是一個**輕量全卷積網路**，輸入是 backbone 輸出的特徵圖，輸出是一組矩形 proposal。

## 核心設計
在特徵圖上以一個 $n \times n$ 的滑動窗（論文中 $n = 3$）掃過每個位置，將該窗的特徵投影到一個低維向量（如 256 或 512 維），再分支到兩個 sibling FC 層：
- **cls 層**：輸出每個 anchor 是「物件 / 非物件」的二元機率，共 $2k$ 個值。
- **reg 層**：輸出每個 anchor 的 4 個 bounding box 修正參數，共 $4k$ 個值。

其中 $k$ 是每個位置的 **anchor** 數量。

## Anchor
論文在每個位置放置 $k = 9$ 個 anchor：
- **3 種尺度（scale）**：$128^2, 256^2, 512^2$ 像素。
- **3 種長寬比（aspect ratio）**：$1{:}1, 1{:}2, 2{:}1$。

對一張 $W \times H$ 的特徵圖，總共會有 $W \cdot H \cdot k$ 個 anchor。Anchor 的關鍵性質：
- **Translation invariant**：每個位置的 anchor 設定都一樣，所以「物件在哪裡」可以被網路平移地處理。
- **Multi-scale**：用「多種 anchor 尺度」取代「圖像金字塔」或「特徵金字塔」，計算成本低。

## 正負樣本分配
- **正樣本**：與某個 ground-truth box 的 **IoU > 0.7**，或是與某 GT 有最高 IoU（避免該 GT 沒有正樣本對應）。
- **負樣本**：與所有 GT 的 IoU < 0.3。
- 介於之間的 anchor 不參與訓練。

訓練時每張圖隨機採樣 256 個 anchor，正:負比例為 1:1（不足則用負樣本補）。

## Bounding Box 參數化
RPN 不直接回歸絕對座標，而是相對於 anchor 的偏移量：
$$
\begin{aligned}
t_x &= (x - x_a) / w_a, & t_y &= (y - y_a) / h_a \\
t_w &= \log(w / w_a), & t_h &= \log(h / h_a)
\end{aligned}
$$
$(x, y, w, h)$ 是預測框、$(x_a, y_a, w_a, h_a)$ 是 anchor、加上 $^*$ 表示 ground-truth。這種**對數空間**的 $t_w, t_h$ 讓不同尺度的回歸誤差可以等效處理。

## 損失函數
RPN 的多任務損失：
$$
L(\{p_i\}, \{t_i\}) = \frac{1}{N_{\text{cls}}}\sum_i L_{\text{cls}}(p_i, p_i^*) + \lambda\frac{1}{N_{\text{reg}}}\sum_i p_i^* L_{\text{reg}}(t_i, t_i^*)
$$
其中：
- $L_{\text{cls}}$：二元 log loss（物件 / 非物件）。
- $L_{\text{reg}}$：smooth $L_1$ loss，只在正樣本（$p_i^* = 1$）時生效。
- $\lambda$：兩項權重，論文用 $\lambda = 10$ 平衡兩者數量級。
- $N_{\text{cls}} \approx 256$（mini-batch size），$N_{\text{reg}} \approx 2400$（anchor 數）。

# Fast R-CNN Head
RPN 篩出 top-$N$（訓練時 2000、測試時 300）proposal 後，丟給後段：
1. **RoI Pooling**：將任意大小的 proposal 對應到特徵圖區域，再切成固定的 $H \times W$ 網格（如 $7 \times 7$），每格取 max。
2. 接 FC 層 → 兩個輸出頭：
    - **softmax classifier**：$C + 1$ 類（含背景）。
    - **bbox regressor**：對每個類別輸出一組 $(t_x, t_y, t_w, t_h)$，做第二階段的 box 精修。

# 訓練策略
RPN 與 Fast R-CNN 共享卷積層，但兩者目標不同（一個學 proposal、一個學最終偵測），如果直接 joint training 容易彼此干擾。論文提出三種策略：

## 1. 4-Step Alternating Training（論文採用）
1. 用 ImageNet 預訓練的 backbone 初始化，**獨立訓練 RPN**。
2. 用 RPN 產生的 proposal **獨立訓練 Fast R-CNN**（此時兩網路尚未共享卷積）。
3. 用 Fast R-CNN 的 backbone **初始化 RPN**，固定共享層，只 fine-tune RPN 自己的層。
4. 固定共享層，只 fine-tune Fast R-CNN 自己的層。
此時兩個網路真正共享卷積特徵。

## 2. Approximate Joint Training
把 RPN 與 Fast R-CNN 的 loss 加總一次反向傳播，但忽略 proposal 座標對 RPN 參數的梯度。實作較簡單，速度快 25–50%，準確度幾乎一樣。

## 3. Non-approximate Joint Training
完整考慮 proposal 座標的梯度，需要 RoI warping 層可微分（後來的工作如 Mask R-CNN 中的 RoI Align 接近這個概念）。

# 推論流程
1. Backbone 跑全圖一次 → 共享特徵圖。
2. RPN 輸出所有 anchor 的 cls/reg → 得到 ~20k proposals。
3. 依 cls score 取前 N（訓練 12000 / 測試 6000），做 **NMS（IoU 閾值 0.7）**，最後留下 top-N（訓練 2000 / 測試 300）。
4. 對留下的 proposal 做 RoI Pooling → classifier + regressor。
5. 對最終偵測再做一次 per-class NMS，輸出結果。

# 實驗結果（PASCAL VOC 2007）
| 方法 | Proposal | mAP | Test 速度 |
| --- | --- | --- | --- |
| Fast R-CNN (SS) | Selective Search | 66.9% | ~2 s |
| Faster R-CNN (RPN, ZF) | RPN | 59.9–69.9% | 17 fps |
| Faster R-CNN (RPN, VGG-16) | RPN | **73.2%** | 5 fps |

關鍵觀察：RPN 的 proposal 不僅更快，**品質還比 Selective Search 好**——因為 RPN 是用偵測任務的訊號學出來的，而 SS 是純啟發式。

# 與其他方法的關係
- **R-CNN / Fast R-CNN**：Faster R-CNN 是這條 two-stage 路線的最終形態。
- **YOLO / SSD**：與 Faster R-CNN 同期出現的 **single-stage** 偵測器，把 RPN 與分類合而為一，更快但傳統上略遜於 two-stage 的精度。可參考 [[YOLOv7]]。
- **Mask R-CNN (2017)**：在 Faster R-CNN 上加一個 mask branch，並把 RoI Pooling 換成 RoI Align。
- **FPN (2017)**：在 backbone 上加多尺度特徵融合，搭配 Faster R-CNN 進一步提升小物件偵測。

# 實作細節 / 注意事項
- Anchor 必須涵蓋資料集中物件的尺寸分布；醫學影像或特殊資料常需要重新設計尺度與長寬比。
- RoI Pooling 的量化會造成定位誤差（特別是小物件），實務上多換成 **RoI Align**。
- RPN 訓練時的正負樣本不平衡是常見坑點，注意 mini-batch sampling 的比例。
- NMS 閾值（0.7 for proposals, 0.3–0.5 for final detections）對結果敏感。

# 啟發
- **「學出來的 proposal」勝過手刻 heuristic**：這個觀念後來在許多 vision task 中反覆出現（如 DETR 把 anchor + NMS 也學掉）。
- **共享特徵**是設計多任務／多階段架構時的省錢關鍵。
- **Anchor + 偏移回歸**的設計影響了之後幾乎所有 anchor-based detector，直到 anchor-free 方法（FCOS、CenterNet）出現才被部分取代。
