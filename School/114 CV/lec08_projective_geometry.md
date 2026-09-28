#note

# Lecture 08 — Projective Geometry（射影幾何）

> 課程：NTU EE 114 Computer Vision｜講者：簡韶逸 教授
> 投影片來源致謝：Marc Pollefeys
> 參考教科書：Hartley & Zisserman, *Multiple View Geometry in Computer Vision*

---

## 0. 為什麼要學射影幾何？

電腦視覺裡，相機把 3D 世界「投射」到 2D 影像平面上，這個過程會破壞歐氏幾何的許多性質：
- **平行線會在影像中相交**（想像鐵軌消失在地平線）。
- **長度、角度、面積**都不再保留。
- **比例**（ratio）會被扭曲，只有少數「高階不變量」還留著（比如 cross-ratio）。

歐氏幾何（Euclidean）處理不了這些現象，因此我們把座標系擴張成 **射影空間（Projective Space, $\mathbb{P}^n$）**。射影幾何的核心思想：
1. 用 **齊次座標（homogeneous coordinates）** 把「無窮遠的方向」也變成正常的「點」。
2. 線性代數就能描述所有的 projective transformation（不需要分母裡藏除法的非線性）。
3. 不同種類的變換（Euclidean、Similarity、Affine、Projective）可以排成一個 **階層**，每一層保留的不變量（invariant）都比下一層少。

本講的大綱：

- **Projective 2D Geometry**：2D 的點、線、圓錐曲線、變換與不變量、1D projective、cross-ratio。
- **Projective 3D Geometry**：3D 點、平面、直線（Plücker 座標）、二次曲面（quadrics）。

---

## 1. 2D 齊次座標（Homogeneous Coordinates）

### 1.1 直線的齊次表示

一條 2D 直線的方程式：
$$
ax + by + c = 0
$$

注意到 $(ka)x + (kb)y + kc = 0$ 對任意 $k \neq 0$ 也是同一條直線，所以 $(a, b, c)^T \sim k(a, b, c)^T$。直線等於 **向量的等價類別**，我們寫成：
$$
\mathbf{l} = (a, b, c)^T
$$

### 1.2 點的齊次表示

點 $(x, y)$ 在直線 $\mathbf{l} = (a, b, c)^T$ 上 iff $ax + by + c = 0$。寫成內積：
$$
(x, y, 1)\begin{pmatrix}a\\b\\c\end{pmatrix} = 0 \ \Longleftrightarrow\ \mathbf{x}^T \mathbf{l} = 0
$$

定義齊次座標：
$$
\mathbf{x} = (x_1, x_2, x_3)^T \sim k(x_1, x_2, x_3)^T
$$

對應到 2D 實平面的點 $(x_1/x_3, x_2/x_3)^T$。**一個 2D 點需要 3 個數字，但只有 2 個自由度**（因為同比例視為同點）。

### 1.3 關鍵對稱關係

> 點 $\mathbf{x}$ 在線 $\mathbf{l}$ 上 $\ \Leftrightarrow\ \mathbf{x}^T \mathbf{l} = \mathbf{l}^T \mathbf{x} = 0$

這個對稱性是後面「Duality（對偶）原理」的起點。

---

## 2. 點與線的基本運算

### 2.1 兩條直線的交點
$$
\mathbf{x} = \mathbf{l} \times \mathbf{l}'
$$

### 2.2 過兩點的直線
$$
\mathbf{l} = \mathbf{x} \times \mathbf{x}'
$$

**直覺**：cross product 產生的向量同時垂直兩個輸入向量。令 $\mathbf{l} = \mathbf{x}\times\mathbf{x}'$，則 $\mathbf{l}^T\mathbf{x}=0$ 且 $\mathbf{l}^T\mathbf{x}'=0$，正好是兩點都在這條線上的條件。

### 2.3 例子（投影片上的 $x=1$、$y=1$ 兩線）

- $x=1$ 對應 $\mathbf{l}=(1,0,-1)^T$
- $y=1$ 對應 $\mathbf{l}'=(0,1,-1)^T$
- 交點 $\mathbf{l}\times\mathbf{l}' = (1,1,1)^T \Rightarrow (1,1)$ ✔

---

## 3. 無窮遠點（Ideal Points）與無窮遠線（Line at Infinity）

