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
假定今有 $n$ 個類別 $C= \{c_1,c_2,...,c_n\}$ ，每個類別都有先驗機率 $P(c_n)$ 。

接著建構所有類別的 likelihood 
$$p(X|c_i)$$
此方程式用於表示在特定類別中樣本的機率分布。

接著使用 likelihood 建構後驗機率
$$p(C|x) = \frac{p(x|C)P(C)}{p(x)}$$
接著使用 MAP 找出使後驗機率最大化之類別
$$\hat c_{\text{map}} = \text{arg max}_{i = \{1,2,...i\}} \{p(c_i|x)\}$$

$\hat c_{\text{map}}$ 就為最後的答案

> 以上的例子並沒有結合損失函式

若定義損失函式為 $L(c_m|c_n)$ ，代表將 $c_m$ 分類成˙ $c_n$ 的損失。利用此損失函式結合後驗機率就可以得到
$$R(d|x) = \sum_CL(d|c)p(c|x)$$
最大化損失函式後就可以得到考慮損失後的分類結果。

