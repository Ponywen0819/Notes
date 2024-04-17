# 介紹
---
貝式推測理論 ( Bayesian Decision Theory) 是一種使用機率與成本決定最佳決策的理論，它包含以下不同部分：
- 先驗機率 ( Prior Probability )
- [概似函式 ( likelihood function )](Likelihood.md)
- 後驗機率 ( Posterior Probability )
- 損失函式 ( Loss Function )
- 風險函式 ( Risk Function )

以蒐集到的資料建立概似函式，並將其與先驗機率結合得出後驗機率。

再根據不同的決策結果建立損失函式，並使用其函式與後驗機率結合得出風險函式。

根據風險函式的結果決定最佳的決策。


# 範例
---
假定今有 $n$ 個類別 $\{c_1,c_2,...,c_n\}$ ，每個類別都有先驗機率 $P(c_n)$ 。


