#note

# 2D 網格 BFS(最短路)

BFS 從起點一層一層往外擴,**第一次抵達某格時就是最短步數**(每步權重相同時成立)。網格題的標準骨架:`queue` + `dist`(或 `visited`)+ 方向表。

方向表寫法見 [[range-based for 與結構化綁定]]。

# 標準範例:回傳起點到終點的最短步數

```cpp
#include <bits/stdc++.h>
using namespace std;

// grid[r][c] == 0 可走、== 1 是牆;回傳 (0,0) 到 (m-1,n-1) 最短步數,不可達回 -1
int bfs(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    if (grid[0][0] == 1 || grid[m-1][n-1] == 1) return -1;

    static constexpr int DIRS[4][2] = {{0,1}, {0,-1}, {1,0}, {-1,0}};
    vector<vector<int>> dist(m, vector<int>(n, -1));  // -1 = 尚未拜訪(兼作 visited)
    queue<pair<int,int>> q;

    dist[0][0] = 0;
    q.push({0, 0});

    while (!q.empty()) {
        auto [r, c] = q.front();
        q.pop();
        if (r == m-1 && c == n-1) return dist[r][c];   // 抵達終點

        for (auto& [dr, dc] : DIRS) {
            int nr = r + dr, nc = c + dc;
            if (nr < 0 || nc < 0 || nr >= m || nc >= n) continue;  // 越界
            if (grid[nr][nc] == 1) continue;                       // 牆
            if (dist[nr][nc] != -1) continue;                      // 已拜訪過
            dist[nr][nc] = dist[r][c] + 1;
            q.push({nr, nc});
        }
    }
    return -1;   // 走完仍沒到終點
}

int main() {
    vector<vector<int>> grid = {
        {0,0,0},
        {1,1,0},
        {0,0,0},
    };
    cout << bfs(grid) << "\n";   // 4
}
```

# 三個關鍵點

1. **先標記、再入隊**:設 `dist[nr][nc]` 的**同時**就 `push`,別等到出隊才標。否則同一格可能被多次入隊、退化成重複走訪。
2. **`dist == -1` 一石二鳥**:同時當「沒拜訪過」的 visited 旗標與「步數」的記錄,不用另開 `visited` 陣列。
3. **層序性質**:普通 queue 的 BFS,只要每步成本相同,出隊順序就是距離遞增 → 第一次碰到終點即最短。

# 常見變體

## 多源 BFS(一次從多個起點擴散)
把**所有起點**一開始就全部入隊、`dist` 設 0。常見於「最近的 X 有多遠」類題(如爛橘子、離最近 0 的距離)。
```cpp
for (每個起點 (r,c)) { dist[r][c] = 0; q.push({r,c}); }
// 之後迴圈完全一樣
```

## 0-1 BFS(邊權只有 0 或 1)→ 用 deque
權重 0 的鄰居 `push_front`、權重 1 的 `push_back`,維持隊列單調 → $O(V+E)$ 求最短路,不必動用 Dijkstra 的 heap。你之前看的 `findSafeWalk` 就是這招。
```cpp
deque<pair<int,int>> q;
// cost 0: q.emplace_front(nr, nc);   cost 1: q.emplace_back(nr, nc);
```

## 需要不同邊權(>1)
BFS 不再成立,改用 [[priority_queue|Dijkstra]]。

# 8 方向 / 對角可走
方向表補上四個對角即可:
```cpp
static constexpr int DIRS[8][2] =
    {{0,1},{0,-1},{1,0},{-1,0},{1,1},{1,-1},{-1,1},{-1,-1}};
```

# 複雜度
- 時間 $O(m\times n)$:每格最多入隊一次。
- 空間 $O(m\times n)$:`dist` 陣列 + 隊列。
