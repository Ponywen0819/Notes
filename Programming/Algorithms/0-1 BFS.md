#note

# 0-1 BFS

當圖的**邊權只有 0 或 1** 時,不必動用 [[priority_queue|Dijkstra]] 的 heap,用一個 **deque(雙端佇列)** 就能在 $O(V+E)$ 求最短路。是介於普通 [[2D grid BFS|BFS]] 與 Dijkstra 之間的特例。

# 核心想法

普通 BFS 之所以正確,是因為 queue 裡的節點**距離單調遞增**(同一層一起處理)。邊權有 0/1 時想維持這個單調性:

- 走一條 **權重 0** 的邊 → 距離不變 → 該鄰居應該和當前節點**同一層**處理 → `push_front`(插到隊首,馬上輪到)。
- 走一條 **權重 1** 的邊 → 距離 +1 → 排到**下一層** → `push_back`(插到隊尾)。

> 直覺:`push_front` = 「這步不花成本,插隊先處理」;`push_back` = 「這步花 1,乖乖排隊」。deque 因此始終保持「隊首距離 ≤ 隊尾距離」,和普通 BFS 一樣一層層擴散。

# 演算法骨架

```cpp
deque<int> q;
dist[src] = 0;
q.push_front(src);

while (!q.empty()) {
    int u = q.front();
    q.pop_front();
    for (auto [v, w] : adj[u]) {        // w ∈ {0, 1}
        if (dist[u] + w < dist[v]) {    // 鬆弛 (relax)
            dist[v] = dist[u] + w;
            if (w == 0) q.push_front(v); // 0 邊:插隊首
            else        q.push_back(v);  // 1 邊:排隊尾
        }
    }
}
```

> **同一節點可能被多次入隊**(先用較大距離入隊,之後找到更短又入隊)。上面靠「鬆弛條件 `dist[u]+w < dist[v]`」保證只在更優時才更新,結果仍正確。要更快可加**過期跳過**:出隊時若 `dist` 已更小就 `continue`(需在 deque 存 `(dist, u)`)。

# 網格範例:移除最少障礙物(LeetCode 2290 型)

`grid[r][c]==0` 是空格(走過去成本 0)、`==1` 是障礙(走過去要「移除」,成本 1)。求從左上到右下**最少移除幾個障礙**。

```cpp
#include <bits/stdc++.h>
using namespace std;

int minObstacles(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    static constexpr int DIRS[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
    vector<vector<int>> dist(m, vector<int>(n, INT_MAX));

    deque<pair<int,int>> q;
    dist[0][0] = 0;
    q.push_front({0, 0});

    while (!q.empty()) {
        auto [r, c] = q.front();
        q.pop_front();
        for (auto& [dr, dc] : DIRS) {
            int nr = r + dr, nc = c + dc;
            if (nr < 0 || nc < 0 || nr >= m || nc >= n) continue;
            int w = grid[nr][nc];               // 進入該格的成本:0 或 1
            if (dist[r][c] + w < dist[nr][nc]) {
                dist[nr][nc] = dist[r][c] + w;
                if (w == 0) q.push_front({nr, nc});
                else        q.push_back({nr, nc});
            }
        }
    }
    return dist[m-1][n-1];
}

int main() {
    vector<vector<int>> grid = {
        {0,1,1},
        {1,1,0},
        {1,1,0},
    };
    cout << minObstacles(grid) << "\n";   // 2
}
```

方向表寫法見 [[range-based for 與結構化綁定]]。

# 關鍵點

1. **成本掛在「邊」上**:上例的 `w` 是「**進入鄰居**」的成本,取決於鄰居格的值,不是當前格。
2. **鬆弛條件不可省**:`dist[u]+w < dist[v]` 是正確性關鍵(對比普通 BFS 只需 `visited`)。因為 0 邊會讓節點被重新考慮。
3. **push_front vs push_back 別寫反**:0 → front、1 → back。寫反會失去單調性、答案錯。
4. **早停**:出隊到終點即可回傳(第一次出隊時距離已最優);上例為求簡潔跑到隊列空,兩者皆可。

# 什麼時候用哪個

| 邊權 | 用法 | 複雜度 |
| --- | --- | --- |
| 全部相同(如都 1) | 普通 [[2D grid BFS\|BFS]](queue) | $O(V+E)$ |
| 只有 0 和 1 | **0-1 BFS**(deque) | $O(V+E)$ |
| 任意非負 | [[priority_queue\|Dijkstra]](heap) | $O(E\log V)$ |

> 一句話:**0-1 BFS = 用 deque 把 Dijkstra 的 $\log V$ 拿掉的特例**。看到「邊權只有 0/1」(可走 vs 要付一次代價)就想到它。