兩條 **平行線**：
$$
\mathbf{l}=(a,b,c)^T,\quad \mathbf{l}'=(a,b,c')^T
$$
它們在歐氏平面沒有交點，但在射影平面上：
$$
\mathbf{l}\times\mathbf{l}' = (b,\,-a,\,0)^T
$$
第 3 個分量為 0，代表這是 **無窮遠的點**（若除以 0 來轉換回歐氏座標，座標會發散）。

- 向量 $(b, -a)$ 正是直線的切向量（方向），$(a,b)$ 則是法向量。
- 所有 **方向相同** 的平行線共享同一個無窮遠點。

**無窮遠線（Line at Infinity）**：
$$
\mathbf{l}_\infty = (0,0,1)^T
$$
所有無窮遠點都在它上面（因 $(x_1,x_2,0)^T\cdot(0,0,1)^T=0$）。

重要結論：
$$
\mathbb{P}^2 = \mathbb{R}^2 \cup \mathbf{l}_\infty
$$
在 $\mathbb{P}^2$ 裡無窮遠點跟普通點沒有結構上的區別。

---

## 4. 射影平面模型（幾何圖像）

把 $\mathbb{P}^2$ 想像成 $\mathbb{R}^3$ 中 **通過原點的直線們**：
- 每條過原點的線 = 一個點。
- 每個過原點的平面 = 一條線。
- 把平面 $x_3=1$ 當作「實際的影像平面」，任何過原點的直線會跟它相交於一個點 → 這就是該 projective point 的歐氏表示。
- 如果那條線 **躺在 $x_3=0$ 平面上**，它永遠不跟 $x_3=1$ 相交 → 對應到無窮遠點。

兩個基本的入射關係：
1. 兩點唯一決定一條線。
2. 兩線唯一決定一個交點（含無窮遠情況）。

第二條正是射影幾何的亮點：**歐氏幾何中的特例（平行）在射影幾何裡不再是特例**。

---

## 5. Duality（對偶）原理

點與線在 $\mathbb{P}^2$ 中地位完全對稱：

| 點 | 線 |
|---|---|
| $\mathbf{x}^T \mathbf{l} = 0$ | $\mathbf{l}^T\mathbf{x}=0$ |
| $\mathbf{x} = \mathbf{l}\times\mathbf{l}'$ | $\mathbf{l}=\mathbf{x}\times\mathbf{x}'$ |

**對偶原理**：任何 2D 射影定理，把「點」跟「線」的角色互換，得到的敘述也一定是定理。

例：Pascal 定理跟 Brianchon 定理就是對偶。

---

## 6. 圓錐曲線（Conics）

2 次平面曲線：
$$
ax^2 + bxy + cy^2 + dx + ey + f = 0
$$
齊次化（$x\to x_1/x_3$、$y\to x_2/x_3$）後：
$$
ax_1^2 + bx_1x_2 + cx_2^2 + dx_1 x_3 + ex_2 x_3 + fx_3^2 = 0
$$
可寫成矩陣形式：
$$
\mathbf{x}^T C \mathbf{x}=0,\quad
C=\begin{pmatrix}a & b/2 & d/2\\ b/2 & c & e/2\\ d/2 & e/2 & f\end{pmatrix}
$$
$C$ 是 **3×3 對稱矩陣**，有 6 個未知數，但 scale 無關 ⇒ **5 自由度**。

### 6.1 五點定錐線

每個點 $(x_i, y_i)$ 給一條線性方程
$$
ax_i^2 + bx_iy_i + cy_i^2 + dx_i + ey_i + f = 0
$$
把 5 點堆疊起來得到 $5\times 6$ 矩陣，解其 null space 就是 $C$ 的參數。

### 6.2 切線

在 $C$ 上的點 $\mathbf{x}$ 的切線：
$$
\mathbf{l} = C\mathbf{x}
$$

**直覺**：$C$ 在 $\mathbf{x}$ 處的梯度即是切線法向。

### 6.3 Dual Conic（對偶圓錐曲線）

跟 $C$ 相切的直線 $\mathbf{l}$ 滿足：
$$
\mathbf{l}^T C^*\mathbf{l}=0
$$
當 $C$ 滿秩時 $C^* = C^{-1}$。這是一個「由所有切線構成的圓錐曲線」，也稱 **line conic / conic envelope**。

