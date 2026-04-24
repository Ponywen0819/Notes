
# Background
Knowledge Distillation (KD) emerged as a practical solution to the ever-growing size and computational demands of deep neural networks. By transferring “dark knowledge” from a large, high-capacity **Teacher** model to a smaller **Student**model, KD enables deployment of lightweight yet performant models in resource-constrained environments.

# What's Knowledge Distillation
## 主要目標
由於大型語言模型難以取用與高成本的特性，發展出在不犧牲大多數效能的狀況下，大幅縮減模型大小的技術，並且在模型尺寸大幅縮減後，推論速度與運算成本也會隨尺寸縮小。
## 教師與學生架構
為知識蒸餾中的核心架構，有兩個主要角色
- 教師模型
	尺寸大，在目標任務可以達到最佳效能的模型
- 學生模型
	尺寸小、結構簡單，透過模仿教師模型提升效能
## 如何進行學習
LLM 模型，可以看作從一個機率空間經過複雜的運算後轉換到另一個機率空間，而不同的模型所輸出的機率空間也不盡相同，為了確認不同模型機率空間的差別，會使用 KL 散度 ( KL Divergence ) 量化空間的距離，而這個距離就可以用來訓練學生模型。

# What’s Knowledge Distillation
## Teacher–Student Architecture
The core framework of knowledge distillation involves two primary roles:
- **Teacher Model**  
  A large-scale model that achieves optimal performance on the target task.
- **Student Model**  
  A smaller, simpler model that improves its performance by imitating the teacher model.

# What's Reasoning Distillation
Reasoning Distillation is a specialized form of Knowledge Distillation that focuses on transferring not only the final outputs of a large “Teacher” model but also its multi-step reasoning process (Chain-of-Thought) to a smaller “Student” model.


### What's Chain of Thought


# Relate wrok
在文章中提到的三個模型，在訓練過程中都提及這篇論文或是與這篇論文的想法類似。
Imitate, Explore, and Self-Improve: A Reproduction Report on Slow-thinking Reasoning Systems
![[Pasted image 20250504110013.png]]

可以分成幾個技術重點，分別為
- 資料收集
- 格式統一
- 提示模板

##### 資料收集
研究人員選擇使用現有的推理模型 ( 在研究中稱為 o1-like ) 進行資料的收集。

![[Pasted image 20250504110845.png]]

由不同的 domain 資料集中收集問題與解答，利用推理模型取得思考鏈與最後解答。

為了增加資料的品質，研究者也加入多種方式篩選 ( Reject sampling )
- 錯誤答案
- 混合語言推論
- 重複推論片段
- 不知所云內容

為了讓模型可以擁有更為泛化的推論能力，篩選較為困難的問題於資料集中。

因為研究者認為在較為簡單的任務上，推論的內容對訓練也沒有太多益處。


##### 格式統一

![[Pasted image 20250504111319.png]]

研究者在也觀察到不同的模型，推論與解答的格式各有不同，如圖中的 QwQ 與 R1 的輸出格式，這會造成難以提取推論與解答部分。

研究者通過兩個步驟將模型蒸餾得結果格式統一
1. 用 R1 蒸餾並格式化好的示例（含 Thought + Solution）對模型做指令微調
2. 在推論段落後插入 <|begin_of_solution|> 標記，讓模型持續生成直到填補缺失的解答部分


最後的格式如下
```
<|begin_of_thought|>

{different step of thought separated by \n\n}

<|end_of_thought|>

<|begin_of_solution|>

{formated step-by-step final solution}

<|end_of_solution|>
```

##### 提示模板

為了引導模型系統性地展開「分析→總結→探索→反思→回溯→迭代」流程，再輸出簡潔解答，研究者會使用以下模板
```

Your role as an assistant involves thoroughly exploring questions through a systematic long

thinking process before providing the final precise and accurate solutions. This requires

engaging in a comprehensive cycle of analysis, summarizing, exploration, reassessment, reflection,

backtracing, and iteration to develop well-considered thinking process.

Please structure your response into two main sections: Thought and Solution.

In the Thought section, detail your reasoning process using the specified format:

“‘

<|begin_of_thought|>

{thought with steps separated with "\n\n"}

<|end_of_thought|>

”’

Each step should include detailed considerations such as analisying questions, summarizing

relevant findings, brainstorming new ideas, verifying the accuracy of the current steps, refining

any errors, and revisiting previous steps.

In the Solution section, based on various attempts, explorations, and reflections from the Thought

section, systematically present the final solution that you deem correct. The solution should

remain a logical, accurate, concise expression style and detail necessary step needed to reach the

conclusion, formatted as follows:

“‘

<|begin_of_solution|>

{final formatted, precise, and clear solution}

<|end_of_solution|>

”’

Now, try to solve the following question through the above guidelines:
```




