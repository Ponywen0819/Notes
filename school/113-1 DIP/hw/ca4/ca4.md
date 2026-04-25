#homework

#### Prob1 **Execution time comparison**
![[Pasted image 20241209212901.png]]

#### Prob 2

a. 
![[Pasted image 20241209214214.png]]

b. 
![[Pasted image 20241209214232.png]]

c. 
![[Pasted image 20241209214249.png]]'

d.
預設情況下，整體影像比較模糊，在 $scale = 8$ 的狀況下，圖像輪廓部分變得更加突出。而 $scale = -8$ 時對比度增強。


#### Prob3

a. 
![[Pasted image 20241209221839.png]]
-  小波分解的水平分量捕捉了圖像中水平方向的邊緣或變化。
-  垂直分量捕捉圖像中的垂直邊緣或變化。
- 對角分量描述圖像中對角方向的邊緣或變化。

b.
![[Pasted image 20241209221952.png]]
就他被歸零了，但 wavecut 本來就只作用於最高層，所以其他分解都沒有影響

c.
![[Pasted image 20241209223048.png]]
- 與原影像相比，重建影像只保留了高頻細節，如邊緣和紋理，整體圖像會顯得缺乏主體結構信息，更多的是輪廓和細節的表現。

#### Prob4
![[Pasted image 20241209223844.png]]
![[Pasted image 20241209223822.png]]
最佳平滑層級 level = 1
對應的 PSNR = 13.6024

#### Prob5

![[Pasted image 20241209225253.png]]

![[Pasted image 20241209224517.png]]

![[Pasted image 20241209224529.png]]
最終重組的影像與原本的影像基本上一樣，平均差異在 $7.6939e-12$