---

## 7. Projective Transformation（射影變換 / Homography）

### 7.1 定義

$h:\mathbb{P}^2 \to \mathbb{P}^2$ 是 projectivity iff 它保持 **共線性**（三個共線點映射後仍共線）。

**定理**：$h$ 是 projectivity ⇔ 存在可逆的 $3\times 3$ 矩陣 $H$ 使 $h(\mathbf{x})=H\mathbf{x}$。

$$
\mathbf{x}'=H\mathbf{x}, \quad H=\begin{pmatrix}h_{11}&h_{12}&h_{13}\\h_{21}&h_{22}&h_{23}\\h_{31}&h_{32}&h_{33}\end{pmatrix}
$$
9 個元素，scale 無關 ⇒ **8 DOF**。別名：**collineation = homography = projective transformation**。

### 7.2 平面對平面：central projection

兩個平面間的 central projection（從同一個投影中心望過去）恰好是 $\mathbf{x}'=H\mathbf{x}$。應用：

- **從斜拍影像恢復正視圖（removing projective distortion）**
- **Mosaicing / panorama**（相機原地旋轉拍攝，多張影像間用 H 對齊）
- **平面物件的 augmented reality**

### 7.3 從 4 點校正投影失真

選 4 個點在畫面裡，並且知道它們在「真實平面」的座標：
$$
x' = \frac{h_{11}x+h_{12}y+h_{13}}{h_{31}x+h_{32}y+h_{33}},\quad
y' = \frac{h_{21}x+h_{22}y+h_{23}}{h_{31}x+h_{32}y+h_{33}}
$$
乘開去掉分母後是線性的，每個對應點提供 2 條方程，4 點 → 8 方程 → 可解 8 DOF。**不需要相機校準**。

### 7.4 線與圓錐曲線的變換法則

給點變換 $\mathbf{x}'=H\mathbf{x}$：

| 物件 | 變換法則 | 推導 |
|---|---|---|
| 點 | $\mathbf{x}'=H\mathbf{x}$ | 定義 |
| 線 | $\mathbf{l}'=H^{-T}\mathbf{l}$ | 保持 $\mathbf{x}'^T\mathbf{l}'=\mathbf{x}^T\mathbf{l}=0$ |
| 圓錐曲線 | $C'=H^{-T}CH^{-1}$ | 代入 $\mathbf{x}^TC\mathbf{x}=0$ |
| 對偶圓錐 | $C'^{*}=HC^{*}H^T$ | 對線的變換平方化 |

這裡的關鍵觀念：**covariant（點）** 跟 **contravariant（線、法向量）** 的差異。

---

## 8. 變換的階層（Hierarchy of Transformations）

從「強約束」到「弱約束」排列：

| 類別 | 矩陣形式 | DOF | 保留的性質 |
|---|---|---|---|
| **Euclidean / Isometry** | $\begin{pmatrix}R & t\\0^T & 1\end{pmatrix}$, $R^TR=I$ | 3 | 長度、角度、面積 |
| **Similarity** | $\begin{pmatrix}sR & t\\0^T & 1\end{pmatrix}$ | 4 | 長度比例、角度、面積比例、圓形點 $I,J$ |
| **Affine** | $\begin{pmatrix}A & t\\0^T & 1\end{pmatrix}$ | 6 | 平行性、平行線長度比、面積比、$\mathbf{l}_\infty$ |
| **Projective** | $\begin{pmatrix}A & t\\\mathbf{v}^T & v\end{pmatrix}$ | 8 | 共線性、交切階數、cross-ratio |

### 8.1 Class I — Isometry（等距變換）

$$
\begin{pmatrix}x'\\y'\\1\end{pmatrix}
=\begin{pmatrix}\epsilon\cos\theta & -\sin\theta & t_x\\ \epsilon\sin\theta & \cos\theta & t_y\\ 0&0&1\end{pmatrix}
\begin{pmatrix}x\\y\\1\end{pmatrix}
$$

- $\epsilon = +1$：orientation-preserving（歐氏變換、剛體運動）
- $\epsilon = -1$：加上鏡射
- DOF = 3（1 旋轉 + 2 平移），**2 個對應點** 即可解。