# Models
### Sky-T1-32B 
使用 QwQ-32B-Preview 進行資料的收集，與 STILL-2 策略類似，但並沒有提到針對蒸餾對象的模型進行微調，而是直接使用 QwQ 模型輸出利用 GPT-4o-mini 進行重寫 以符合特定的格式。

並且也證明了格式的重要性，使用 APPs 資料集進行驗證，在沒有格式化的狀況下，只能假設結果都在最後一個區塊，正確率只能達到 25% ，而有格式化的狀況下則可以達到 90% 以上的正確率

使用 Qwen2.5 32B-Instruct 進行 SFT，使用 $1e{-5}$  學習率與96 batch size的設定訓練了
3 個 epoch，使 Qwen2.5 32B-Instruct 低於 o1 的表現可以提升至可以相互匹敵的程度。

![[Pasted image 20250504133354.png]]

在這個研究中，研究者發現了幾個現象
- 模型大小限制
	在初期研究，研究者也同時訓練了 7B 與 14B 的模型，相較於結果 32B 是較小的，但經過測試發現這些小模型不僅沒有獲得明顯的提升，並且有較高的機率出現重複的內容，導致較低的效率。研究者認為 32B 是一個分水嶺，在小的模型就沒辦法很好的獲取蒸餾的訊息。
- 資料混合的重要性
	在初期研究，研究者先是聚焦在數學領域蒸餾 32B 模型，驗證得到 43% 的正確率後，在使用編碼領域加入蒸餾，發現模型在數學能力上下滑。而在使用數學領域與編碼領域資料集中較為困難的問題組成資料集並進行訓練後，發現能力下滑的現象被解決了。這個發現近一步驗證 STILL-2 研究的假設。

### DeepSeek-R1

在這篇研究中，產生了數個模型，包含 2 個全尺寸模型與六個蒸餾模型
- DeepSeek-R1
- DeepSeek-R1-Zero
- DeepSeek-R1-Distill-Qwen-1.5B
- DeepSeek-R1-Distill-Qwen-7B
- DeepSeek-R1-Distill-Qwen-14B
- DeepSeek-R1-Distill-Qwen-32B
- DeepSeek-R1-Distill-Llama-8B
- DeepSeek-R1-Distill-Llama-70B

![[Pasted image 20250504141248.png]]

在 DeepSeek-R1 的論文中用於訓練 DeepSeek-R1 的資料由兩個部分組成
- Reasoning Data
	由數個步驟微調而來
	1. 冷啟動微調
		直接對 DeepSeek-R1-Zero 題詞，引導模型輸出 CoT 輸出，並經由人工後處理得到高品質資料，接著進行 SFT 微調。
	2. 推理導向強化學習
		根據模型輸出的格式、語言、正確率進行強化學習。
	3. 推理題詞與 Reject-Sampling
		收集模型輸出，並剔除錯誤答案、混合語言與宂餘的推理段落 ( Reject-sampling )
- Non-Reasoning
	使用 DeepSeek-V3 SFT 資料集配合輸出 CoT 的提示詞收集而成

總共收集 800K 的資料。

這些資料接著使用 SFT 與 RL 對 DeepSeek-V3 進行訓練，將訓練完成的模型稱為 DeepSeek-R1。
### Bespoke-Stratos-32B
使用與 Sky-T1 一樣的訓練方法，使用 DeepSeek-R1 ( Better then o1 ) 作為蒸餾來源。

經過 SFT 訓練後，可以達到與 DeepSeek-R1 相差不多的表現，並且相較於 DeepSeek-R1-Distill-Qwen-32B ( 使用與 DeepSeek-R1 一樣的資料集，使用 STF 蒸餾 ) 使用 47 倍較少的資料。



# 結論
## 模型大小對於蒸餾的影響
在 Sky-T1 的技術報告中，研究者認為 32B 以上的模型，能夠最好承接大模型的能力，並且把 Qwen2.5-14B-Coder-Instruct 模型特別提出進行論述支持，在該模型的訓練結果並沒有得到更多的效能提升。

而 DeepSeek-R1 的報告中，使用Qwen2.5-14B 模型，表現則與 32B 大小模型的效能相差不多。
![[Pasted image 20250504170605.png]]

在 DeepSeek-R1 報告中，並未提到模型提升幅度，這個問題可能需要近一步研究。

## SFT 的效率與可控性
在報告中的三種模型都在訓練過程使用 SFT ，並且都顯示出 SFT 具有以下優勢
- 收斂迅速
- 穩定提升模型對齊度
但同時，在 DeepSeek-R1 的報告中也提到使用 SFT 訓練的模型還需使用 RL 以對齊使用者友善，這也是 SFT 的缺點。