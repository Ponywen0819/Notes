# 方法
核心方法為**隨機漫步**，定義長度為 $l$ ，  $c_i$ 表示在漫步中第 $i$ 個節點，由 $c_0$ 開始
$$
P(c_i = x| c_{i-1} = v) = \begin{cases}
\frac{\pi_{vx}}{Z} , &\text{if}(v,x)\in E\\
0, &\text{otherwise}
\end{cases}
$$


