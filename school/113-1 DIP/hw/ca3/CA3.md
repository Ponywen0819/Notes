## Problem 2
![[prob2_res_1.png]]

![[prob2_res_2.png]]

Alpha 方法在兩種情況下都表現不優。在低雜訊的情況下高斯濾波器表現最好，在高雜訊的狀況下中值法反而是最好的，並且相較於其他的方法保留了多一點細節。

高斯濾波器的表現算是中規中矩，不論在何種情況下都可以表現去接受的表現。

## Prob3
![[prob3_res_1.png]]
![[prob3_res_2.png]]

最明顯的發現是 midpoint 在高雜訊環境中表現十分糟糕，並且在低雜訊的環境中也顯得非常糟糕。

Adaptive median filter 表現非常優異。

## Prob 4
![[prob4_res_1.png]]
![[prob4_res_2.png]]
在此類型的雜訊中，outlier 的效果最好，其餘的方法則是相差不遠。

其中 midpoint 與 mean 表現不優。

