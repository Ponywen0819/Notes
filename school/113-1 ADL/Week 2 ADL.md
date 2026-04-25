#note

# Sequence Modeling
## Meaning Representations in Computers
 兩種方法
 - Knowledge base
 - Corpus base

### Knowledge base
透過語言專家進行字詞間關係的定義。

透過字詞間不同關係的連結，會形成一個網路，稱為 `Wordnet`

**缺點**：
- 沒辦法處理不在網路中的詞彙
- 字詞間的關係受到專家主觀意見影響
- 需要很多人力
- 較難以計算字詞間相關性

### Corpus base
使用 `one-hot` 編碼，使用向量的方式表示。

每一個維度都代表一個特定的字詞，這也隱含表示每個維度鼻此之間沒有關聯，造成無法進行相似度計算。

而 **Neighbor base** 就是為了解決該問題的改良，在建構空間時，每個維度會使用 **附近的字詞** 來建構，這個模式的假設相似的字詞，周圍出現的字詞也會相似。


而 **附近的字詞** 所包含的範圍有兩種模式
- Window
- Full doc

在 Window 中會根據該字詞在文本中的位置前後 $n$ 個字詞作為鄰居，而 Full doc 則使用整個文本的字詞作為鄰居。

> [!NOTE] Full doc 我記得是用於表示文本內容的方法
> 
> 


**Window Neighbor base** 的方法會讓字詞向量是一個**稀疏向量**，稀疏向量在資訊密度上是比較不好的，因此使用幾種方法用於降維。

而其中一種方法稱為 Singular Value Decomposition，而這種方法的時間複雜度為 $O(mn^2)$ ，造增加詞彙的成本較大。

#### Word embedding 
為了解決稀疏字詞向量的問題，直接使用文本訓練，產出固定維度的空間，字詞間可以相互計算，因此可以使用更自由的數學計算。


## Language Modeling
目標為將字詞序列做機率建模。

較為傳統的方法稱為 **n-gram**，該方法就是將字詞前的 $n-1$ 個字詞後出現該字詞的機率進行建模，會受到訓練資料的影響，會錯誤判斷陌生序列的機率。


### Neural Language Modeling
依然使用  **n-gram** 的概念，但使用 nn 的方式進行建模，藉由 word-embedding 所展現的字詞相關性使陌生序列仍舊可以計算。

### Recurrent Neural Network
由於先前的語言模型的 window 都是固定的，沒辦法很好的利用序列的相關。

使用 RNN 進行字詞序列的訓練，整個序列的 context 都可以被使用。

## Recurrent Neural Network
### Back-propagation through Time 
peko

### Training issue
容易有梯度消失或爆炸的問題。

為了解決這個問題提出了兩種解法
- Clipping
- Gating

在 Clipping 中，主要是在處理梯度爆炸的問題，在此模式中，若梯度超過某個閾值，會將該梯度進行等比縮小，讓梯度縮小。

而 Gating 則是處理梯度消失的問題，主要的概念就是讓輸入直接通過，改善梯度下降得情形。





## RNN 應用



