#note

# Lecture 09 — Estimation of Transformations（變換的估計）

> 課程：NTU EE 114 Computer Vision｜講者：簡韶逸 教授
> 投影片來源致謝：Marc Pollefeys
> 參考教科書：Hartley & Zisserman, *Multiple View Geometry in Computer Vision*, 第 4 章

---

## 0. 這一講在做什麼？

lec08 告訴我們 **有哪些變換**（Euclidean、Similarity、Affine、Projective）。這一講的核心問題是：

> 給一堆 **有雜訊、而且可能混有錯誤對應（outliers）的點對**，怎麼 **估計** 背後那個變換矩陣？

我們會以 **2D homography $H$** 作為主要範例，但整個方法論（cost function、Gold Standard、Normalization、RANSAC）對其它模型（Fundamental matrix $F$、camera matrix $P$、Trifocal tensor $T$）都通用。

大綱：
1. **待估參數與量測需求**
2. **DLT（Direct Linear Transformation）**：代數的線性解
3. **各種 Cost Function**：代數距離、幾何距離、重投影誤差、Sampson 誤差
4. **Normalization**：為什麼不做就完蛋
5. **Gold Standard Algorithm**：maximum likelihood 解
6. **Robust Estimation**：RANSAC 處理 outliers
7. **Automatic Homography Estimation**：整條 pipeline

---

## 1. 參數估計（Parameter Estimation）

我們在 CV 中常需估計幾類變換：

| 問題 | 對應 | 待估參數 | 備註 |
|---|---|---|---|
| **2D homography** | $\mathbf{x}_i' = H\mathbf{x}_i$ | $H\in\mathbb{R}^{3\times 3}$（8 DOF） | 平面對平面 / 相機原地旋轉 |
| **3D→2D 投影** | $\mathbf{x}_i = P\mathbf{X}_i$ | $P\in\mathbb{R}^{3\times 4}$（11 DOF） | 相機校準 |
| **Fundamental matrix** | $\mathbf{x}_i'^T F\mathbf{x}_i = 0$ | $F\in\mathbb{R}^{3\times 3}$（7 DOF） | 兩視圖的 epipolar 幾何 |
| **Trifocal tensor** | 三視圖的三線性關係 | $T$（18 DOF） | 三視圖 |

這一講以 **2D homography** 為標竿。

### 1.1 需要多少量測？

基本原則：**獨立方程式數 ≥ 未知自由度**。

以 2D homography 為例：
- 每個對應點 $\mathbf{x}_i\leftrightarrow\mathbf{x}_i'$ 給 2 條獨立方程（$x'$、$y'$ 兩分量）。
- $H$ 有 8 DOF → 需要 $2n \geq 8$，亦即 $n\geq 4$ 點。

**4 點** = 最小解，**exact solution**。

### 1.2 近似解 vs. 最小解

- **Minimal solution**（4 點）：exact 代數解，但對雜訊敏感，常用於 RANSAC 內部的假設生成。
- **More points**：沒有真正的 exact 解（因量測誤差），改尋「最佳」解 — 需要定義 **cost function**。

### 1.3 Gold Standard Algorithm

- **Cost function** 是在某個統計假設（通常 Gaussian noise）下 **最佳** 的目標函數。
- **最小化這個 cost function 的演算法** 稱為 **Gold Standard algorithm**（金標準）。
- 其它演算法的品質，就以 Gold Standard 為基準比較。

---

## 2. Direct Linear Transformation (DLT)

### 2.1 推導

從 $\mathbf{x}_i' = H\mathbf{x}_i$ 出發。注意齊次座標只有方向意義，所以改寫：
$$
\mathbf{x}_i' \times H\mathbf{x}_i = 0
$$
（平行向量的外積為零。）

把 $H$ 的三列寫作 $\mathbf{h}^{1T},\mathbf{h}^{2T},\mathbf{h}^{3T}$：
$$
H\mathbf{x}_i = \begin{pmatrix}\mathbf{h}^{1T}\mathbf{x}_i\\ \mathbf{h}^{2T}\mathbf{x}_i\\ \mathbf{h}^{3T}\mathbf{x}_i\end{pmatrix}
$$
令 $\mathbf{x}_i'=(x_i', y_i', w_i')^T$，展開外積：
$$
\mathbf{x}_i'\times H\mathbf{x}_i
=\begin{pmatrix}
y_i'\mathbf{h}^{3T}\mathbf{x}_i - w_i'\mathbf{h}^{2T}\mathbf{x}_i\\
w_i'\mathbf{h}^{1T}\mathbf{x}_i - x_i'\mathbf{h}^{3T}\mathbf{x}_i\\
x_i'\mathbf{h}^{2T}\mathbf{x}_i - y_i'\mathbf{h}^{1T}\mathbf{x}_i
\end{pmatrix}=0
$$

