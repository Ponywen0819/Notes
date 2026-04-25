#report 
# 亮點
- 使用及低成本 ( $450 ) 達到可以匹敵 o1-preview 的實力

17K data used to train Sky-T1-32B-Preview.

![[Pasted image 20250504082612.png]]

## Data curation Process
1. Generate training data with QwQ-32B-Preview ( which is comparable to o1-preview )，
2. Discard QwQ samples if they are incorrect according to the solutions.
	Called reject sampling, every response will compare with correct answer.
3. Rewrite QwQ traces with 4o-mini into well-formatted version
   without rewrite, it's hard to parse the response, will cause bad performance.
## Training
Fine tune from Qwen2.5-32B-Instruct( which is worse then o1-preview )

![[Pasted image 20250504083728.png]]

