#paper

[SWE-Explore: Benchmarking How Coding Agents Explore Repositories (Zhang et al., 2026)](https://arxiv.org/abs/2606.07297)

> arXiv:2606.07297v1 ｜ 2026-06-05 ｜ cs.SE / cs.CL ｜ CC BY-NC-ND 4.0
> 作者:Shaoqiu Zhang¹、Yuhang Wang¹、Jialiang Liang²、Yuling Shi¹、Wenhao Zeng¹、Maoquan Wang⁴、Shilin He⁵、Ningyuan Xu⁴、Siyu Ye³、Kai Cai⁴、Xiaodong Gu¹(通訊)
> ¹上海交大 ²新疆大學 ³UIUC ⁴獨立研究者 ⁵香港中文大學
> Code: github.com/Qiushao-E/SWE-Explore-Bench ｜ Data: HuggingFace SWE-Explore-Bench

---

# 一、問題與動機

## 1.1 SWE-bench 的協定:優點也是限制
`SWE-bench` 這類 repo 級 benchmark 把每次修復嘗試化約成**單一 pass/fail 預測**(resolved / unresolved)。這讓模型好比較,卻**掩蓋了成功的內部機制** —— 一個整體分數看不出到底是哪一步成功或失敗:讀對程式碼、定位 bug、生成補丁、還是驗證修復。

## 1.2 兩種被混在一起的失敗模式
一旦跳出「單一預測」,會看到兩種截然不同的失敗:
1. **探索失敗**:agent **沒找到**修復所需的相關程式碼。
2. **綜合失敗**:agent 讀到了足夠證據,卻**沒能寫出正確補丁**。

後者現有可執行 benchmark 抓得到,**前者卻大多被隱藏**。真實 repo 有上千個檔案(本資料集平均 **759 檔、18 萬行**非測試程式碼),要判斷「哪幾行**承載了**某 issue 的證據」,即使對最終解出它的 agent 也很難。

## 1.3 SWE-Explore 的定位
把**倉庫探索 (repository exploration)** 單獨拉出來當**可比較的評測目標**:給一個 issue + repo,explorer 回傳一份**排序的程式碼區段清單**,在固定行數預算下,對照「成功解出同一 issue 的獨立軌跡實際查閱過的程式碼」評分。稀疏檢索器、互動式 agent、長上下文選擇器,都被當成「**同一種 ranked region list 產生器**」來公平比較 —— 且**不必寫補丁、不必驗證**。

> 一句話:**不看你補得對不對,看你有沒有、且多早找到該讀的那幾行。** 把黑箱的 resolve rate 拆成 line-level 可量測的探索能力。

---

# 二、任務定義 (Task Formulation)

- **輸入**:issue $q$ + repo 快照 $\mathcal{R}$。
- **輸出**:排序區段清單
$$f:(q,\mathcal{R})\mapsto P=(r_1,\dots,r_K),\qquad r_i=(p_i,s_i,e_i)$$
每個區段 $r_i$ = 檔案路徑 $p_i$ + 行區間 $[s_i,e_i]$。
- **約束**:主實驗固定 **$K=5$**(對齊 ground-truth 平均 4.7 個核心區段)、行數預算 **$B=500$**。
- **explorer 不能**:寫最終補丁、存取 ground truth、與 repo 互動(它只是產生清單)。

---

# 三、Ground-Truth 標註 —— 最大創新

## 3.1 為什麼不人工標
手工標「必要 context」在此規模下**又貴又難一致**:數百個跨 10 語言的 repo 級 issue,不同標註者對 helper / config / test 的邊界畫法不同。

## 3.2 用「成功軌跡」反推(trajectory-grounded)
改從**已驗證成功的 agent 軌跡**萃取監督訊號。每個保留的 instance 要求**至少 2 條**來自強模型的成功軌跡(GPT-5.4、Gemini-3-Pro、Sonnet-4.6、GLM-5.1、Kimi-K2.6),在原 harness 下解出。
核心假設:**不同成功路徑都反覆讀到的區段 = 核心 context**(繞不開的證據)。

## 3.3 三步流程
1. **抽取 reads**:從每條軌跡收集所有能對應到明確「檔案–行區間」的**讀取動作** —— 編輯器開檔、`cat`/`head`/`tail`/`sed -n`、`grep -n` 命中行 —— 正規化成 $(p,s,e)$。無法明確對應的(如自由格式終端互動)**直接丟棄,不做啟發式擴張**,保證監督嚴格 grounded。
2. **聚合成核心 / optional**:
   - **交集** $R_{int}=\bigcap_{\tau\in T}R(\tau)$:保守核心候選。**逐檔、行級**取交集 —— 例:兩條軌跡分別讀 `parser.py:40–80` 與 `parser.py:60–100`,交集貢獻 `parser.py:60–80`。
   - **optional** $R_{opt}^{(m)}$:某模型家族特有、落在交集外的讀取。
3. **精修 + 人工審核**:LLM 把「承重的 optional 讀取」少量提升進核心 → 作者**逐一人工審核**、剔除無據區段。最終 $R_{core}$(精修後的交集)是主實驗**唯一評分目標**。

## 3.4 資料統計
- **來源**:SWE-bench Verified + SWE-bench-Pro + SWE-bench Multilingual,經成功驗證過濾後保留。
- **規模**:**848 instance、203 repo、10 種語言**。
- **每 instance 平均**:核心 4.3 檔 / 4.7 區段 / 1,578 行;issue 文字 191 詞;來源軌跡 2.9 條;實際被補丁改動 1.4 檔。
- **repo 平均**:759 檔、179.6K 行(非測試)。

> 巧妙處:**讓多個強 agent 去解、取它們共同繞不開的部分當答案** —— 幾乎零人工標註,又反映真實修復行為。代價:純交集偏保守、union 太吵,refined core 是折衷的經驗近似(附錄有 ablation)。

---

# 四、評估三維度 (Metrics)

記 $L(P)$ = 預測覆蓋的 $(p,\ell)$ 行集合,$Y=L(R_{core})$ = 核心行集合。

## 4.1 Coverage & Accuracy(覆蓋與準確)
- **Line Precision** $=|L(P)\cap Y|/|L(P)|$
- **Line Recall** $=|L(P)\cap Y|/|Y|$;**F1** 為兩者調和平均。
- **HitFile**:命中的 ground-truth 檔案比例(碰到對的「鄰里」)。
- **HitRegion**:至少被一個預測重疊到的核心區段比例。

## 4.2 Ranking Under Budget(預算下的排序)
- **nDCG@B**:每個區段 gain $g_i$ = 它**新覆蓋的核心行數**,折扣 $\log_2(i+2)$,只累計「累積行數不超過 $B$」的最長前綴,再對同預算下的理想排序正規化。
$$\text{DCG@}B=\sum_{i\in P_{\le B}}\frac{g_i}{\log_2(i+2)}$$
  - **用行數預算(而非 rank cutoff)的用意**:一個又長又沒料、把預算吃光的區段,會被罰得跟「漏掉有用內容」一樣重。
- **FUH(First Useful Hit)** $=1-i^\star/|P|$,$i^\star$ 為第一個命中核心的排名 → 獎勵**越早**浮出證據。

## 4.3 Efficiency & Noise(效率與雜訊)
- **Context Efficiency**:預測可見行落在 $L(R_{core})\cup L(R_{opt})$ 的比例(有多少是 grounded 證據 vs 離題)。
- **Noise Rate**:既不碰核心、也不碰 optional 的區段比例(純雜訊診斷)。

## 4.4 下游驗證(一次性 sanity check)
把 explorer 輸出的區段**以外的整個 repo 都遮掉**,只留這些區段當 context,交給固定 patcher 產補丁、用原 SWE-bench harness 判定 resolve → 檢驗「上游指標是否真的預測下游修復」。**這不是標準評測流程的一部分**,只用來背書指標。

---

# 五、實驗

## 5.1 設定
- **explorer 四類**:
  - **基線**:Oracle(直接回 $R_{core}$)、Random。
  - **稀疏檢索**:BM25、TF-IDF、Potion(輕量 dense retriever)。
  - **通用 coding agent**:Claude Code、Codex、OpenHands、Mini-SWE-Agent、AweAgent。
  - **專用定位器**:AutoCodeRover、LocAgent、OrcaLoca、CoSIL。
- **主指標**(高相關、低冗餘):Precision、nDCG@500、HitFile、Context Efficiency;輔以 Recall/F1/hit/noise 當參考。
- 除 Table 5(比模型)外,所有 agentic explorer 都用 **GPT-5.4** 當底層模型。

## 5.2 指標是否預測下游修復?(§4.2)
在 $n=150$ 子集上,把各 explorer 的 5 區段餵給固定 Mini-SWE-Agent patcher(GPT-5.4 與 Gemini-3-Pro,取平均 resolve rate),算上游指標與下游 resolve 的相關:

| 指標 | Pearson $r$ | Spearman $\rho$ |
| --- | --- | --- |
| **Context Efficiency** | **+0.950** | +0.739 |
| **FUH** | +0.928 | +0.675 |
| **Rec@100** | +0.926 | **+0.845**(最強 rank 相關) |
| **HitFile** | +0.925 | +0.695 |
| nDCG@500 | +0.921 | +0.460 |
| Noise Rate | −0.812 | −0.562 |

**啟示**:最強訊號**不是純檔案層、也不是純 recall** —— Context Efficiency 最高(有用的 context 要**既相關又精簡**);Rec@100 說明「**緊預算下的早期覆蓋**」最能預測修復。nDCG 類 Pearson 高但 Spearman 弱 → 能分「大檔次」但難排「相近的 explorer」。故論文主張**用混合指標組**,而非單一分數。

## 5.3 下游 resolve rate(Table 3,GPT-5.4 patcher)
| Explorer | Resolve % | | Explorer | Resolve % |
| --- | --- | --- | --- | --- |
| **Oracle** | 59.7 | | OrcaLoca | 45.3 |
| **CoSIL** | **59.3** | | AutoCodeRover | 44.7 |
| Codex | 50.3 | | LocAgent | 44.7 |
| Mini-SWE-Agent | 50.0 | | AweAgent | 41.3 |
| Claude Code | 48.0 | | TF-IDF | 26.0 |
| OpenHands | 47.7 | | RAG(Potion) | 23.3 |
| | | | BM25 | 12.7 / Random 4.7 |

> **CoSIL(59.3)幾乎追平 Oracle(59.7)**;稀疏檢索遠遠落後(BM25 僅 12.7)。

## 5.4 探索品質(Table 6,K=5,GPT-5.4)—— 節錄關鍵欄
| Explorer | Prec | **Recℓ** | F1 | HitFile | nDCG@500 | CtxEff | Noise↓ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Oracle | 1.000 | 0.953 | 0.964 | 0.923 | 0.858 | 1.000 | 0.000 |
| BM25 | 0.055 | 0.021 | 0.024 | 0.079 | 0.132 | 0.087 | 0.910 |
| TF-IDF | 0.117 | 0.049 | 0.054 | 0.140 | 0.223 | 0.190 | 0.821 |
| OpenHands | 0.489 | 0.179 | 0.209 | 0.645 | 0.867 | 0.737 | 0.245 |
| Claude Code | 0.598 | 0.154 | 0.202 | **0.667** | 0.938 | 0.829 | 0.186 |
| Codex | 0.523 | 0.194 | 0.223 | 0.649 | 0.901 | 0.762 | 0.249 |
| AweAgent | 0.577 | 0.140 | 0.182 | **0.682** | **0.954** | **0.829** | 0.191 |
| AutoCodeRover | **0.680** | 0.233 | 0.291 | 0.280 | 0.720 | 0.738 | **0.034** |
| **CoSIL** | 0.581 | **0.788** | **0.602** | 0.544 | 0.824 | **0.898** | 0.471 |

## 5.5 六個關鍵發現
1. **Agentic 探索明顯高於非 agentic 檢索一階**:BM25/TF-IDF/Potion 貼近 Random;每個 agent 都遠高。→ 探索**不是**一次性 lexical/embedding 檢索抓得到的,**多步互動是必要的**。
2. **低 F1 主要是 recall 問題**:通用 agent HitFile 與 nDCG 都高,但 **Recℓ 只有 ~0.14–0.19** → 常常**早早找到看似對的檔案,卻漏掉大量該讀的具體行**。廣度覆蓋才是瓶頸,不是精度。
3. **換模型只移動 operating point、不解決瓶頸**(Table 5,同 Mini-SWE-Agent 換 LLM):GPT 家族最強(GPT-5.4 HitFile 0.655、更精簡;GPT-5.4-mini 覆蓋更多、排序更早),Kimi-K2.6/Sonnet-4.5 中段,GLM-4.7/Gemini-3-Pro(HitFile 0.369)落後。但**全部都是「檔案強、行弱」的同一形狀** → 高 recall 靠的是**更好的探索機制,不是更強的補丁模型**。
4. **通用 agent 行為驚人相似**:Claude Code/Codex/OpenHands/Mini-SWE/AweAgent 幾乎佔同一 operating point(高檔案命中、早排序、精簡 context、低行 recall)。→ 研究探索子問題**不一定需要複雜的修復 harness**。
5. **專用定位器只有在「拓寬搜尋」時才有幫助**:AutoCodeRover 精確但保守(Prec 0.680 但 HitFile 僅 0.280);OrcaLoca 雜訊極低(0.003)卻漏很多;LocAgent 像通用 agent。**唯一例外是 CoSIL** —— 靠**迭代式 code-graph 搜尋**拿到最高 non-oracle Recℓ(0.788)與 F1(0.602)。
6. **line-level 評估帶來檔案層以外的資訊**:HitFile 有用,但**分不出 explorer 有沒有露出檔案裡對的那幾行**;高 HitFile vs 低 Recℓ 的落差,正是 line-level 設計的價值。

## 5.6 受控 context 退化實驗(§4.4)
問:patcher 更怕**缺相關 context** 還是**多餘無關 context**?把 Oracle context 只露 $\alpha\in\{0,25,50,75,100\}\%$ 核心(缺context),或把移除的預算用隨機非核心區段補滿(冗餘context):
- **缺 context 是主導失敗模式**:resolve rate 呈**門檻式** —— 部分 context 時一直低,在 $\alpha=50\to75$ 之間才跳升 → **要好幾塊核心證據同時在場**才修得對。
- **過門檻後冗餘可容忍**:$\alpha\ge75$ 時冗餘曲線緊貼缺context曲線 → 現代 patcher 能容忍額外無關程式碼,**只要關鍵證據已在**。
- 但**證據稀缺時冗餘最傷**:$\alpha=0$ 時隨機非核心程式碼讓 resolve 掉 **7–9 pp**。
- **caveat**:easier 子集在 $\alpha=0\to25$ 反而下降(可能是空 context 時模型靠記憶/issue-only prior)→ **空 context 基線在經典 repo 上可能被高估**,要小心解讀。

---

# 六、結論 (Conclusion,原文)

SWE-Explore 把倉庫探索從端到端修復中獨立出來,用**排序、行級的 context 選擇**評測。結論三點:① 探索指標確實追蹤下游修復;② **當前 agent 擅長找對檔案,但在行級仍嚴重 recall-limited**;③ **缺核心證據比適度冗餘更傷**。目標是催生「讀得更廣、露出 repair 真正需要的 span」的 explorer。

---

# 七、限制 (Limitations)

- **Ground truth 有路徑偏差**:只代表「被評測 agent 解出來」的路徑,其他有效解法可能沒涵蓋。
- **交集是經驗近似**:純交集保守、union 太吵,refined core 折衷但仍非唯一真值。
- **line-level 標註成本高**:雖大幅自動化,精修 + 人工審核仍需投入。

---

# 八、重點摘要 (Takeaways)

- **把「探索/定位」從二元 resolve 中拆出**,用 coverage / ranking / efficiency 三維度 + line-level ground truth 細看。
- **標準答案 = 多條成功軌跡的行級交集(精修+審核)**,近乎零人工、反映真實修復繞不開的核心。
- **最大發現:現代 explorer 檔案層強、行層弱,瓶頸是 line-level recall(廣度覆蓋);換模型改不了這形狀。**
- **CoSIL 的迭代 code-graph 搜尋**(Recℓ 0.788、下游 59.3 追平 Oracle)是突破方向:結構化、迭代探索勝過一次性讀檔/shell 導航。
- **缺證據 >> 雜訊**:context 工程應**重 recall(找齊)、輕 precision(去雜)**;LLM 對冗餘的容忍度高於對缺失。
- 最預測下游的指標:**Context Efficiency、FUH、Rec@100**。

---

# 九、這篇研究可能的啟發 (Inspirations)

- **評測要拆解中間能力,而非只看端到端成敗**:與 [[Agents' Last Exam (ALE)]] 的「gate-and-score、細粒度可驗證」同精神 —— 二元 pass/fail 掩蓋 agent 到底卡在哪。任何做 agent 評測的人都該問:**能不能把黑箱結果拆成可量測的中間步驟?**
- **「用成功軌跡的交集當 ground truth」是可複用的低成本標註法**:不靠人工,而是**讓多個強 agent 去解、取共同繞不開的部分**。可遷移到任何「難定義標準答案、但可驗證成敗」的任務。
- **line-level recall 是 coding agent 的真瓶頸**:改進方向不在「更會寫補丁」,而在**更廣、更結構化地探索 repo**(CoSIL 的 code-graph)。呼應 [[Zep - Temporal KG for Agent Memory|Zep]]:**沿結構(圖)探索 > 純向量/一次性檢索**,兩篇都指向「結構化探索」勝扁平檢索。
- **「缺證據 >> 雜訊」對 context 工程的啟示**:與其壓 precision,不如確保 recall;LLM 對冗餘的容忍度比對「缺關鍵資訊」高得多。也連到 [[Latent Context Language Models (LCLM)|LCLM]] 的「壓縮 + 按需展開」—— 要點都是「確保關鍵證據在場」。
- **開放問題**:① 固定行數預算下同時拉高 recall(廣覆蓋)與 ranking(排前面);② code-graph / 迭代搜尋為何贏,能否蒸餾進通用 agent;③ ground truth 路徑偏差如何緩解(涵蓋更多元解法)。
