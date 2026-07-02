#note

# C++ STL:`priority_queue`(優先佇列)

`priority_queue` 是一個**容器適配器 (container adapter)**,底層用**二元堆積 (binary heap)** 實作。特性:**每次取出的都是目前最大(或最小)的元素**,不是先進先出。

- header:`#include <queue>`
- **預設是 max-heap**(大的先出)。
- 主要操作複雜度:`push` / `pop` $O(\log n)$、`top` $O(1)$。

# 基本用法

```cpp
#include <queue>
priority_queue<int> pq;   // 預設 max-heap

pq.push(3);
pq.push(1);
pq.push(4);

pq.top();   // 4  (看最大,不移除)
pq.pop();   // 移除最大 (4),無回傳值
pq.size();  // 剩幾個
pq.empty(); // 是否為空
```

> 注意:`pop()` **不回傳**被移除的值。要用值請先 `top()` 取出、再 `pop()`。

# 常用成員函式

| 函式                   | 作用                | 複雜度         |
| -------------------- | ----------------- | ----------- |
| `push(x)`            | 插入元素              | $O(\log n)$ |
| `emplace(args...)`   | 原地建構後插入(免多一次複製)   | $O(\log n)$ |
| `pop()`              | 移除頂端(最大)元素        | $O(\log n)$ |
| `top()`              | 讀取頂端元素(const ref) | $O(1)$      |
| `size()` / `empty()` | 大小 / 是否為空         | $O(1)$      |

> 沒有 `clear()`、不能迭代、不能隨機存取 —— 它只保證「頂端是極值」。要清空就 `pq = {};` 或逐一 pop。

# 做成 min-heap(最小先出)

最常見的需求。兩種寫法:

```cpp
// 寫法一:用內建 greater 比較器(最常用)
priority_queue<int, vector<int>, greater<int>> minpq;

// 寫法二:塞入時取負,取出時再取負 (只適用數值、小技巧)
priority_queue<int> pq;
pq.push(-x);        // 存 -x
int v = -pq.top();  // 取出還原
```

模板三個參數:`priority_queue<T, Container, Compare>`
- `T`:元素型別
- `Container`:底層容器,預設 `vector<T>`
- `Compare`:比較器,預設 `less<T>`(→ max-heap);換成 `greater<T>` → min-heap。

# 自訂比較器 (Custom Comparator)

**關鍵直覺(容易記反)**:`Compare(a, b)` 回傳 `true` 代表「a 的優先權**比 b 低**」→ a 會排在 b 後面、比較晚出來。
所以想要「**小的先出**」,比較器要寫成 `a > b`(讓大的優先權低)。

## 方式一:lambda(C++11 起,最直覺)
```cpp
auto cmp = [](const P& a, const P& b) {
    return a.cost > b.cost;   // cost 小的先出 (min-heap by cost)
};
priority_queue<P, vector<P>, decltype(cmp)> pq(cmp);
```

## 方式二:struct + `operator()`
```cpp
struct Cmp {
    bool operator()(const P& a, const P& b) const {
        return a.cost > b.cost;   // cost 小的先出
    }
};
priority_queue<P, vector<P>, Cmp> pq;   // 不必傳 cmp 進建構子
```

## 方式三:pair 直接排序
`pair` 有內建字典序比較,不用寫比較器就能排:
```cpp
priority_queue<pair<int,int>> pq;   // 先比 first,再比 second (max-heap)
pq.push({dist, node});
// Dijkstra 常見:min-heap of (dist, node)
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
```

# 典型應用

## Dijkstra 最短路
```cpp
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq; // (dist, node)
dist[s] = 0; pq.push({0, s});
while (!pq.empty()) {
    auto [d, u] = pq.top(); pq.pop();
    if (d > dist[u]) continue;          // 過期條目,跳過
    for (auto [v, w] : adj[u])
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            pq.push({dist[v], v});
        }
}
```

> 這裡的 `auto [d, u] = pq.top()` 與 `for (auto [v, w] : adj[u])` 用到**結構化綁定 + range-based for**,寫法細節見 [[range-based for 與結構化綁定]]。

## Top-K:找最大的 K 個 → 用「大小為 K 的 min-heap」
```cpp
// 維持一個 min-heap,超過 K 個就 pop 掉最小的 → 留下的就是最大的 K 個
priority_queue<int, vector<int>, greater<int>> pq;
for (int x : nums) {
    pq.push(x);
    if (pq.size() > K) pq.pop();
}
// pq.top() 就是第 K 大
```

> 直覺:要「最大的 K 個」反而用 **min-heap**,因為要能隨時丟掉「目前這 K 個裡最小的」。

# 常見陷阱

- **比較器方向記反**:回傳 `true` = 優先權**低** = 晚出。想要小的先出就寫 `a > b`。
- **`pop()` 不回傳值**:先 `top()` 再 `pop()`。
- **不能遍歷 / 不能改中間元素**:需要「可更新優先權」時,用「lazy deletion(過期條目跳過)」如上方 Dijkstra,或改用 `set` / 索引堆。
- **溢位**:數值大時比較器內的運算(如相減)可能溢位,直接比大小較安全。

# 與其他容器的取捨

| 需求 | 選擇 |
| --- | --- |
| 只要反覆取極值 | `priority_queue` |
| 要有序、可查找、可刪任意元素 | `std::set` / `multiset` |
| 先進先出 | `std::queue` |
| 兩端操作 | `std::deque` |