### 8.2 Class II — Similarity（相似變換）

在 isometry 上加一個 **各向同性縮放 $s$**：
$$
H_S = \begin{pmatrix}sR & t\\ 0^T & 1\end{pmatrix},\ R^TR=I
$$
DOF = 4。**保形（equi-form）**：形狀不變，只改大小跟位置。保留角度、長度比。

### 8.3 Class III — Affine

$$
H_A = \begin{pmatrix}A & t\\ 0^T & 1\end{pmatrix}
$$
$A$ 是任意 $2\times 2$ 可逆矩陣，DOF = 6。可以分解為：
$$
A = R(\theta)\,R(-\phi)\,D\,R(\phi),\quad D=\begin{pmatrix}\lambda_1 & 0\\ 0 & \lambda_2\end{pmatrix}
$$
→ 多了 **非等向性縮放（non-isotropic scaling）**，所以比例跟角度都會改變，但平行性仍保留。**3 個對應點** 決定。

### 8.4 Class IV — Projective

$$
H_P = \begin{pmatrix}A & t\\ \mathbf{v}^T & v\end{pmatrix},\ \mathbf{v}=(v_1,v_2)^T
$$
DOF = 8。多出來的 2 個自由度（相對 affine）是 **無窮遠線的位置**，也就是說它會把無窮遠線搬到有限處，產生「消失點（vanishing points）、地平線（horizon）」。

### 8.5 Action on Line at Infinity

Affine 對無窮遠點：
$$
\begin{pmatrix}A & t\\ 0^T & 1\end{pmatrix}\begin{pmatrix}x_1\\x_2\\0\end{pmatrix}=\begin{pmatrix}Ax_1/x_2\\0\end{pmatrix}
$$
→ 無窮遠點仍在無窮遠（只是在無窮遠線上移動）。這就是 affine 的「保留 $\mathbf{l}_\infty$」特徵。

Projective：
$$
\begin{pmatrix}A & t\\ \mathbf{v}^T & v\end{pmatrix}\begin{pmatrix}x_1\\x_2\\0\end{pmatrix}=\begin{pmatrix}Ax_1/x_2\\v_1x_1+v_2x_2\end{pmatrix}
$$
→ 第 3 個分量不再是 0，所以無窮遠線被搬到有限處。這就是為什麼照片裡看得見消失點。

### 8.6 Projective 的唯一分解

$$
H = H_S H_A H_P
=\begin{pmatrix}sR & t\\0^T & 1\end{pmatrix}
 \begin{pmatrix}K & 0\\0^T & 1\end{pmatrix}
 \begin{pmatrix}I & 0\\\mathbf{v}^T & v\end{pmatrix}
=\begin{pmatrix}A & t\\ \mathbf{v}^T & v\end{pmatrix}
$$
- $K$：上三角、$\det K=1$
- $A = sRK + t\mathbf{v}^T$
- 若 $s>0$，分解唯一。

這個分解正是「先做 projective 扭曲 → 再 affine 拉伸 → 再相似（旋轉縮放平移）」的概念，對應到 **層層校正（rectification）** 的流程。

---

## 9. Cross-Ratio（交比）

一條線上 4 個共線點 $\mathbf{x}_1,\mathbf{x}_2,\mathbf{x}_3,\mathbf{x}_4$ 的 cross-ratio：
$$
\text{Cross}(\mathbf{x}_1,\mathbf{x}_2,\mathbf{x}_3,\mathbf{x}_4)=\frac{|\mathbf{x}_1\mathbf{x}_2|\,|\mathbf{x}_3\mathbf{x}_4|}{|\mathbf{x}_1\mathbf{x}_3|\,|\mathbf{x}_2\mathbf{x}_4|}
$$
其中 $|\mathbf{x}_i\mathbf{x}_j|=\det\!\begin{pmatrix}x_{i1}&x_{j1}\\x_{i2}&x_{j2}\end{pmatrix}$。

**cross-ratio 是 projective 變換下唯一的獨立不變量**。只要你知道影像中 4 個共線點的 cross-ratio，跟真實世界的 cross-ratio 是一樣的，可以反推尺度資訊。

---

## 10. Affine Rectification（從 Projective 還原到 Affine）

### 10.1 基本想法

