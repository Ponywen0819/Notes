#paper

[CRAFTER: A Multi-Agent Harness for Editable Scientific Figure Generation from Diverse Inputs](https://arxiv.org/abs/2605.30611)

> arXiv:2605.30611v1 ｜ 2026-05-28 ｜ cs.CV / cs.AI / cs.CL
> 作者:Haozhe Zhao, Shuzheng Si, Zhenhailong Wang 等(UIUC、清華、北大)
> 程式碼:https://github.com/HaozheZhao/Crafter

---

# 一、問題 (Problem)

科學圖表是傳達研究的關鍵媒介,但畫「可發表品質」的圖極耗工。現有自動化系統有兩個根本缺陷:

1. **範圍太窄**:每個系統只針對「**一種圖型**」且只吃「**純文字**」輸入。但研究者實際會做學術圖、海報、資訊圖,而且常從草稿、部分版面、參考元素開始迭代,而非從零文字生成。
2. **輸出不可編輯**:raster(點陣圖)無法局部修改 —— 想改某個標籤、換配色、重排元件都做不到。Code-generation(TikZ)雖可編輯卻缺乏 icon 與風格化版面的視覺豐富度;近期 raster→vector 嘗試則受限於元素抽取不可靠、組合脆弱。

**核心洞察**:科學圖表 ≠ 自然影像,而是「**離散語義元件的結構化組合**」(標籤框、箭頭、icon、註解,各有精確空間關係)。生成器在這種版面上產生的是**局部錯誤**(亂碼標籤、錯位連接線),而且:

- **單純重試無效** —— 每次失敗的樣態都不同。
- **累積自由文字修正會自我矛盾** —— 「放大標題」後又「減少留白」彼此衝突,品質悄悄退化。

→ 需要的不是更強的 backbone,而是一層 **harness(協調外殼)**:用「演進中的結構化規格」當記憶,做**針對性修正**與**閉環驗證**。

---

# 二、方法 (Method)

## Harness 抽象:四角色迴圈

把 harness 形式化成一個共享「**演進規格 S**(evolving specification,記錄當前計畫、修訂史、診斷)」的四角色迴圈。每一輪 t:

| 角色 | 符號 | 做什麼 |
| --- | --- | --- |
| **Designer** | D | 依 input 與 S 產出可執行計畫 `pₜ` |
| **Executor** | E | 把計畫渲染成產物 `aₜ`(**可插拔** —— 圖像生成器 or 程式碼生成器) |
| **Verifier** | V | 發出**指令式診斷 `dₜ`**(逐維度分數 + 指認缺陷 + 修正建議,而非單一純量分數) |
| **Reviser** | R | 把診斷轉成 **typed edits**(加版面約束、禁某類元素、調整某元件大小)寫回 S |

兩個關鍵性質:**E 可插拔**(任務特定行為全在 D/V/R 的 prompt 裡)、**R 寫結構化 typed edits 而非堆疊自由文字** → 規格跨輪保持內部一致。論文用同一 pattern 實例化成兩套系統。

## CRAFTER —— 圖表生成 harness(5 個協作代理)

- **Intent Reasoner**:分析輸入,推斷圖的溝通角色與所需視覺元素,種下初始規格 S₀。
- **Plan Generator (D)** + **Image-Gen Backend (E)** + **Critic (V)** + **Specification Refiner (R)** + **Convergence Judge**(每輪決定 accept / refine / revert)。

三個機制對應三個失效模式:

1. **多樣性驅動的計畫探索 (Diversity-Driven Plan Exploration)**:D 產生 **K 個**不同視覺框架的候選計畫,E **並行**渲染,選最佳當起點。可在花費精修預算前就逃離「根本不適合的構圖選擇」(如把對比網格畫成方塊圖)。K 依輸入自適應。
2. **結構化修正層 (Structured Corrective Layer)**:用 typed edits 取代自由文字累積,避免矛盾悄悄堆積。
3. **Verify-then-Refine + 指令式 Critic**:Critic 沿 6 個軸給逐維度診斷而非純量分;有 **early-exit gate**(首輪夠好就跳過)與 **best-so-far checkpoint**(退步就回退到 a*),最多 T=3 輪 —— 因為 LLM 驅動的迭代編輯**非單調**。

## CRAFTEDITOR —— raster → 可編輯 SVG(同一 harness pattern,三階段)

1. **Extraction**:VLM 規劃 keep/delete,剝掉文字疊層與雜訊,得到乾淨的圖形資產。
2. **Processing**:每個資產加 caption、grounding,分類為 vector 或 raster。
3. **Composition**:D 生成兩個不同溫度的 SVG 骨架 → 選優 → E 注入資產 → **hybrid critic**(VLM 看全域版面 + 程式化檢查器查文字溢出/箭頭端點/元素重疊/缺漏元件)迭代精修,最多 T=4 輪。

---

# 三、CRAFTBENCH(新基準)

- **279 樣本**,涵蓋 **3 種圖型 × 4 種輸入條件**,首個測「跨圖型、跨條件泛化」的基準。
- **4 種任務**:Text-to-Image(179,64.2%)、Mask-Completion(30)、Sketch-conditioned(40)、Key-Element composition(30)。
- **3 種風格**:Academic(140)、Poster(109)、Infographic(30)。
- **資料來源**:18 領域 arXiv 學術圖、頂會獲獎海報、長文研究部落格資訊圖;經三階段(收集 → 過濾 → 標註),553 候選由人工縮到 279,reference 條件樣本需 3 位研究生標註員**一致同意**。
- **評測協定**:Gemini 3.5 Flash 當 judge,候選與目標**各自獨立**打分(消除 pairwise 的位置偏差),逐維度 0–10,加權後對 human-drawn target 算 lenient win-rate;在學術 T2I 上退化為 PaperBanana 式評分。

---

# 四、結果 (Results)

主結果(Table 2,所有 agentic 方法共用同一 backbone **Nano Banana 2** 與 VLM,以隔離編排設計的效果):

| 方法 | PaperBanana-Bench Overall | CraftBench Overall |
| --- | --- | --- |
| Nano Banana 2(standalone) | 11.13 | 19.90 |
| PaperBanana(最強 agentic baseline) | 33.73 | 28.00 |
| **CRAFTER (w/ Nano Banana 2)** | **50.34** | **50.20** |

- CRAFTER 在兩個基準都拿最高總分,**領先最強 agentic baseline 16.61 分(PaperBanana-Bench)/ 22.20 分(CraftBench)**。
- **唯一**在每個維度、每個任務都**全面超越自身 backbone** 的方法。對比之下 PaperBanana 在更廣的基準上增益從 22.60 縮到 8.10,且在 sketch 任務**反而輸給 backbone** —— 正是論文指出的「泛化失敗」。
- **執行器無關**:把 Nano Banana 2 換成 Pro,總分只變動 0.34 → harness 的貢獻不靠更強生成器。
- **消融(PaperBanana-Bench)**:移除任一機制掉 **5.04~8.90 分**。corrective layer(−8.90)與 plan exploration(−8.56)最關鍵。
- **CRAFTEDITOR**:對 80 個 CRAFTER 輸出、3 個 VLU judge 集成評分,**總分 8.04** vs. AutoFigure-Edit 6.91 vs. Edit-Banana 3.69,七軸全勝;在 text / arrow 等結構軸領先最多。移除 iterative composition 掉 2.15 分。

---

# 五、重點摘要 (Takeaways)

- **核心主張**:結構化生成的瓶頸是「局部錯誤的修正」,解法是 **harness(編排層)** 而非更大的模型 —— 「not a better generator but a harness」。
- **關鍵設計**:① 共享演進規格當記憶;② **typed edits** 取代自由文字(避免矛盾累積);③ **指令式診斷** 取代純量分(給可行動的修正目標);④ **計畫層分支** 在花預算前逃離壞構圖;⑤ best-so-far 回退對抗非單調迭代。
- **泛化來自 prompt-level adaptation**:架構不變,任務特定行為全在 prompt → 同一 pipeline 橫跨圖型與輸入條件。
- **首個 generation-to-editing 端到端流程**:CRAFTER(生成)+ CRAFTEDITOR(轉可編輯 SVG)。
- **可遷移性**:harness 是 executor-agnostic,未來更強生成器可直接換入;作者預期此 pattern 能擴展到科學圖以外的結構化輸出領域。

---

# 六、可借鏡之處 / 與我相關

- 「harness over backbone」對任何**結構化輸出 + 局部錯誤**的生成任務都適用(程式碼、表格、版面)。
- **typed edits vs free-text**:迭代修正時用結構化操作維持規格一致性,是對抗「prompt 累積矛盾」的通用招數,值得用在自己的 agent pipeline。
- **指令式 critic(逐維度診斷 + 修正建議)** 比單一分數更能驅動有效迭代。
- VLM-as-judge 用「**各自獨立打分**」消除 pairwise 位置偏差 —— 評測設計的細節。
