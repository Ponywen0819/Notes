#paper

[Zep: A Temporal Knowledge Graph Architecture for Agent Memory (Rasmussen et al., 2025)](https://arxiv.org/abs/2501.13956)

> arXiv:2501.13956 ｜ 2025-01-20 ｜ cs.CL / cs.AI / cs.IR
> 作者:Preston Rasmussen、Pavlo Paliychuk、Travis Beauvais、Jack Ryan、Daniel Chalef(Zep AI)
> 核心引擎:**Graphiti**(temporally-aware KG engine)

---

# 一、問題與背景

## 1.1 為什麼 agent 需要「記憶」
LLM 本身是無狀態的:每次呼叫只看得到當下 context window 裡的內容。要讓 agent 跨多輪、跨多 session 記住使用者說過什麼、商業資料如何變化,就需要一個外掛的**記憶層 (memory layer)**,在每次查詢時把「相關的過去」撈回來塞進 prompt。

## 1.2 靜態 RAG 的侷限
目前主流做法是 **RAG**:把文件切塊 (chunk)、嵌入成向量、存進向量庫,查詢時用相似度撈回 top-k。問題在於它預設知識是**靜態文件**:
- **無法處理「事實會變」**:使用者去年說喜歡咖啡、今年改喝茶,向量庫裡兩塊都在,檢索時無從判斷哪個是現況。
- **無法融合結構化資料**:企業場景同時有「對話」(非結構化)與「業務資料」(結構化,如 CRM、訂單),純 chunk 檢索難以把兩者關聯。
- **缺乏關係**:chunk 之間彼此孤立,撈到一塊就只有那一塊,沒有「順著關係再往外看」的能力。

## 1.3 Zep 的定位
Zep 是一個給 AI agent 的**記憶層服務**,核心是 **Graphiti** —— 一個**帶時間感知的知識圖譜引擎**。它把持續進來的對話與資料,**即時**合成進一張會演化的知識圖,並保留「事實隨時間成立/失效」的完整歷史。

> 與 [[Temporal Knowledge Graph Survey|TKG]] 的關係:學術界的 TKG 研究多在「學嵌入、做 link prediction」;Zep 是把 TKG 的**概念(四元組、時間有效區間)產品化**,當成 agent 的可查詢記憶。**一句話定位:把 agent 記憶從「靜態 RAG」升級成「會更新、能處理矛盾的時序知識圖」。**

---

# 二、Graphiti 的三層階層圖

整張圖記為 $\mathcal{G}=(\mathcal{N},\mathcal{E},\phi)$($\phi$ 是關聯函數,定義邊接到哪些節點)。設計上分三個子圖,直接對應**認知心理學的兩種記憶**:情節記憶 (episodic,記得「發生過什麼事」) 與語意記憶 (semantic,記得「概念之間的關聯」)。

## 2.1 Episode 子圖 $\mathcal{G}_e$ —— 原始、無損
- **episodic node**:存放原始輸入,可以是一則對話訊息、一段文件、或一筆 JSON 業務資料。
- **關鍵特性:無損 (non-lossy)**。原文完整保留,所有語意抽取都從這裡出發。
- **episodic edge**:把一則 episode 連到它「提到」的語意實體 —— 等於記錄「這個事實/實體**出處**是哪則訊息」,可回溯、可重抽。

> 為什麼要無損保留:抽取難免遺漏或出錯;留著原文,日後可以用更好的模型重抽,也可在檢索時 fallback 回原始細節。

## 2.2 Semantic Entity 子圖 $\mathcal{G}_s$ —— 抽取後的語意層
- **entity node**:從 episode 抽出、並**去重 (dedup)** 後的實體(人、公司、地點、概念…)。每個實體有名稱與一段摘要。
- **semantic edge**:實體之間的關係,也就是一條 **fact**(例:`Alice —works_at→ Acme`)。
- 這層是「結構化知識」的所在,大多數檢索與推理在這層進行。

## 2.3 Community 子圖 $\mathcal{G}_c$ —— 高層摘要
- **community node**:把**強連通的實體群集**起來的高層節點,附一段該群的摘要。
- **community edge**:連到群內成員實體。
- 用途:提供「鳥瞰式」的主題視角,讓檢索能在更高抽象層取用(例如「關於這位使用者的整體偏好」)。

> **三層的整體直覺**:最底層保住原文(可回溯)、中層是結構化語意(可推理)、上層是主題摘要(可鳥瞰)。檢索時可依問題在不同抽象層取材。

---

# 三、雙時間模型 (Bi-Temporal Model)——全篇最關鍵

這是 Zep 區別於一般 memory 的核心。每條邊(fact)同時掛在**兩條獨立的時間軸**上,共四個時間戳。

## 3.1 兩條時間軸
- **$T$ — valid time(事件在現實中何時為真)**
  - $t_{valid}$:事實開始成立的時間。
  - $t_{invalid}$:事實不再成立的時間。
- **$T'$ — transaction time(資料在系統中何時被記錄)**
  - $t'_{created}$:這條邊何時寫進系統。
  - $t'_{expired}$:這條邊何時在系統中被標記失效。

## 3.2 為什麼要兩條軸
兩條軸回答**兩種根本不同的問題**:
- valid time → 「**這件事什麼時候是真的?**」(現實世界的時間)
- transaction time → 「**系統什麼時候知道這件事的?**」(資料庫的時間)

這讓 Zep 能正確處理「**回溯更新**」:例如使用者今天才告訴你「我其實上個月就換工作了」。事件的 valid time 是上個月,但 transaction time 是今天 —— 兩條軸分開記,才不會把「何時發生」和「何時得知」混為一談。

## 3.3 邊失效機制 (Edge Invalidation)
當新事實與舊事實**在時間上重疊且互相矛盾**時:
1. 系統用 **LLM 比對新邊 vs 語意相關的既有邊**,判斷是否矛盾。
2. 若矛盾,把**舊邊的 $t_{invalid}$ 設成「使它失效的新邊的 $t_{valid}$」**。
3. 原則:**一律以新資訊為準**(prioritize new information)。
4. **舊邊不刪除**,只標記失效 → 歷史完整保留、可回溯。

## 3.4 一個具體例子
> - 2023-01,對話提到「Alice 在 Acme 上班」→ 建邊 `Alice —works_at→ Acme`,$t_{valid}=$2023-01,$t'_{created}=$2023-01。
> - 2024-03,對話提到「Alice 上個月跳槽到 Globex」→ 建新邊 `Alice —works_at→ Globex`,$t_{valid}=$2024-02。
> - LLM 偵測到兩條 `works_at` 在時間上重疊且矛盾 → 把舊邊 `→Acme` 的 $t_{invalid}$ 設為 2024-02,$t'_{expired}=$2024-03。
> - 結果:查「Alice 現在在哪上班」→ Globex;查「Alice 2023 年在哪」→ 仍可從未刪除的舊邊得到 Acme。

> **這就是處理「知識會過期」的乾淨做法**:不是覆蓋、不是 append 兩份讓檢索去猜,而是**讓舊事實在正確的時間點優雅失效、且保留歷史**。

---

# 四、建圖流水線 (Construction Pipeline)

每當有新 episode 進來,Graphiti 跑一條 LLM 驅動的流水線把它融進圖。

## 4.1 實體抽取 (Entity Extraction)
- 餵給 LLM 的是**當前訊息 + 前 $n=4$ 則訊息**當上下文(避免指代消解失敗)。
- **說話者自動被當成一個實體**。
- 用 **reflection**(讓模型再檢查一次自己的抽取)來壓低幻覺。

## 4.2 實體解析與去重 (Entity Resolution)
新抽出的實體要判斷「是不是已經存在的某實體」:
1. 把實體**名稱嵌入成 1024 維向量**,用語意相似找候選。
2. **另外**對既有實體的名稱與摘要做**全文檢索**,補上語意相似抓不到的候選(例如縮寫、別名)。
3. 把候選交給 LLM 用 entity-resolution prompt 判斷是否同一實體。
4. 判為重複 → 合併,更新一個整合後的名稱與摘要。

> 雙管齊下(語意嵌入 + 全文)是重點:embedding 擅長近義、全文擅長字面,互補才抓得全。

## 4.3 Fact / 邊抽取 (Edge Extraction)
- 抽出連接實體的謂詞 (predicate),形成 semantic edge。
- 同一 fact 可在不同實體對之間多次抽取 → 透過 **hyper-edge** 建模「多實體事實」(超過兩個實體共同參與的事實)。

## 4.4 邊去重 (Edge Deduplication)
- 同樣用混合檢索(語意 + 全文)找候選邊。
- **關鍵約束:只在「相同實體對」之間比對**。這既降低計算複雜度,又避免把不同實體對的邊誤判成重複。

## 4.5 寫入 (Schema / Write)
- 用**預先定義好的 Cypher query** 把資料插進 Neo4j。
- **刻意不讓 LLM 自己生 DB query** → 保證 schema 格式一致、減少幻覺帶來的寫入錯誤。

---

# 五、社群偵測與摘要 (Community)

## 5.1 演算法選擇
- 用 **label propagation**,而非 Leiden —— 為了效率(可增量更新)。

## 5.2 動態更新 vs 全量刷新
- **動態版**:當新實體加入,只做 label propagation 的**單一遞迴步** —— 看鄰居各屬哪些社群,把新節點歸到**佔多數**的那個社群。
- **取捨**:動態更新省延遲與成本,但社群會**逐漸偏離**「完整重跑一次」的結果 → 因此仍需**定期全量刷新**校正。

## 5.3 社群摘要
- 對成員節點做 **map-reduce 式的迭代摘要**,生成該社群的高層描述。
- 社群名(含關鍵詞與主題)也被嵌入,供 cosine 相似度檢索取用。

---

# 六、檢索機制 (Retrieval)

檢索被形式化為三段組合函數:
$$f(\alpha) = \chi\bigl(\rho\bigl(\varphi(\alpha)\bigr)\bigr) = \beta$$
query $\alpha$ → 搜尋 $\varphi$ → 重排 $\rho$ → 構造 $\chi$ → 格式化好的 context $\beta$。

## 6.1 搜尋 $\varphi$(三路並行)
- **$\varphi_{cos}$ — cosine 語意相似**:對實體名、fact 文字、社群名做 embedding 檢索。
- **$\varphi_{bm25}$ — Okapi BM25 全文**:走 Neo4j Lucene,抓字面匹配。
- **$\varphi_{bfs}$ — 廣度優先搜尋**:從初始命中的節點/邊出發,做 **n-hop 圖遍歷**,帶出圖上鄰近的節點與邊。

> **$\varphi_{bfs}$ 是 KG 記憶相對純向量 RAG 的最大優勢**:純語意相似會漏掉「用詞不同、但關係上高度相關」的脈絡;沿著圖結構往外走 n 跳,能補回這些 embedding 抓不到的關聯(圖上越近,往往對話脈絡也越相關)。

## 6.2 重排 $\rho$
支援多種 reranker,可組合:
- **RRF**(Reciprocal Rank Fusion):融合多路檢索的排名。
- **MMR**(Maximal Marginal Relevance):兼顧相關性與多樣性、去冗餘。
- **episode 提及頻率**:常被提到的越重要。
- **節點距離**:離 query 焦點越近越優先。
- **cross-encoder LLM 評分**:最精準但最貴。

## 6.3 構造 $\chi$
把選中的內容組成 context 字串:
- **邊**:fact 文字 + 其 valid/invalid 時間(讓 LLM 知道這事實的時效)。
- **實體節點**:名稱 + 摘要。
- **社群節點**:摘要。

---

# 七、實驗

## 7.1 DMR(Deep Memory Retrieval)
- **設定**:500 段多輪對話(每段 ~60 則),單輪事實檢索題。

| 記憶 | 模型 | 準確率 |
| --- | --- | --- |
| MemGPT | gpt-4-turbo | 93.4% |
| **Zep** | gpt-4-turbo | **94.8%** |
| **Zep** | gpt-4o-mini | **98.2%** |

- **作者自評 DMR 不夠難**:只有單輪事實題、措辭模糊、對話整段塞得進 context window,鑑別度低。所以改用更貼近企業的 LongMemEval。

## 7.2 LongMemEval(LME)
- **設定**:對話平均 **~115k tokens**,六種題型(單 session user/assistant/preference、多 session、知識更新、時序推理)。

| 記憶 | 模型 | 準確率 | 中位延遲 | context tokens |
| --- | --- | --- | --- | --- |
| Full-context | gpt-4o | 60.2% | 28.9s | 115k |
| **Zep** | gpt-4o | **71.2%** | **2.58s** | **1.6k** |
| Full-context | gpt-4o-mini | 55.4% | 31.3s | 115k |
| **Zep** | gpt-4o-mini | **63.8%** | **3.20s** | **1.6k** |

- **整體**:gpt-4o **+18.5% 準確率、延遲降 ~91%**;gpt-4o-mini **+15.2%、延遲降 ~90%**。context 從 115k → **1.6k**(約 1.4%)。

## 7.3 分題型分析(gpt-4o)
- **單 session 偏好:+184%**(20.0% → 56.7%)—— 提升最大。
- **時序推理:+38.4%**(45.1% → 62.4%)—— 雙時間模型直接受益。
- **多 session:+30.7%**(44.3% → 57.9%)。
- **知識更新:+6.5%**(78.2% → 83.3%)。
- **single-session-assistant:−17.7%**(94.6% → 80.4%)← **唯一退步、值得注意**。

> 主訊息:**用對話量約 1.4% 的 token,換到更高準確率 + 快一個數量級的延遲**。原因是昂貴的「理解」在**建圖時離線做完**,查詢時只取結構化結果。能力越強的模型(gpt-4o > gpt-4o-mini)在複雜題型上受益越大。

---

# 八、限制

- **抽成圖會遺失原文細節**:single-session-assistant 退步 17.7%,因為這類題目直接讀原文反而好 —— 圖抽取是有損的。
- **DMR benchmark 太簡單**:規模小、題目模糊,不反映企業場景;MemGPT 在 LME 上甚至**跑不出有效回答**。
- **缺對應 benchmark**:沒有評測「對話史 + 結構化商業資料融合」的標準集,而這正是 Zep 主打的場景。
- **社群動態更新會漂移**:需定期全量刷新校正。
- **建圖成本**:抽取/解析/失效判斷都是 LLM 呼叫,大規模下的成本與延遲,正文著墨有限。

---

# 九、重點摘要 (Takeaways)

- **agent 記憶 = 時序知識圖**,而非靜態 RAG:Graphiti 即時把對話/資料建成會演化的圖。
- **雙時間模型**(valid time $T$ + transaction time $T'$ + 邊失效)是核心 —— 能處理**知識更新與矛盾**,且可回溯。
- **三層階層**(episode 無損 / entity 語意 / community 摘要)對應情節 vs 語意記憶,支援多抽象層檢索。
- **混合檢索**(cosine + BM25 + 圖 BFS)+ 多種重排;$\varphi_{bfs}$ 是 KG 相對向量 RAG 的關鍵優勢。
- **效益**:LME 上 +18.5% 準確率、延遲降 ~91%、context 省到 1.4%。
- **取捨**:抽成圖會丟原文細節;建圖的 LLM 成本是隱性代價。

---

# 十、這篇研究可能的啟發 (Inspirations)

- **「離線建圖、線上輕查詢」是長記憶的好範式**:把昂貴的理解(抽取/消歧/矛盾判斷)**前置到 ingestion**,查詢時只取結構化結果 → 同時拿到準確率與低延遲。與 [[Latent Context Language Models (LCLM)]] 的「壓縮 + 按需展開」是同精神的兩條路:**都在避免每次查詢都重讀全文**。
- **雙時間 + 邊失效 是處理「知識會過期」的乾淨機制**:多數 agent memory 只 append、不會讓舊事實優雅失效。valid/transaction 兩軸 + 不刪只標 invalid,既支援時序推理又可回溯,值得任何做 memory 的人借鏡。對照 [[Temporal Knowledge Graph Survey|TKG survey]]:Zep 把學術 TKG 概念**產品化**成 agent 記憶。
- **圖結構 BFS 補向量檢索的盲點**:純語意相似會漏掉「關係上相關但用詞不同」的脈絡;沿圖 n-hop 擴展能補回 —— 提示**記憶檢索不該只靠 embedding**。
- **開放問題**:① 抽成圖必然遺失原文細節(single-session-assistant 退步)→ 何時保留原文、何時抽象,是個 routing 問題;② 建圖的 LLM 成本/延遲在大規模下的可擴展性;③ 缺「對話 + 商業結構化資料融合」的 benchmark,是後續該補的評測缺口。