任何 projective 變換都把「某條線 $\mathbf{l}$」搬到無窮遠線 $\mathbf{l}_\infty$ 的位置。如果我們 **能在影像中找出真實世界的無窮遠線**（通常是地平線 horizon），就可以構造一個 $H$ 把它送回 $(0,0,1)^T$，這樣剩下的變換只剩 affine。

### 10.2 構造方法

找到影像中的 $\mathbf{l}_\infty$ 的像 $\mathbf{l}=(l_1,l_2,l_3)^T$（$l_3\neq 0$），取：
$$
H'_P = \begin{pmatrix}1 & 0 & 0\\0 & 1 & 0\\ l_1 & l_2 & l_3\end{pmatrix}
$$
則 $H_P'^{-T}(l_1,l_2,l_3)^T = (0,0,1)^T = \mathbf{l}_\infty$。

### 10.3 怎麼找影像中的無窮遠線？

利用 **兩組真實世界中的平行線**：
- 每組平行線在影像中相交 → 消失點 $\mathbf{v}_1, \mathbf{v}_2$。
- 兩個消失點共線，連線即是 $\mathbf{l}_\infty$ 的像 = 地平線。

恢復後，**平行性、長度比、面積比** 都回來了，但角度還不準。

### 10.4 距離比例（Distance Ratios）

Affine rectification 完成後，一條線上三個共線點的距離比
$$
d(a,b):d(b,c)=a:b
$$
可以從像中直接量測（因為 affine 保留平行線長度比）。

---

## 11. Circular Points 與 Metric Rectification

### 11.1 圓形點

$$
I = (1, i, 0)^T,\qquad J=(1, -i, 0)^T
$$
這兩個 **複數** 點在每個歐氏圓上都會出現（因 $x_1^2+x_2^2+\ldots$ 在 $x_3=0, x_1^2+x_2^2=0$ 處的解）。

**代數直覺**：
$$
I=(1,0,0)^T+i(0,1,0)^T
$$
它編碼了 $x$、$y$ 兩個 **正交方向**。

**重要性質**：$I,J$ 在 $H$ 下是 fixed points iff $H$ 是 similarity。

### 11.2 對偶的圓形點圓錐

$$
C_\infty^* = IJ^T + JI^T = \begin{pmatrix}1&0&0\\0&1&0\\0&0&0\end{pmatrix}
$$
- Rank 2，退化圓錐。
- $C_\infty^*$ 在 $H$ 下是 fixed iff $H$ 是 similarity。

### 11.3 Projective 版的角度公式

歐氏下兩直線 $\mathbf{l}, \mathbf{m}$ 的夾角：
$$
\cos\theta = \frac{l_1m_1+l_2m_2}{\sqrt{(l_1^2+l_2^2)(m_1^2+m_2^2)}}
$$
projective 不變版：
$$
\cos\theta = \frac{\mathbf{l}^T C_\infty^*\mathbf{m}}{\sqrt{(\mathbf{l}^T C_\infty^*\mathbf{l})(\mathbf{m}^T C_\infty^*\mathbf{m})}}
$$
**正交（垂直）的條件**：
$$
\mathbf{l}^T C_\infty^*\mathbf{m}=0
$$

### 11.4 從影像中求 $C_\infty^*$ → Metric Rectification

$C_\infty^*$ 在一般 projective 變換下會變成：
$$
C_\infty^{*\prime} = (H_P H_A) C_\infty^* (H_P H_A)^T
=\begin{pmatrix}KK^T & K^T\mathbf{v}\\ \mathbf{v}^TK & \mathbf{v}^T\mathbf{v}\end{pmatrix}
$$
如果能從影像中估出 $C_\infty^{*\prime}$（用 5 組垂直線對即可 → 5 個約束 $\mathbf{l}^T C_\infty^*\mathbf{m}=0$），再對它做 SVD：
$$
C_\infty^{*\prime}=U\begin{pmatrix}1&0&0\\0&1&0\\0&0&0\end{pmatrix}U^T,\quad H=U
$$
就恢復到 similarity 層級（相差 scale + 旋轉 + 平移）。

### 11.5 兩階段法

