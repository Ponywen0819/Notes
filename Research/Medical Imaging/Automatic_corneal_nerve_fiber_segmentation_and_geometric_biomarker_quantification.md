#paper

[Automatic corneal nerve fiber segmentation and geometric biomarker quantification (Zhang et al., Eur. Phys. J. Plus 2020)](https://link.springer.com/article/10.1140/epjp/s13360-020-00127-y)

publish year: 2020

# 概述
針對**角膜共軛焦顯微影像（Corneal Confocal Microscopy, CCM）** 中的神經纖維建立完整自動化流程，包含：
1. 影像正規化以降低不同來源影像的差異。
2. 以 **U-Net + Attention Gating** 進行像素級切分。
3. 從切分結果萃取幾何／拓樸生物標記（曲率、長度、分岔點等），用於後續疾病診斷（如糖尿病神經病變早期偵測）。

# 方法論

利用**正規化方法**先減少不同影像中的差異，再利用**卷積模型**進行神經的切分，最後利用切分結果作為生物標記為後續的分析提供資訊。

## 影像正規化

因為觀察到影像會有以下特徵
- 光漂白 ( Photo-bleaching )
- 深度衰減 ( Attenuation in depth )
- 光照不均
- 影像採集差異 ( Image acquisition )

造成不均勻的影像成像與不同影像的差異，因此需要應用正規化方法減低可能影響卷積網路判斷的因素。

這裡的方法參照 [Foracchia 等人的研究](https://doi.org/10.1109/TMI.2005.852442)（Luminosity and contrast normalization in retinal images, MedIA 2005）。核心是把影像中每個像素分解為**亮度（luminosity）** 與**對比度（contrast）** 兩個成分，再以局部平均與標準差將它們拉到統一的目標分布，使得不同採集條件下的影像具有可比的灰階尺度。

## 使用 U-Net 進行切分
這裡使用 [Ronneberger 等人 (MICCAI 2015)](https://arxiv.org/abs/1505.04597) 提出的 **U-Net 架構**，但為了近一步讓網路聚焦在需要切分的區域中，引入了 [Oktay 等人 (MIDL 2018)](https://arxiv.org/abs/1804.03999) 提出的 **Attention gating modules ( AG )** 近一步提升聚焦的能力。

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
使用 **Cake wavelet** 進行**方向得分 (orientation score, OS)** 轉換。其概念是把一張 2D 影像 $f(x, y)$ 透過一組覆蓋整個頻域、各自只保留某個方向頻段的「蛋糕切片」型 wavelet $\psi_\theta$，提升到一個 3D 空間：
$$
U_f(x, y, \theta) = \bigl(f * \overline{\psi_\theta}\bigr)(x, y),\quad \theta \in [0, \pi)
$$
這個 3D 表示 $U_f$ 在每個位置 $(x, y)$ 上保留了**所有方向**的局部響應，使得在原圖中互相交叉或分岔的線狀結構，在 OS 空間中會被解開到不同的 $\theta$ 層。

**用 OS 偵測分岔點的流程**：
1. 將 U-Net 輸出的神經分割圖（而非原始 CCM 影像）作為輸入做 cake wavelet 轉換。使用切分後的乾淨影像可以避免原圖低對比區產生過弱的方向響應。
2. 對每個位置 $(x, y)$，沿 $\theta$ 軸找出局部極大值的個數。
3. 若某位置有 **3 個方向上的極大響應**（三叉），則判定為分岔點；角膜神經分岔幾乎都是 T 型／Y 型三叉，幾乎沒有十字交叉。
4. 將 OS 得到的分岔點與骨架法（3 × 3 全一卷積）得到的分岔點互補，可以修正骨架法在曲度大、分支緊密處容易誤判的位置。

相較於只看骨架的拓樸結構，OS 利用了**整段神經的方向延續性**，因此對小分支與低對比的分岔較不容易遺漏。

# 資料集
論文在兩組資料上驗證：
- **CNM (Corneal Nerve fiber images of Maastricht)**：用於切分任務的主要資料集，包含手動標註的神經纖維 ground truth。
- **CNT (Corneal Nerve Tortuosity)**：用於驗證曲率（tortuosity）量測的資料集，包含臨床上由專家評定的曲度等級。

# 結果
- **切分**：U-Net + AG + 正規化的組合在 CNM 上的 sensitivity / specificity 與 Dice 均優於當時 state-of-the-art 的傳統方法（如 Gabor filter、log-Gabor 等）與基本 U-Net。
- **曲率（VTI）**：在 CNT 上，自動量得的 VTI 與專家評分有高度相關，能有效區分高、中、低三個曲度等級。
- **分岔偵測**：OS-based 偵測比單純骨架法的 false positive 與 false negative 都更低，特別是在分支密集區。

# 結論與啟發
- 將「正規化 → FCN 切分 → 幾何後處理」拆成三個模組，每一塊都可以獨立替換或升級，是處理 CCM 影像的合理範式。
- **Attention Gating** 的引入對於前景（神經）佔比小、背景複雜的醫學影像特別有效，可作為自己改 U-Net 時的首要嘗試方向。
- **Cake wavelet / orientation score** 是一個值得記住的工具，特別適合處理「線狀、有交叉、需要保留方向資訊」的結構，例如：
    - 角膜／視網膜神經纖維
    - 視網膜血管
    - 表皮神經纖維（與 [[Intraepidermal_Nerve_Fiber_Analysisin_Human Patients_and_AnimalModels_of_Peripheral Neuropathy_A_Comparative_Review|IENF Comparative Review]] 中的影像分析任務有共通性）
- VTI 的設計同時考慮了**角度變化（$\text{SD}_\theta$）**、**鞍點數量（$N$）**、**曲線/弦長比（$M$）**，比單純用「曲線長度 / 弦長」更能反映複雜彎折，可借用於其他需要量化「彎曲程度」的場景。

# 待釐清 / 後續可看
- $\text{SD}_\theta$ 的計算是針對整段中心線還是 sliding window，論文需再細看。
- VTI 的常數 0.1 是經驗值，論文是否有針對不同結構重新校正？
- OS 偵測閾值（要多高才算是極大響應）的選擇方式。