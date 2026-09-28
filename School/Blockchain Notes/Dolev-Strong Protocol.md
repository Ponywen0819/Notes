#note

# Dolev-Strong Protocol

由 Dolev 與 Strong (1983) 提出的**拜占庭廣播 (Byzantine Broadcast, BB)** 協定，是第一個證明：在同步網路下，只要使用**數位簽章**，即使有 $n - 1$ 個拜占庭節點仍能達成廣播一致性的演算法。

# 問題設定 (Byzantine Broadcast)
網路中有 $n$ 個節點 $P_1, P_2, \dots, P_n$，其中：
- 一個指定的**傳送者 (sender)** $P_1$，持有一個輸入位元 $b \in \{0, 1\}$。
- 至多 $f$ 個節點是**腐壞節點 (corrupted / Byzantine)**，可能任意說謊、不回應、串謀。
- 其他節點為**誠實節點 (honest)**。

協定執行結束後，每個節點 $P_i$ 輸出一個值 $y_i$，必須滿足：
- **Validity**：若 $P_1$ 誠實，則所有誠實節點輸出 $y_i = b$。
- **Consistency / Agreement**：所有誠實節點輸出相同值（即使 $P_1$ 是拜占庭）。

## 系統假設
- **同步網路**：訊息會在已知 round 時間內送達；協定按 round 推進。
- **數位簽章 (digital signature)**：每個節點都有一對 PKI 公私鑰；簽章不可偽造，誠實節點可驗證任何節點的簽章。
- **腐壞節點數量上界** $f \leq n - 1$（這是本協定的關鍵威力，能容忍幾乎所有節點作惡，只要還有一個誠實節點）。

# 核心想法
讓每一個聲稱來自 $P_1$ 的訊息攜帶**所有轉發者的簽章鏈**，誠實節點只接受「簽章鏈長度足夠且未被偽造」的訊息。把總共執行 $f + 1$ 個 round——比腐壞節點多一輪——這樣**至少有一輪由誠實節點傳遞訊息**，誠實節點之間就能對齊看到的內容。

每個節點 $P_i$ 在過程中維護一個集合
$$
\mathrm{extr}_i \subseteq \{0, 1\}
$$
存放它「曾經見過合法簽章鏈」的位元值，最終可能是 $\emptyset, \{0\}, \{1\}, \{0, 1\}$ 之一。

# 協定流程
記號：$\langle b \rangle_{i_1, i_2, \dots, i_k}$ 表示「位元 $b$ 依序被 $P_{i_1}, P_{i_2}, \dots, P_{i_k}$ 簽過」的訊息。

## Round 0（傳送者廣播）
$P_1$ 對輸入 $b$ 簽章，產生 $\langle b \rangle_1$，送給其他所有節點。

## Round $r$（$r = 1, 2, \dots, f$）
每個節點 $P_i$ 收到一條訊息 $\langle b \rangle_{1, j_1, \dots, j_{r}}$，若同時滿足：
1. 第一個簽章是 $P_1$ 的；
2. 簽章鏈包含 $r$ 個**不同**的後續簽章者；
3. 每一個簽章都驗證通過；
4. $b \notin \mathrm{extr}_i$，

則：
- 將 $b$ 加入 $\mathrm{extr}_i$；
- 在訊息尾端附上自己的簽章 $\langle b \rangle_{1, j_1, \dots, j_r, i}$，轉發給其他所有節點。

注意：每個節點對同一個位元值**最多轉發一次**（因為加入 $\mathrm{extr}_i$ 後就不再處理）。

## Round $f + 1$（輸出）
最後 $P_i$ 看自己的 $\mathrm{extr}_i$：
$$
y_i = \begin{cases}
b, & \text{若 } \mathrm{extr}_i = \{b\}\\
0, & \text{若 } \mathrm{extr}_i = \emptyset \text{ 或 } \mathrm{extr}_i = \{0, 1\}
\end{cases}
$$
即只有恰好一個值在集合中時就輸出該值，否則輸出預設值 $0$（這是論文常見的選擇，也可以選 $\bot$）。

# 範例
假設 $n = 4$，$f = 1$，傳送者 $P_1$ 想廣播 $b = 1$。

## 情境 A：$P_1$ 誠實
- **Round 0**：$P_1$ 送 $\langle 1 \rangle_1$ 給 $P_2, P_3, P_4$。
- **Round 1**：$P_2, P_3, P_4$ 都驗證 $P_1$ 的簽章，將 $1$ 加入各自的 $\mathrm{extr}$，並轉發 $\langle 1 \rangle_{1, i}$。
- **Round 2 ($f + 1$)**：所有誠實節點 $\mathrm{extr}_i = \{1\}$，輸出 $y_i = 1$。✅ Validity 成立。

## 情境 B：$P_1$ 是拜占庭，且只跟 $P_2$ 說 $1$
$P_1$ 在 Round 0 只送 $\langle 1 \rangle_1$ 給 $P_2$，不送給 $P_3, P_4$。
- **Round 1**：$P_2$ 驗證後加入 $1$，轉發 $\langle 1 \rangle_{1, 2}$ 給 $P_3, P_4$。$P_3, P_4$ 收到，鏈長 = 2 = round 編號 ✓，加入 $1$ 並準備在 round 2 轉發。
- **Round 2 ($f + 1$)**：$P_3, P_4$ 也轉發。所有誠實節點 $\mathrm{extr}_i = \{1\}$，輸出 $y_i = 1$。✅ Consistency 成立。