1. **已做 affine rectification（$\mathbf{v}=0$）**：
   $\mathbf{l}^T\!\!\begin{pmatrix}KK^T & 0\\0^T & 0\end{pmatrix}\!\mathbf{m}=0$ 給出 2 條方程（$k_{11}^2+k_{12}^2$、$k_{12}k_{22}$、$k_{22}^2$）→ 再 2 組垂直線對即可。

2. **從純 projective 直接做**：
   需要 5 組正交對應（因 $C_\infty^*$ 有 5 自由度）。

---

## 12. Projective 3D Geometry

### 12.1 3D 點

$$
\mathbf{X}=(X_1,X_2,X_3,X_4)^T\in\mathbb{P}^3,\quad
\text{若 }X_4\neq 0,\ \mathbf{X}=(X/X_4,\ Y/X_4,\ Z/X_4,\ 1)^T
$$
projective 變換：$\mathbf{X}' = H\mathbf{X}$，$H$ 是 $4\times 4$ 可逆矩陣 → $4^2-1=15$ DOF。

**對偶**：$\mathbb{P}^3$ 裡點對偶於平面、線對偶於線（3D 的特殊對稱性）。

### 12.2 平面

$$
\pi_1 X + \pi_2 Y + \pi_3 Z + \pi_4 = 0\ \Leftrightarrow\ \boldsymbol{\pi}^T\mathbf{X}=0
$$
4 自由度（scale 無關後 3）。歐氏解釋：$\tilde{\mathbf{n}}=(\pi_1,\pi_2,\pi_3)^T$ 是法向量，$\pi_4/\|\tilde{\mathbf{n}}\|$ 是原點到平面的距離。

**變換規則**（跟 2D 線同型）：
$$
\boldsymbol{\pi}' = H^{-T}\boldsymbol{\pi}
$$

### 12.3 從三個點求平面

$$
\begin{pmatrix}\mathbf{X}_1^T\\\mathbf{X}_2^T\\\mathbf{X}_3^T\end{pmatrix}\boldsymbol{\pi}=0
$$
解 **right null space** 即可。

或用共面條件 $\det[\mathbf{X}\ \mathbf{X}_1\ \mathbf{X}_2\ \mathbf{X}_3]=0$ 展開：
$$
\boldsymbol{\pi} = (D_{234}, -D_{134}, D_{124}, -D_{123})^T
$$
其中 $D_{ijk}$ 是除去第 $i,j,k$ 行後的行列式。

### 12.4 從三個平面求點

對偶操作：求 $[\boldsymbol{\pi}_1\ \boldsymbol{\pi}_2\ \boldsymbol{\pi}_3]^T$ 的 right null space。

### 12.5 平面上參數化點

用 $\mathbf{X}=M\mathbf{x}$，$\mathbf{x}$ 是 3 維參數（當作 projective plane 的座標）。要求 $\boldsymbol{\pi}^T M = 0$，$M$ 不唯一，一種選法：
$$
M=\begin{pmatrix}\mathbf{p}\\I_{3\times 3}\end{pmatrix},\quad \mathbf{p}=(-b/a,\,-c/a,\,-d/a)^T
$$
（假設 $\boldsymbol{\pi}=(a,b,c,d)^T$）。

---

## 13. 3D 直線（Plücker 座標）

### 13.1 為什麼 3D 直線很尷尬？

在 $\mathbb{P}^3$ 裡，直線有 4 自由度（方向 2 + 位置 2），但沒辦法用單一向量 + 對等類 clean 表示。解決方法：**Plücker 表示**。

### 13.2 Span 表示

過兩點 $\mathbf{A},\mathbf{B}$ 的直線：
$$
W=\begin{pmatrix}\mathbf{A}^T\\\mathbf{B}^T\end{pmatrix}
$$
線上任一點 $\lambda\mathbf{A}+\mu\mathbf{B}$。

### 13.3 對偶表示（Intersection of two planes）

$$
W^{*}=\begin{pmatrix}\mathbf{P}^T\\\mathbf{Q}^T\end{pmatrix},\quad W W^{*T} = W^* W^T = 0_{2\times 2}
$$

### 13.4 Plücker Matrix（推薦）

$$
L = \mathbf{A}\mathbf{B}^T - \mathbf{B}\mathbf{A}^T
$$
- 4×4、反對稱（skew-symmetric）、rank 2。
- 真正自由度 4。
- 跟 2D 的 $\mathbf{l}=\mathbf{x}\times\mathbf{y}$ 類比。
- 跟 $\mathbf{A},\mathbf{B}$ 的選擇無關（只差 scale）。
- 變換：$L'=HLH^T$。

