publish year: 2020

# 方法論

利用**正規化方法**先減少不同影像中的差異，再利用**卷積模型**進行神經的切分，最後利用切分結果作為生物標記為後續的分析提供資訊。

## 影像正規化

因為觀察到影像會有以下特徵
- 光漂白 ( Photo-bleaching )
- 深度衰減 ( Attenuation in depth )
- 光照不均
- 影像採集差異 ( Image acquisition )

造成不均勻的影像成像與不同影像的差異，因此需要應用正規化方法減低可能影響卷積網路判斷的因素。

這裡的方法參照 [Foracchia 等人的研究]()

## 使用 U-Net 進行切分
這裡使用 []() 提出的 **U-Net 架構**，但為了近一步讓網路聚焦在需要切分的區域中，引入了 []() 提出的 **Attention gating modules ( AG )** 近一步提升聚焦的能力。

這裡的模型使用 cross entropy of a pixel-wise soft-max loss 作為訓練損失函數。

![[Pasted image 20250427100207.png]]
模型架構

由於訓練記憶體不足，因此將原始圖片 ( 1535 $\times$ 1535 ) 切分成帶有重複區域的小區塊 ( 288 $\times$ 288 )。

每張圖片進行 8 次的資料增強，由隨機翻轉與旋轉進行，並使用 two-fold 進行訓練，訓練代數 ( epoch ) 為 50。

## 曲線狀神經生物標記萃取
流程如下
- 連通性修補
- 骨架化
- 小分支去除
- 分岔點偵測
- 切分骨架
- 計算分段的曲率計算

論文中使用 Matlab 中內建的 “bridge" 修補殘缺的神經連結，套用骨架化演算法獲取血管的骨架，接著進行 20 次只有單一鄰居像素的刪除，進行 20 次的原因是因為最粗的神經直徑就為 20 像素。

將骨架進行提取後他接著使用全一值 3 $\times$ 3 核心進行卷積，若有像素大於 3 則可以判斷他為分岔點。以分岔點作爲邊界對骨架進行切分，即可獲得神經的分段。

接著計算每個分段的 vessel tortuosity index ( VTI )
$$ VTI = \frac{0.1\text{SD}_\theta \times N \times M \times L_A}{L_C}$$

$\text{SD}_\theta$ 為沿中心線每一個像素處切線與 x 軸夾角，$N$ 代表一階導數的 critical point 數量，$L_A$ 代表曲線的長度，$L_C$ 代表中心線的長度

$$M = \frac{1}{I_p + 2} \sum^{I_p + 2}_{i = 1} \frac{L_{A_i}}{L_{C_i}}$$
$I_p$ 代表二階導數的鞍點數量
![[Pasted image 20250427105339.png]]

## 使用方向響應 ( orientation scores ) 檢測神經纖維分支
使用 Cake wavelet 進行