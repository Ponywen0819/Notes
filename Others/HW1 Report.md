#homework

## Library

- heapq
- csv
- json

  

## Adjacent Matrix
1. 初始化 Adjacent Matrix ，將所有元素設置為 0
2. 尋訪所有 link ，並將 matrix 中相應的元素設置為 1

## Betweenness value
1. 對所有 node 進行尋訪
	1. 將本次迭代的 node 設定為 source，利用 Dijkstra 計算最小路徑
	2. 建構 path_matrix\[s]\[t]\[v]，其意義為 $\sigma_{st}(v)$
	3. 尋訪 path_matrix\[s]\[t] ，將相應數值相加，得出 betweenness