**例**：$x$ 軸（$\mathbf{A}=(0,0,0,1)^T$、$\mathbf{B}=(1,0,0,0)^T$）
$$
L=\begin{pmatrix}0&0&0&-1\\0&0&0&0\\0&0&0&0\\1&0&0&0\end{pmatrix}
$$

### 13.5 對偶 Plücker

$$
L^* = \mathbf{P}\mathbf{Q}^T - \mathbf{Q}\mathbf{P}^T
$$
變換：$L^{*\prime} = H^{-T}L^*H^{-1}$。

$L$ 跟 $L^*$ 的元素對應：
$$
l_{12}:l_{13}:l_{14}:l_{23}:l_{42}:l_{34} = l_{34}^*:l_{42}^*:l_{23}^*:l_{14}^*:l_{13}^*:l_{12}^*
$$

### 13.6 入射與連接公式

| 操作 | 公式 |
|---|---|
| 點 + 線 → 平面 | $\boldsymbol{\pi}=L^*\mathbf{X}$ |
| 點在線上 | $L^*\mathbf{X}=0$ |
| 平面 ∩ 線 → 點 | $\mathbf{X}=L\boldsymbol{\pi}$ |
| 線在平面中 | $L\boldsymbol{\pi}=0$ |
| 兩線共面 | $\boldsymbol{\pi}(L_1,L_2)=0$ |

---

## 14. Quadrics（二次曲面）與 Dual Quadrics

### 14.1 定義

$$
\mathbf{X}^T Q \mathbf{X} = 0,\quad Q\in\mathbb{R}^{4\times 4}\text{ 對稱}
$$
- 自由度 $\binom{4+1}{2}-1 = 9$。
- 一般 9 點決定一個 quadric。
- $\det Q=0$ → degenerate quadric。
- **極平面（polar plane）**：$\boldsymbol{\pi}=Q\mathbf{X}$。
- **平面與 quadric 的交** 是圓錐曲線：$C = M^T Q M$（$M$ 是平面上點的參數化）。
- 變換：$Q' = H^{-T}QH^{-1}$。

### 14.2 Dual Quadric

對平面的方程：
$$
\boldsymbol{\pi}^T Q^* \boldsymbol{\pi} = 0
$$
若 $Q$ 非退化，$Q^*=Q^{-1}$。變換：$Q^{*\prime}=HQ^*H^T$。

### 14.3 Quadrics 的分類（依 rank 與 signature）

| Rank | Sign. | 對角 | 方程 | 實現 |
|---|---|---|---|---|
| 4 | 4 | (1,1,1,1) | $X^2+Y^2+Z^2+1=0$ | 無實點 |
| 4 | 2 | (1,1,1,-1) | $X^2+Y^2+Z^2=1$ | **球** |
| 4 | 0 | (1,1,-1,-1) | $X^2+Y^2=Z^2+1$ | 單葉雙曲面 |
| 3 | 3 | (1,1,1,0) | $X^2+Y^2+Z^2=0$ | 單點 |
| 3 | 1 | (1,1,-1,0) | $X^2+Y^2=Z^2$ | **錐** |
| 2 | 2 | (1,1,0,0) | $X^2+Y^2=0$ | 單直線 |
| 2 | 0 | (1,-1,0,0) | $X^2=Y^2$ | 兩平面 |
| 1 | 1 | (1,0,0,0) | $X^2=0$ | 單平面 |

Projective 等價於球面的：球、橢球、雙葉雙曲面、拋物面（從投影角度來說都是「凸且封閉」）。

**Ruled quadrics（含直線）**：單葉雙曲面、錐、兩平面。

---

## 15. 3D 變換的階層

| 類別 | 矩陣 | DOF | 不變量 |
|---|---|---|---|
| **Projective** | $\begin{pmatrix}A & t\\ \mathbf{v}^T & v\end{pmatrix}$ | 15 | 相交性、切接性 |
| **Affine** | $\begin{pmatrix}A & t\\ 0^T & 1\end{pmatrix}$ | 12 | 平面平行性、體積比、質心、$\Pi_\infty$ |
| **Similarity** | $\begin{pmatrix}sR & t\\ 0^T & 1\end{pmatrix}$ | 7 | absolute conic $\Omega_\infty$ |
| **Euclidean** | $\begin{pmatrix}R & t\\ 0^T & 1\end{pmatrix}$ | 6 | 體積 |

