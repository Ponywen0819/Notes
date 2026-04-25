#paper

[Comparison of a simple method for quantitation of intraepidermal nerve fibres with a standard image analysis method using hypothenar skin (Wilder-Smith & Chow, J Neurol 2006)](https://link.springer.com/article/10.1007/s00415-006-0147-6)

publish year: 2006

# 概述
比較兩種 IENFD（intraepidermal nerve fibre density）的計算方法，分別為**顯微鏡內尺（intraocular ruler）簡易測量法**與**電腦影像分析法**，結果顯示兩者測量值高度一致，證實簡易測量法在臨床與資源有限環境下的可行性。

研究動機：標準化的 IENFD 計算流程仰賴電腦影像分析軟體（如 Bioquant、ImageJ 等），需要相機、電腦與訓練後的操作者，對許多臨床診斷單位而言取得成本高。若能驗證一個只需顯微鏡與目鏡內尺的簡易方法，將可大幅降低 IENFD 量測的門檻。

# 受試者
- 40 位健康受試者（mean age 41.1 歲，range 21–71）。
- 族群分布：24 位華人、11 位印度人、5 位馬來人。
- 性別：女性 30 位。
- 切片來源：**手掌小魚際肌（hypothenar）** 區皮膚。
- 染色方式：以 panaxonal 抗體 **PGP 9.5** 進行免疫過氧化酶（immunoperoxidase）染色。

選擇 hypothenar 而非常用的小腿外側（distal leg），是因為手部位置較不易受周邊神經病變早期影響，可作為健康族群的標準對照。

# 方法
兩種方法皆計算同一張切片中**穿越真皮–表皮交界（DEJ）的神經纖維數量**，差別只在於「表皮長度」如何測得。最終 IENFD 統一以
$$
\text{IENFD} = \frac{\text{nerve fibres count}}{\text{epidermal length (mm)}}
$$
表示，單位為 fibres/mm。

## 顯微鏡內尺測量法（簡易法）
在相同切片與 40× 放大下，將目鏡內尺刻度對齊切片上「角質層上緣」與「真皮–表皮交界」兩端。

直接讀取刻度差值，獲得表皮總長度（mm）。

特點：
- 只需要一支具備內尺刻度的目鏡，不需電腦或影像擷取設備。
- 表皮長度以**直線距離**近似，忽略表皮輪廓的彎曲。
- 單張切片量測時間 < 1 分鐘。

## 電腦影像分析法（gold standard）
在軟體中，沿「角質層上緣」（stratum corneum upper margin）以滑鼠手動或利用半自動邊緣偵測工具描繪表皮邊界路徑，由軟體計算路徑的弧長作為表皮長度。

特點：
- 量測的是表皮輪廓的**真實曲線長度**，理論上比直線測量更精確。
- 需要影像擷取設備、影像分析軟體與訓練後的操作者。
- 單張切片量測時間明顯較長（取決於切片複雜度與軟體流暢度）。

## 神經纖維計數規則
雙方皆採用 EFNS / Peripheral Nerve Society 的計數準則（與 [[Intraepidermal_Nerve_Fiber_Analysisin_Human Patients_and_AnimalModels_of_Peripheral Neuropathy_A_Comparative_Review|IENF Comparative Review]] 中一致）：
- 跨越 DEJ 的纖維才計數。
- 在 DEJ 處或以下分支者，每一分支分別計數。
- 接觸但未跨膜者不計。
- 純粹位於表皮內、無真皮連接的孤立軸突不計。

# 結果
| 方法 | IENFD (fibres/mm) | 2SD |
| --- | --- | --- |
| Intraocular ruler | **3.07** | 1.56 |
| Image software analysis | **3.05** | 1.54 |

兩法平均值差異 < 1%，標準差幾乎相同。論文以 Bland–Altman 分析及相關係數驗證，兩法之間具高度一致性（high agreement），無系統性偏差。

# 討論
- 簡易法之所以可行，原因之一是**表皮輪廓相對平直**：在 hypothenar 區的切片中，DEJ 雖有 rete ridge 起伏，但在 40× 視野下，直線近似的弧長誤差通常 < 5%，且兩種方法在計算 IENFD 時，分子（神經纖維數）並不受測量方式影響，僅有分母（表皮長度）的微小差異。
- 對 distal leg、足底等表皮較厚或 rete ridge 較深的部位，直線近似的誤差是否同樣可接受，論文未驗證；後續使用此法時應留意切片來源。
- 由於免去電腦軟體與影像擷取流程，簡易法在資源有限的醫療單位（如熱帶地區、社區診所）特別有意義，同時亦可作為大規模流行病學研究的快速篩檢工具。

# 對本研究的啟發
- 若要建立小型的 IENFD 量測流程，可優先嘗試此簡易法，不必一開始就投資完整的影像分析設備。
- 若研究目標部位為足底、小腿等 rete ridge 明顯的位置，需另外驗證簡易法的誤差。
- 提供了一組 hypothenar 區的健康人 IENFD 參考值（約 3 fibres/mm），可作為比對對照組的依據。

# 參考族群數值
- Hypothenar IENFD（健康人）：**約 3 fibres/mm**（2SD ≈ 1.5）。
- 此值低於常見的 distal leg 參考值（多為 7–13 fibres/mm，依年齡與性別不同），原因是手掌部位本身的神經纖維密度較低；不可與 distal leg 的參考值直接互換比較。