因為 $\mathbf{h}^{jT}\mathbf{x}_i = \mathbf{x}_i^T\mathbf{h}^j$，上式可以整理成 **$A_i\mathbf{h}=0$** 形式：

$$
A_i = \begin{pmatrix}
\mathbf{0}^T & -w_i'\mathbf{x}_i^T & y_i'\mathbf{x}_i^T\\
w_i'\mathbf{x}_i^T & \mathbf{0}^T & -x_i'\mathbf{x}_i^T\\
-y_i'\mathbf{x}_i^T & x_i'\mathbf{x}_i^T & \mathbf{0}^T
\end{pmatrix},\quad
\mathbf{h} = \begin{pmatrix}\mathbf{h}^1\\\mathbf{h}^2\\\mathbf{h}^3\end{pmatrix}\in\mathbb{R}^9
$$

### 2.2 為什麼只有 2 列獨立？

$A_i$ 是 $3\times 9$，但第三列恰為 **$x_i'\cdot(\text{第 1 列}) + y_i'\cdot(\text{第 2 列})$**（乘上合適的 $w_i'$ 後），所以三列線性相依。每對應點只貢獻 **2 條** 獨立方程 — 跟自由度計數一致。

常用精簡形式：
$$
A_i = \begin{pmatrix}
\mathbf{0}^T & -w_i'\mathbf{x}_i^T & y_i'\mathbf{x}_i^T\\
w_i'\mathbf{x}_i^T & \mathbf{0}^T & -x_i'\mathbf{x}_i^T
\end{pmatrix}
$$

### 2.3 堆疊與求解

把 $n$ 個點合併：
$$
A\mathbf{h}=0,\quad A\in\mathbb{R}^{2n\times 9}
$$

- **$n=4$**（8 條方程）：$A$ 是 $8\times 9$，秩 8 → null space 一維，$\mathbf{h}$ 就在其中。
- **$n>4$**：通常 $A$ 滿秩 9，$A\mathbf{h}=0$ 沒有非零解（或只有 $\mathbf{h}=0$，沒意義）。

### 2.4 過定系統（over-determined）怎麼辦？

有雜訊時 $A\mathbf{h}=0$ 無法完全滿足，改為：
$$
\min_{\mathbf{h}} \|A\mathbf{h}\|\quad\text{s.t.}\ \|\mathbf{h}\|=1
$$
（加約束避免平凡解 $\mathbf{h}=0$，且 $H$ 本就 scale 無關。）

標準解法：**SVD**。令 $A=U\Sigma V^T$，$\mathbf{h}$ 取 $V$ **最後一行**（對應最小奇異值的方向）。

### 2.5 DLT 演算法流程

> **DLT Algorithm**
> 1. 對每對應點 $\mathbf{x}_i\leftrightarrow\mathbf{x}_i'$ 建 $A_i$（通常只取前 2 列）。
> 2. 把 $n$ 個 $A_i$ 疊成 $A\in\mathbb{R}^{2n\times 9}$。
> 3. 對 $A$ 做 SVD；$\mathbf{h}$ = $V$ 的最後一行。
> 4. 把 $\mathbf{h}$ reshape 成 $3\times 3$ 矩陣 $H$。

### 2.6 Inhomogeneous Solution（較不推薦）

另一做法：令 $h_9=1$，把最後一個自由度固定，其餘 8 個未知用線性最小平方解。問題：若真實 $h_9=0$（例：原點被映射到無窮遠），這個設定直接失效；$h_9$ 接近 0 時也很不穩。**所以一般不推**。

> 註：$h_9 = H_{33} = 0$ 意味 $(0,0,1)^T$ 被映到 $H_{33}=0$ 方向，即原點映到無窮遠。

### 2.7 Degenerate Configurations（退化構型）

對應點如果分佈太特殊，DLT 的 null space 會超過 1 維 → 解不唯一。投影片上的兩個退化例：

- **Case A**：若存在矩陣 $H^*=\mathbf{x}'_4\,\mathbf{l}^T$（rank 1，$\mathbf{l}$ 是過前三點的一條線）滿足 $H^*\mathbf{x}_i=\mathbf{0},\ i=1,2,3$ 且 $H^*\mathbf{x}_4 = k\mathbf{x}'_4$ → 任何 $\alpha H^* + \beta H$ 都是解，null space 變 2 維。
- **Case B**：如果 $H^*$ 是 **唯一解**，但它 rank 1 不是 homography → 根本沒有 homography 對應。

**直覺結論**：4 個點不能有 3 點共線，否則退化。

### 2.8 從線、混合型求解

- **從 2D 線對應**：$\mathbf{l}_i' = H^{-T}\mathbf{l}_i$，仍可列成 $A\mathbf{h}=0$ 形式。**至少 4 條線**。
- **3D homography**（15 DOF）：**至少 5 點或 5 平面**。
- **2D affine**（6 DOF）：**至少 3 點或 3 線**。
- **Conic**：每個 conic 提供 5 個約束。
- **混合**：2 點 + 2 線 → 不夠唯一；3 點 + 1 線 或 1 點 + 3 線 可以。