自由度組成：Euclidean = 3 旋轉 + 3 平移。Similarity +1 各向同性縮放。Affine +2 旋轉 + 3 非等向縮放（$A$ 的 SVD）。Projective +3（$\mathbf{v}$ 的 3 個值）。

### 15.1 Screw Decomposition

任何 3D 旋轉 + 平移都可以分解為：
- 繞某條 **螺旋軸（screw axis）** 旋轉。
- 沿這條軸 **平移**。

把平移分成 $t = t_{/\!/} + t_\perp$，$t_{/\!/}$ 平行於旋轉軸、$t_\perp$ 垂直。垂直的部分可以透過「旋轉軸適當平移」被吸收掉，剩下平行分量就是沿軸的平移量。

---

## 16. 無窮遠平面 $\Pi_\infty$、Absolute Conic $\Omega_\infty$ 與它的對偶

### 16.1 Plane at Infinity $\Pi_\infty$

$$
\Pi_\infty = (0,0,0,1)^T
$$

**性質**：
1. 包含所有方向向量 $D=(X_1,X_2,X_3,0)^T$。
2. 兩平面平行 ⇔ 它們的交線在 $\Pi_\infty$ 上。
3. 直線平行於另一直線（或平面）⇔ 它們的交點在 $\Pi_\infty$ 上。

**$\Pi_\infty$ 在 $H$ 下是 fixed ⇔ $H$ 是 affinity**。

**證明（投影片上）**：若 $H$ 是 affine，$H_A^{-T}$ 的最後一行是 $(0,0,0,1)$，於是 $H_A^{-T}\Pi_\infty=\Pi_\infty$。

### 16.2 Absolute Conic $\Omega_\infty$（之後會在 Camera Calibration 用到）

- 躺在 $\Pi_\infty$ 上的一個「虛球面」：$X_1^2+X_2^2+X_3^2=0,\ X_4=0$。
- Similarity fixes $\Omega_\infty$ pointwise。
- 它的像 $\omega$（IAC）用於相機內部參數校準。
- Dual absolute conic $\Omega_\infty^*=\text{diag}(1,1,1,0)$。

這些在後面幾講（camera calibration、stratified reconstruction）會大量用到，本講先知道它存在。

---

## 17. 小結與重點回顧

1. **齊次座標** 把射影變換變成線性代數，把「無窮遠」變成正常點。
2. **點與線對偶**：$\mathbf{x}^T\mathbf{l}=0$ 是整個 2D projective 的骨架。
3. **變換階層**（從嚴到寬）：Euclidean ⊂ Similarity ⊂ Affine ⊂ Projective。每下一層失去一種不變量：
   - Euclidean → Similarity：失去絕對長度
   - Similarity → Affine：失去角度、形狀、圓形點 $I,J$ 被挪位
   - Affine → Projective：失去平行性、$\mathbf{l}_\infty$ 被挪位
4. **Cross-ratio** 是唯一的 projective 不變量。
5. **Rectification** 的分層策略：
   - Projective → Affine：定位影像中的 $\mathbf{l}_\infty$。
   - Affine → Similarity：定位 $I, J$（或等價地 $C_\infty^*$）。
6. **3D 版本** 的結構類似：
   - 點 ↔ 平面對偶。
   - 直線用 **Plücker matrix**（4×4 skew-symmetric, rank 2）描述。
   - Quadric 是 2 次曲面，dual 在平面空間。
   - $\Pi_\infty$ 保留 ⇔ affine；$\Omega_\infty$ 保留 ⇔ similarity。

---

## 18. 延伸閱讀

- Hartley & Zisserman, *MVGCV* 第 2、3 章（2D/3D Projective Geometry）。
- Faugeras, *Three-Dimensional Computer Vision* 第 2 章。
- 下一講會介紹 **如何從資料估計這些變換**（homography estimation via DLT、RANSAC、Gold Standard）→ 對應 lec09。