關鍵：$f + 1 = 2$ 輪保證即使 $P_1$ 在 round 0 偷偷只告訴一個誠實節點，這個誠實節點也來得及在剩下的 round 把訊息傳出去。

## 情境 C：$P_1$ 拜占庭企圖製造分歧
$P_1$ 嘗試在最後一刻 (round $f$) 才丟出新訊息，期望某些節點來不及傳。但因為合法訊息要求簽章鏈長度等於 round 數，而**最後一輪 $f + 1$ 已經沒人會再轉發**，所以任何在 round $f$ 才出現的新位元值要被某個誠實節點接受，**該訊息在 round $f$ 必須已經有 $f$ 個不同簽章**——其中至少一個來自誠實節點，而誠實節點若在 round $r < f$ 就簽了，他必然也轉發給其他誠實節點，於是大家在 round $f$ 結束前都看得到。

# 正確性
## Validity（$P_1$ 誠實時）
$P_1$ 在 round 0 簽好正確的 $b$ 廣播。所有誠實節點在 round 1 收到 $\langle b \rangle_1$，驗證通過後將 $b$ 加入 $\mathrm{extr}$。由於簽章不可偽造，攻擊者無法產生 $\langle b' \rangle_1$（$b' \neq b$），因此沒有節點會將 $b'$ 加入 $\mathrm{extr}$。最終所有誠實節點 $\mathrm{extr}_i = \{b\}$，輸出 $b$。

## Consistency（$P_1$ 拜占庭時）
**關鍵引理**：若任一誠實節點在 round $r$ 結束時將位元 $b$ 加入 $\mathrm{extr}$，則所有誠實節點在 round $f + 1$ 結束時都會把 $b$ 加入 $\mathrm{extr}$。

證明分兩種情況：
- **若 $r \leq f$**：該誠實節點 $P_i$ 在 round $r$ 加入後會轉發 $\langle b \rangle$ 給所有人，鏈長 = $r + 1 \leq f + 1$。其他誠實節點在 round $r + 1$ 收到合法訊息，加入 $\mathrm{extr}$。
- **若 $r = f + 1$**：誠實節點 $P_i$ 在 round $f + 1$ 收到的訊息簽章鏈長度恰為 $f + 1$，包含至少一個誠實節點（因為腐壞節點只有 $f$ 個）。設該誠實節點為 $P_j$，且 $P_j$ 在 round $r' \leq f$ 就已將 $b$ 加入 $\mathrm{extr}_j$ 並廣播。由前一情況可知，所有誠實節點在 round $r' + 1 \leq f + 1$ 結束時都已收到並加入。

因此所有誠實節點最終的 $\mathrm{extr}$ 集合**完全相同**，輸出一致。✅

# 複雜度分析
| 維度 | 數量 |
| --- | --- |
| Round 數 | $f + 1$ |
| 每個誠實節點轉發次數 | 最多 2 次（每個位元值至多一次） |
| 總訊息數 | $O(n^2)$ |
| 每條訊息大小 | $O(f \cdot \kappa)$，$\kappa$ 為簽章長度 |
| 總通訊量 | $O(n^2 \cdot f \cdot \kappa)$ |

訊息的**簽章鏈**長度隨 round 線性增長，是這個協定主要的通訊開銷來源。

# 與其他協定的比較
| 協定 | 假設 | 容錯界 | Round 數 | 是否需簽章 |
| --- | --- | --- | --- | --- |
| **Naive Broadcast** | 同步 | 0（任一節點作惡即失敗） | 1 | ✗ |
| **Dolev-Strong** | 同步 + PKI | $f \leq n - 1$ | $f + 1$ | ✓ |
| **PBFT (Castro-Liskov)** | 部分同步 | $f < n / 3$ | $O(1)$（穩態） | ✓ |
| **Byzantine Agreement (no signatures)** | 同步 | $f < n / 3$ | $f + 1$ | ✗ |

關鍵啟發：
- **數位簽章把容錯界從 $n/3$ 拉到 $n - 1$**，代價是必須有 PKI 與簽章驗證能力。
- $f + 1$ 是同步 BB 的**最佳 round 下界**，Dolev-Strong 達到此下界。

# 限制與後續發展
- **同步假設**強烈，實際網路常為部分同步或非同步，因此實務上多用 PBFT、HotStuff 等部分同步協定。
- **通訊量** $O(n^2 f)$ 對大規模網路偏高；後續有以閥值簽章 (threshold signature) 等技術壓縮的變體。
- 是現代區塊鏈共識（如 Algorand 的 BA⋆、Tendermint）背後 BFT 思想的歷史源頭之一。

# 參考
- Dolev, D., & Strong, H. R. (1983). *Authenticated algorithms for Byzantine agreement.* SIAM Journal on Computing.
- 課程筆記：[[Week 9 Note]]、[[Week 10 Note]]
