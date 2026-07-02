#note

# Range-based for 與結構化綁定 (Structured Bindings)

兩個 C++ 特性疊在一起,就能寫出很乾淨的走訪迴圈:
- **range-based for**(C++11):不用管索引/迭代器,直接跑遍容器。
- **structured bindings**(C++17):把 `pair` / `tuple` / 陣列 / struct **一次拆成多個名字**。

# Range-based for 基礎

```cpp
vector<int> v = {1, 2, 3};

for (int x : v)          // 複製一份 (x 是副本)
    cout << x;

for (int& x : v)         // 參考:可以改動容器內容
    x *= 2;

for (const auto& x : v)  // 唯讀又不複製 —— 大物件(string/vector)首選
    cout << x;
```

**三種選法的口訣**:

| 寫法                        | 用途                    |
| ------------------------- | --------------------- |
| `for (auto x : v)`        | 只讀小型別(int、char),複製沒差  |
| `for (auto& x : v)`       | 要**修改**容器內元素          |
| `for (const auto& x : v)` | 只讀、但**避免複製**大物件(預設首選) |

# 結構化綁定

把一個「多欄位的東西」拆成多個變數:

```cpp
pair<int,int> p = {3, 7};
auto [a, b] = p;         // a=3, b=7

tuple<int,int,int> t = {1, 2, 3};
auto [x, y, z] = t;      // x=1, y=2, z=3
```

# 兩者合體:走訪的常見招式

## 走訪 pair / tuple 的容器

```cpp
vector<pair<int,int>> edges = {{0,1}, {1,2}};
for (auto& [u, v] : edges)      // 直接拆成 u, v,不用寫 e.first / e.second
    cout << u << "->" << v;
```

## 走訪 map(超常用)

```cpp
map<string,int> cnt;
for (auto& [key, val] : cnt)     // 不必再 it->first / it->second
    cout << key << ": " << val;
```

## 從 priority_queue / stack 取值

```cpp
auto [d, node] = pq.top();  pq.pop();   // 一行拆開頂端元素
```
(見 [[priority_queue]])

# 網格方向表(這篇的重點招式)

網格 BFS / DFS / Dijkstra 最常見的乾淨寫法:**用一張方向表 + 合體迴圈**取代四段重複判斷。

```cpp
static constexpr int DIRS[4][2] = {{0,1}, {0,-1}, {1,0}, {-1,0}};
//                                  右      左       下      上   (dx=row, dy=col)

for (auto& [dx, dy] : DIRS) {        // 一圈試一個方向
    int nx = cx + dx, ny = cy + dy;  // 鄰居座標
    if (nx < 0 || ny < 0 || nx >= m || ny >= n) continue;  // 越界檢查
    // ... 走訪 (nx, ny)
}
```

- **`DIRS[4][2]`**:4 個方向,每個是一對 `(dx, dy)`。
- **`static constexpr`**:class 共用一份、編譯期常數(方向表是不變的,不必每個物件複製)。
- **`for (auto& [dx, dy] : DIRS)`**:`DIRS` 每個元素是 `int[2]`,結構化綁定把它拆成 `dx`、`dy`。

> **對比土法**:沒有方向表就要把「上下左右」寫成四段幾乎一樣的 `nx=cx+1…` / `cx-1…` / … —— 又長又容易手滑。方向表把它收成一個迴圈。想加斜向就補 4 個對角:`{1,1},{1,-1},{-1,1},{-1,-1}`(共 8 方向)。

# 常見陷阱

- **忘了 `&` 導致改不動**:`for (auto [k,v] : m)` 拆的是**副本**,改 `v` 不會動到 map 裡的值;要改就 `for (auto& [k,v] : m)`。
- **大物件忘了 `const auto&`**:`for (auto [k,v] : bigMap)` 每圈複製整個 pair,效能差。唯讀就用 `const auto&`。
- **綁定數量要對齊**:`auto [a,b] = tuple<int,int,int>` 會編譯錯 —— 名字數必須等於欄位數。
- **結構化綁定不能跳過欄位**:沒有「只取第二個」的語法,要嘛全拆、要嘛用 `get<>()` / `.second`。
- **版本需求**:range-based for 要 C++11、結構化綁定要 **C++17**(編譯加 `-std=c++17`)。