---

## 3. Cost Functions（損失函數）

DLT 最小化的其實是 **代數誤差**，但代數誤差在幾何／統計意義上並非最優。這一節整理不同的 cost function。

### 3.1 Algebraic Distance（代數距離）

DLT 的目標：
$$
\min_{\|\mathbf{h}\|=1}\|A\mathbf{h}\|^2 = \sum_i\|\mathbf{e}_i\|^2,\quad \mathbf{e}_i = A_i\mathbf{h}
$$

定義兩個齊次向量的代數距離：
$$
d_{\text{alg}}(\mathbf{x}_1,\mathbf{x}_2)^2 = a_1^2 + a_2^2,\quad \mathbf{a}=\mathbf{x}_1\times\mathbf{x}_2
$$
（只取前兩分量的平方和。）對應到 homography：
$$
d_{\text{alg}}(\mathbf{x}_i', H\mathbf{x}_i)^2 = \|\mathbf{e}_i\|^2
$$

特性：
- 不具幾何／統計意義，但 **線性、快**。
- 在適當 normalization 下很實用，常作為非線性優化的 **初始化**。

### 3.2 Geometric Distance（幾何距離）

在影像平面用歐氏距離量測：
- $\mathbf{x}$：量測座標
- $\hat{\mathbf{x}}$：估計座標
- $\bar{\mathbf{x}}$：真值（unknown）
- $d(\cdot,\cdot)$：影像上的歐氏距離（$\mathbb{R}^2$ 上的）

**Error in one image**（第一張影像視為精確）：
$$
\hat H = \arg\min_H\sum_i d(\mathbf{x}_i', H\mathbf{x}_i)^2
$$
適合 **校準板** 這種第一張影像座標準確的情境。

**Symmetric transfer error**（對稱轉移誤差）：
$$
\hat H = \arg\min_H\sum_i\Big[d(\mathbf{x}_i, H^{-1}\mathbf{x}_i')^2 + d(\mathbf{x}_i', H\mathbf{x}_i)^2\Big]
$$
兩張都是量測，都有雜訊 → 正逆方向都該算。

**Reprojection error**（重投影誤差，真正的 Gold Standard）：
$$
(\hat H, \{\hat{\mathbf{x}}_i, \hat{\mathbf{x}}_i'\}) = \arg\min_{H,\hat{\mathbf{x}}_i,\hat{\mathbf{x}}_i'}\sum_i \Big[d(\mathbf{x}_i,\hat{\mathbf{x}}_i)^2 + d(\mathbf{x}_i',\hat{\mathbf{x}}_i')^2\Big]
$$
subject to $\hat{\mathbf{x}}_i' = \hat H\hat{\mathbf{x}}_i$。

**直覺**：在 $\hat H$ 的約束下，為每對 $(\mathbf{x}_i,\mathbf{x}_i')$ 找「最接近兩個量測的、能被 $\hat H$ 關聯的一組估計 $(\hat{\mathbf{x}}_i,\hat{\mathbf{x}}_i')$」，讓兩端重投影誤差平方和最小。

### 3.3 Symmetric Transfer vs. Reprojection

- **Symmetric transfer**：把 $\mathbf{x}$ 用 $H^{-1}$ 投到第一張、把 $\mathbf{x}'$ 用 $H$ 投到第二張，分別量兩端的距離。
- **Reprojection**：引入「真點」$\hat{\mathbf{x}}_i,\hat{\mathbf{x}}_i'$ 一起當參數優化，兩端量的是 $\mathbf{x}$ 到 $\hat{\mathbf{x}}$ 的距離。

Reprojection error 更 **嚴謹**（等同 MLE），但未知數多很多。

### 3.4 Algebraic vs. Geometric 的關係

在「第一張影像無誤差」假設下可以顯式比較：
$$
d(\mathbf{x}_i', \hat{\mathbf{x}}_i)^2 = \frac{d_{\text{alg}}(\mathbf{x}_i', \hat{\mathbf{x}}_i)^2}{w_i \hat w_i}
$$
- 通常取 $w_i=1$，但 **$\hat w_i = \mathbf{h}^{3T}\mathbf{x}_i$ 不為常數**（除非 affinity 之類，$\mathbf{h}^3$ 最後一行是 $(0,0,1)$）。
- **對 affine 變換來說 DLT 其實就是在最小化幾何誤差**，只是換了尺度。
- 對一般 projective，DLT 的代數誤差跟幾何誤差差一個「點相依的尺度因子」，所以 DLT 的解並不是 MLE。

### 3.5 Sampson Error（代數與幾何之間的折衷）

真實的幾何誤差需要優化「真點」$\hat{\mathbf{X}}$，但計算貴。Sampson 想法：**把 $\hat{\mathbf{X}}$ 用一階近似求出閉式解**。

把量測值 $\mathbf{X}=(x,y,x',y')^T$ 當作 4D 向量，代數約束 $\mathbf{A}\mathbf{h}=C_H(\mathbf{X})=0$ 定義一個「變種集合（variety）$\nu_H$」。

**目標**：找 $\hat{\mathbf{X}}$ 使 $\|\mathbf{X}-\hat{\mathbf{X}}\|^2$ 最小且 $C_H(\hat{\mathbf{X}})=0$。

一階 Taylor 近似：
$$
C_H(\mathbf{X}+\delta\mathbf{X})\approx C_H(\mathbf{X}) + \frac{\partial C_H}{\partial \mathbf{X}}\delta\mathbf{X} = 0
$$
令 $J = \partial C_H/\partial\mathbf{X}\in\mathbb{R}^{2\times 4}$、$\mathbf{e}=C_H(\mathbf{X})$，則 $J\delta\mathbf{X}=-\mathbf{e}$。

求最小的 $\|\delta\mathbf{X}\|^2$ 且滿足 $J\delta\mathbf{X}=-\mathbf{e}$，用 **Lagrange 乘數**：
$$
\min_{\delta\mathbf{X}}\|\delta\mathbf{X}\|^2 - 2\boldsymbol{\lambda}^T(J\delta\mathbf{X}+\mathbf{e})
$$
對 $\delta\mathbf{X},\boldsymbol{\lambda}$ 偏微分：
$$
\delta\mathbf{X}=J^T\boldsymbol{\lambda},\quad JJ^T\boldsymbol{\lambda}+\mathbf{e}=0
$$
解出：
$$
\boxed{\delta\mathbf{X} = -J^T(JJ^T)^{-1}\mathbf{e},\quad \|\delta\mathbf{X}\|^2 = \mathbf{e}^T(JJ^T)^{-1}\mathbf{e}}
$$

這個 $\|\delta\mathbf{X}\|^2$ 就是 **Sampson error**。每個點都要算然後加總：
$$
\text{Sampson cost}=\sum_i \mathbf{e}_i^T(J_i J_i^T)^{-1}\mathbf{e}_i
$$

**實用特色**：
- 類似 Mahalanobis distance：$\mathbf{e}^T(JJ^T)^{-1}\mathbf{e}$。
- 跟線性 reparametrization 無關（$\mathbf{e}$ 跟 $J$ 同比例縮放會互消）。
- 遠比純 geometric 少 **未知變數**（不用把 $\hat{\mathbf{X}}_i$ 寫入參數空間）。
- 當 $J$ 接近真雅可比時，Sampson error 非常接近幾何誤差。實務上是最常用的「幾何誤差代表品」。

### 3.6 Statistical Cost Function 與 MLE

假設 **zero-mean, isotropic Gaussian noise**（外點已被移除）：
$$
\Pr(\mathbf{x}) = \frac{1}{2\pi\sigma^2}\exp\!\left(-\frac{d(\mathbf{x},\bar{\mathbf{x}})^2}{2\sigma^2}\right)
$$

**第一張精確、第二張有雜訊**：
$$
\Pr(\{\mathbf{x}_i'\}\mid H) = \prod_i \frac{1}{2\pi\sigma^2}\exp\!\left(-\frac{d(\mathbf{x}_i', H\mathbf{x}_i)^2}{2\sigma^2}\right)
$$
取 log 後 MLE 等價於
$$
\min_H \sum_i d(\mathbf{x}_i', H\mathbf{x}_i)^2
$$
**與 geometric error in one image 相同**。

**兩張都有雜訊**：
$$
\Pr(\cdot\mid H) = \prod_i \frac{1}{2\pi\sigma^2}\exp\!\left(-\frac{d(\mathbf{x}_i,\hat{\mathbf{x}}_i)^2+d(\mathbf{x}_i',\hat{\mathbf{x}}_i')^2}{2\sigma^2}\right)
$$
MLE 等價於 **reprojection error**。

### 3.7 Mahalanobis Distance（非各向同性雜訊）

如果雜訊是有 covariance $\Sigma$ 的一般 Gaussian：
$$
\|\mathbf{X}-\bar{\mathbf{X}}\|_\Sigma^2 = (\mathbf{X}-\bar{\mathbf{X}})^T\Sigma^{-1}(\mathbf{X}-\bar{\mathbf{X}})
$$
多張影像各自獨立：
$$
\sum_i \left(\|\mathbf{X}_i-\bar{\mathbf{X}}_i\|_{\Sigma_i}^2 + \|\mathbf{X}_i'-\bar{\mathbf{X}}_i'\|_{\Sigma_i'}^2\right)
$$

---

## 4. Invariance 與 Normalization（重要！）

### 4.1 座標變換下 DLT 會變答案嗎？

假設我們把兩張影像的座標各做一個變換：
$$
\tilde{\mathbf{x}} = T\mathbf{x},\quad \tilde{\mathbf{x}}' = T'\mathbf{x}'
$$
則新的 homography 應滿足：
$$
\tilde{\mathbf{x}}' = \tilde H\tilde{\mathbf{x}},\quad \tilde H = T' H T^{-1}
$$

**問題**：DLT 在新座標上跑，得到 $\tilde H'$。這個 $\tilde H'$ 是否等於 $T' H T^{-1}$？

### 4.2 DLT 不是不變的！

從代數誤差出發推：
$$
\tilde{\mathbf{e}}_i = \tilde{\mathbf{x}}_i'\times \tilde H\tilde{\mathbf{x}}_i = T'\mathbf{x}_i' \times T'HT^{-1}T\mathbf{x}_i = T'^{*}(\mathbf{x}_i'\times H\mathbf{x}_i) = T'^{*}\mathbf{e}_i
$$
其中 $T'^*$ 是 $T'$ 的 **cofactor matrix**（對 similarity $sR$，$T'^* = s\begin{pmatrix}R^T & 0\\ -s^{-1}t^TR & s^{-1}\end{pmatrix}$）。

對 similarity 特別簡潔：
$$
d_{\text{alg}}(\tilde{\mathbf{x}}_i',\tilde H\tilde{\mathbf{x}}_i)^2 = s^2\,d_{\text{alg}}(\mathbf{x}_i', H\mathbf{x}_i)^2 \cdot (\text{點相依項})
$$
**點相依項** 讓優化結果跟座標系有關，DLT 的解會隨座標系變化。→ **DLT 不是不變的**。

### 4.3 Geometric Error 是不變的（對 similarity）

同樣的代入：
$$
d(\tilde{\mathbf{x}}_i', \tilde H\tilde{\mathbf{x}}_i) = d(T'\mathbf{x}_i', T'HT^{-1}\cdot T\mathbf{x}_i) = d(T'\mathbf{x}_i', T'H\mathbf{x}_i) = s\,d(\mathbf{x}_i', H\mathbf{x}_i)
$$
（對 similarity：距離統一縮 $s$ 倍，不影響 argmin）。
→ **Geometric error 在 similarity 下不變**，選什麼座標系都一樣答案。

### 4.4 Normalization 的重要性

DLT 建出來的 $A$ 各欄的數值「量級」差很多：

| 欄 | 量級 |
|---|---|
| $0$ | 0 |
| $\mathbf{x}_i^T$（像素座標） | $\sim 10^2$ |
| $x_i y_i$（乘積） | $\sim 10^4$ |
| $x_i$ 或 $y_i$ | $\sim 10^2$ |
| $1$ | $1$ |

**差 4 個數量級**！SVD 的數值穩定性會崩潰（小奇異值被 round-off 淹沒）。

**解法**：對每張影像都先做 **Isotropic scaling** pre-conditioning：
1. 把點雲質心平移到原點。
2. 各向同性地縮放，使所有點到原點的 **平均距離是 $\sqrt{2}$**。
3. 兩張影像 **各自** 做，彼此獨立。

對應到矩陣：
$$
T_{\text{norm}} = \begin{pmatrix}\frac{\sqrt{2}}{\overline{w+h}} & 0 & -w/2\\ 0 & \frac{\sqrt{2}}{\overline{w+h}} & -h/2\\ 0 & 0 & 1\end{pmatrix}
$$
（投影片上寫 $w+h$，典型實作用 $\sqrt{2}$ 的平均距離。）

### 4.5 Normalized DLT Algorithm（實務上的標準做法）

> **Normalized DLT**
> 1. 算 $T_{\text{norm}}$，變換 $\tilde{\mathbf{x}}_i = T_{\text{norm}}\mathbf{x}_i$。對第二張也算 $T_{\text{norm}}'$，得 $\tilde{\mathbf{x}}_i' = T_{\text{norm}}'\mathbf{x}_i'$。
> 2. 對 $\tilde{\mathbf{x}}_i \leftrightarrow \tilde{\mathbf{x}}_i'$ 跑原 DLT 得 $\tilde H$。
> 3. **去 normalize**：$H = T_{\text{norm}}'^{-1}\,\tilde H\,T_{\text{norm}}$。

優點：
- **精度大幅提升**（數值穩定）。
- **不受座標原點與尺度選擇影響**（對任意 similarity 不變）。

**這是實務上必做的步驟**，**不做的話 DLT 幾乎不可用**。

---

## 5. Iterative Minimization Methods（非線性優化）

要最小化幾何誤差、Sampson error，或 reprojection error，一般沒有閉式解，得用迭代法。

### 5.1 共通設計

- 比 DLT 慢。
- 要 **初始化**（通常用 Normalized DLT 或 RANSAC 的輸出）。
- **收斂沒保證**、可能卡在 local minima。
- 需要 **停止準則**（相對變化、梯度大小等）。

### 5.2 設計面向

- **Cost function**：要符合統計假設（通常 geometric / Sampson）。
- **參數化（parameterization）**：
  - 最小參數化（8 維）vs. 過參數化（9 維、加 $\|\mathbf{h}\|=1$ 約束）：
    - 最小化複雜、cost surface 不好看；
    - 過參數化配合好的演算法（如 LM）更穩定。
  - 局部參數化（local parameterization）：在當前估計附近切換到最小參數。
  - 也可以 **限制在特定子類**（affine、similarity）以減少參數。
- **初始化**：Normalized DLT，或 RANSAC（有 outliers 時）。

### 5.3 函式規格（function specification）

- 量測向量 $\mathbf{X}\in\mathbb{R}^N$，協方差 $\Sigma$。
- 參數向量 $\mathbf{P}\in\mathbb{R}^M$。
- 映射 $f:\mathbb{R}^M\to\mathbb{R}^N$，值域（surface）$S$ 就是「可實現的量測」。
- Cost：$\|\mathbf{X}-f(\mathbf{P})\|_\Sigma^2 = (\mathbf{X}-f(\mathbf{P}))^T\Sigma^{-1}(\mathbf{X}-f(\mathbf{P}))$。

對應到各 cost function：

| Cost | $f$ 的輸出 | $\mathbf{X}$ 的組成 |
|---|---|---|
| One-image | $(H\mathbf{x}_1, \dots, H\mathbf{x}_n)$ | 2n 維 $\{\mathbf{x}_i'\}$ |
| Symmetric transfer | $(H^{-1}\mathbf{x}_1',\dots,H\mathbf{x}_n)$ | 4n 維 $\{\mathbf{x}_i,\mathbf{x}_i'\}$ |
| Reprojection | $(\hat{\mathbf{x}}_i, \hat{\mathbf{x}}_i')$ | 4n 維 $\{\mathbf{x}_i,\mathbf{x}_i'\}$；需同時優化 $\hat{\mathbf{x}}_i$ |

### 5.4 演算法

- **Newton's method**
- **Levenberg-Marquardt (LM)** — 標準選擇
- **Powell's method**（不需梯度）
- **Simplex / Nelder-Mead**（無梯度）

### 5.5 Levenberg-Marquardt 快速回顧

給 $f:\mathbb{R}^M\to\mathbb{R}^N$，量測 $\mathbf{X}$。線性化：
$$
f(\mathbf{p}+\boldsymbol{\Delta}) \approx f(\mathbf{p}) + J\boldsymbol{\Delta},\quad J=\frac{\partial f}{\partial\mathbf{p}}
$$

目標：
$$
\min_{\boldsymbol{\Delta}}\|(\mathbf{X}-f(\mathbf{p}))-J\boldsymbol{\Delta}\|^2
$$

正規方程：
$$
(J^TJ)\boldsymbol{\Delta}=J^T\mathbf{r},\quad \mathbf{r}=\mathbf{X}-f(\mathbf{p})
$$
$J^TJ$ 就是 **Gauss-Newton Hessian 近似**。

LM 加上 damping $\lambda$：
$$
(J^TJ + \lambda I)\boldsymbol{\Delta}=J^T\mathbf{r}
$$
- $\lambda$ 大 → 接近 gradient descent（保守但穩）。
- $\lambda$ 小 → 接近 Gauss-Newton（快但風險高）。
- 實作：若下降成功就縮小 $\lambda$，失敗就放大。

LM 是電腦視覺估計中最常用的 general-purpose 非線性最小化器。

---

## 6. Gold Standard Algorithm for Homography

> **目標**：給 $n\geq 4$ 對 2D 點對應，求 MLE 的 $H$（同時算出最佳的 $\hat{\mathbf{x}}_i, \hat{\mathbf{x}}_i'$）。

> **Gold Standard Algorithm**
> 1. **Initialization**：用 Normalized DLT 或 RANSAC 得到初估。
> 2. **Geometric minimization**，二選一：
>    - **Sampson error**：
>      - 最小化 Sampson cost（閉式校正）。
>      - 用 LM over 9 個 $\mathbf{h}$ 分量。
>    - **Gold Standard（真正 reprojection）**：
>      - 先由 $H$ 初估每對的最佳 $\hat{\mathbf{x}}_i$；
>      - 對 $\{H, \hat{\mathbf{x}}_1,\dots,\hat{\mathbf{x}}_n\}$ 整個集合最小化 $\sum d(\mathbf{x}_i,\hat{\mathbf{x}}_i)^2 + d(\mathbf{x}_i',\hat{\mathbf{x}}_i')^2$；
>      - 點數多時使用 **sparse LM**（Jacobian 稀疏結構）。

Sparse LM 的竅門是：$\hat{\mathbf{x}}_i$ 跟其他 $\hat{\mathbf{x}}_j$ 無關，Hessian 有 block-diagonal 結構，能節省大量計算。

---

## 7. Robust Estimation：處理 Outliers

之前的 MLE 假設 outliers 已被清掉。實務上，兩張影像間的對應 **一定有錯配**（SIFT 把背景配到前景、重複紋理自相似）。Outliers 即使只有少數，最小平方都會被拉爆。

### 7.1 RANSAC（RANdom SAmple Consensus）

思想：**用隨機小樣本假設模型，看誰最被多數點認可**。

> **RANSAC Algorithm**
> 1. 從 $S$ 中隨機抽 $s$ 個點，算模型。
> 2. 計算所有點到模型的距離；距離 < $t$ 的稱為 **inliers**，組成 $S_i$。
> 3. 若 $|S_i| > T$（某閾值）→ 用 $S_i$ 內所有點重新估模型、終止。
> 4. 若不夠 → 另抽一組，回到 (i)。
> 5. 執行 $N$ 次後，取 **最大** 的 $S_i$，用該子集重估模型。

### 7.2 距離閾值 $t$

選 $t$ 使 inlier 被接受機率 $\alpha$（例 0.95）。在 zero-mean Gaussian 雜訊 $\sigma$ 下，$d_\perp^2/\sigma^2$ 服從 $\chi^2_m$ 分佈，$m$ 為模型的 **codimension**：

| Codim | 模型 | $t^2$（$\alpha=0.95$） |
|---|---|---|
| 1 | Line ($\mathbf{l}$)、Fundamental matrix ($F$) | $3.84\sigma^2$ |
| 2 | Homography ($H$)、Camera matrix ($P$) | $5.99\sigma^2$ |
| 3 | Trifocal tensor ($T$) | $7.81\sigma^2$ |

> （維度 + 餘維度 = 量測空間維度。H 的量測空間 4 維，模型為 2 維流形 → codim 2。）

### 7.3 需要多少次取樣 $N$？

要讓「至少一組抽到全 inlier」的機率達到 $p$（例 0.99）：
$$
(1-(1-e)^s)^N = 1-p\ \Rightarrow\ N=\frac{\log(1-p)}{\log(1-(1-e)^s)}
$$
其中 $e$ 是 outlier 比例、$s$ 是最小樣本數。

| $s\backslash e$ | 5% | 10% | 20% | 25% | 30% | 40% | 50% |
|---|---|---|---|---|---|---|---|
| 2 | 2 | 3 | 5 | 6 | 7 | 11 | 17 |
| 3 | 3 | 4 | 7 | 9 | 11 | 19 | 35 |
| 4 | 3 | 5 | 9 | 13 | 17 | 34 | 72 |
| 5 | 4 | 6 | 12 | 17 | 26 | 57 | 146 |
| 6 | 4 | 7 | 16 | 24 | 37 | 97 | 293 |
| 7 | 4 | 8 | 20 | 33 | 54 | 163 | 588 |
| 8 | 5 | 9 | 26 | 44 | 78 | 272 | 1177 |

Homography 是 $s=4$，所以 50% outlier 下只要 72 次 sample！**RANSAC 可怕地有效率**。

### 7.4 接受的 inlier 比例門檻

通常取 $T=(1-e)n$：收到大約「1 - outlier 率」比例的 inliers 就終止。

### 7.5 Adaptive RANSAC

$e$ 一開始不知道，保守估 50%。每次找到更好的 consensus：
1. 以目前最多 inliers 計算當前 $e = 1 - |\text{inliers}|/n$。
2. 重算 $N$（用公式）。
3. 若 sample_count ≥ $N$ → 停止。

**收斂**：實際 outlier 率越低，$N$ 估值降得越快。

### 7.6 Robust MLE（M-estimator）

RANSAC 完後做的 MLE 如果直接最小化所有 inliers 的平方誤差，任何殘留 outlier 都會干擾。改用 **robust cost function**：
$$
\rho(e) = \begin{cases} e^2 & |e|\leq t \\ t^2 & |e|>t \end{cases}
$$
總目標：
$$
\mathcal{D}_R = \sum_i \rho(d_{\perp i})
$$
效果：超過 $t$ 的點 cost **封頂**，不再拉動優化。此法又被稱為 **reclassifying**，因它在優化過程中會自動重新決定誰是 outlier。

其它 robust cost：Huber、Tukey biweight 等。

### 7.7 其它 robust 演算法

- **LMedS (Least Median of Squares)**：最小化中位數平方誤差；對 $<50\%$ outlier 有效，不需距離閾值，但計算量大。
- 新方法：PROSAC、MAGSAC、USAC 等。

---

## 8. Automatic Homography Estimation（整條 pipeline）

> **輸入**：兩張影像。
> **輸出**：兩張影像之間的 homography $H$。

> **Algorithm**
> 1. **Interest points**：每張影像偵測興趣點（Harris、FAST、SIFT DoG…）。
> 2. **Putative correspondences**：根據某相似度（SAD、SSD、ZNCC、SIFT descriptor）做初步配對。
>    - 若運動受限（例如小旋轉），可限制在「座標接近」的候選內搜尋。
>    - 先進：SIFT / SURF 等具 scale / rotation 不變性的描述子。
> 3. **RANSAC robust estimation**：重複 $N$ 次
>    - (a) 抽 4 對、算 $H$（用 Normalized DLT）。
>    - (b) 計算所有對應的 $d_\perp$。
>    - (c) 算 inliers（$d_\perp<t$）的數量。
>    - 選 inlier 最多的 $H$。
> 4. **Optimal estimation**：用 LM 最小化 ML cost（Sampson 或 reprojection），只用 inliers。
> 5. **Guided matching**：依新的 $H$ 預測位置、在預測點附近重找更多對應。
>    - 可重複 (4)、(5) 直到收斂。

### 8.1 Putative Correspondences 的細節

- **SAD**（sum of absolute differences）：快、不太準。
- **SSD**（sum of squared differences）：常用。
- **ZNCC**（zero-mean normalized cross-correlation）：抗亮度變化。
- **SIFT**：多尺度、多方向、對光照旋轉都穩健，現在幾乎是 baseline。

### 8.2 例子（投影片上的具體數字）

- 偵測 500 個興趣點 / 影像。
- 初始配對 268 個。
- RANSAC：outliers 117、inliers 151。
- 再做 guided matching：最終 **262** 個 inliers。

Guided matching 能從約 151 → 262，收穫很明顯，因為有了 $H$ 可以把搜尋範圍縮得很窄。

---

## 9. 小結與重點回顧

1. **估計問題 = 參數數 vs. 量測數**；Homography 8 DOF → 4 點就能解。
2. **DLT** 提供 $A\mathbf{h}=0$ 的線性／SVD 解，是 **代數誤差** 最小化。
3. **Normalization**（Isotropic scaling）對 DLT 是 **必備** 步驟，否則數值不穩；它也讓 DLT 對座標的選擇不敏感。
4. **Cost function 選擇** 決定統計性質：
   - 代數距離（快、初始化用）
   - 幾何距離（MLE，若一張精確）
   - 對稱轉移誤差（兩張都有雜訊的簡化）
   - 重投影誤差（完整 MLE）
   - Sampson error（幾何的一階近似，減少未知數）
5. **Gold Standard algorithm**：Normalized DLT 初始化 → LM 最小化 Sampson 或 reprojection error。
6. **RANSAC** 處理 outliers：抽樣 → 計 inliers → 最多者勝。需搭配：
   - 距離閾值（$\chi^2$ 表）
   - 取樣次數公式（函數 $e, s, p$）
   - Adaptive 策略
7. **自動 homography pipeline**：interest points → putative matches → RANSAC → LM refine → guided matching。

---

## 10. 常見面試 / 考試觀念題

**Q1. 為什麼 DLT 不是 Gold Standard？**
- 它最小化的是代數誤差 $\|A\mathbf{h}\|^2$，不是幾何／機率意義下的距離；在非 affine 情況，scale 因子 $w_i \hat w_i$ 隨點改變，並不對應 MLE。

**Q2. 為什麼要 normalization？**
- 原始像素座標各欄數值量級差 $10^4$，SVD 數值不穩定；normalize 後座標 ~ $O(1)$，DLT 精度可提升一到兩個量級；而且讓結果對座標選擇不敏感。

**Q3. Sampson error 跟 geometric error 差多少？**
- Sampson 是一階 Taylor 近似，在真解附近誤差 $O(\|\delta\|^2)$，通常極接近；點數一多、$\hat{\mathbf{x}}_i$ 作為變數太多時，Sampson 是 CP 值之選。

**Q4. RANSAC 為什麼這麼有效？**
- 關鍵是 $s$ 小：hypothesis 只要 $s$ 點都是 inlier，機率 $(1-e)^s$ 仍可觀；$N$ 次獨立嘗試的成功機率指數收斂到 1。

**Q5. 若兩張影像是同一平面的照片（或相機只原地旋轉），它們的對應真的是 homography 嗎？**
- 是，這是兩個 homography 存在的最常見物理情境。平面場景投影到兩個視角間就是一個 $H$；相機純旋轉沒有平移時（無 parallax）場景深度無關緊要，整張圖也只差一個 $H$。

---

## 11. 延伸閱讀

- Hartley & Zisserman, *MVGCV* 第 4 章（Estimation — 2D Projective Transformations）。
- Hartley, "In Defense of the Eight-Point Algorithm," PAMI 1997（normalization 重要性的經典）。
- Fischler & Bolles, "Random Sample Consensus," CACM 1981。
- Lowe, "Distinctive Image Features from Scale-Invariant Keypoints," IJCV 2004（SIFT）。
- 下一講會把 cost function / RANSAC 的這套方法論應用到 **Fundamental matrix** 的估計（兩視圖幾何）。
