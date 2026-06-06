#note

# 列表與 ForEach Identity

> 怎麼顯示一串資料,以及一個關鍵但常被忽略的觀念 —— **identity(識別)**。SwiftUI 靠 identity 判斷「列表更新後,哪一列是同一列、哪一列是新的」,選錯會造成動畫亂跳、狀態錯位、效能變差。承接 [[核心架構與機制#View 的識別 (Identity)|核心架構的 Identity 段落]]。

---

# 一、`ForEach` 與 `List` 基礎

- **`ForEach`**:把一個集合「展開成多個 View」的迴圈式建構器(它本身只是產生 View,不帶外觀)。
- **`List`**:可捲動、有列樣式的容器,常和 `ForEach` 一起用。

```swift
List {
    ForEach(items) { item in
        Text(item.name)
    }
}

// 簡單情況可直接把資料給 List
List(items) { item in
    Text(item.name)
}
```

> `List` 負責「捲動 + 列外觀 + 分隔線」;`ForEach` 負責「把資料一個個變成 View」。兩者常一起出現,但職責不同。

---

# 二、核心:SwiftUI 為什麼需要 identity?

當列表資料變了(新增、刪除、重排),SwiftUI 要回答一個問題:

> 「新的這串 View,和上一次那串,**哪些是同一個、哪些是新的、哪些不見了?**」

它靠 **identity** 來配對前後兩次的列。配對正確,才能:

- 對「真的新增/刪除的列」播放正確的**插入/移除動畫**。
- 讓「同一列」的 `@State`、捲動位置、輸入焦點**跟著正確的資料留住**。
- 只更新有變動的列,不整個列表重建 → **效能**。

所以 `ForEach` **必須知道每個元素的「身份」**。這就是它要求 `id` 的原因。

---

# 三、`ForEach` 取得 identity 的三種方式

## 方式 1:元素遵循 `Identifiable`(最佳)

讓資料型別自帶一個穩定的 `id`,`ForEach` 就會自動採用:

```swift
struct Item: Identifiable {
    let id = UUID()        // 穩定、唯一的身份
    var name: String
}

ForEach(items) { item in    // 不用寫 id:,自動用 item.id
    Text(item.name)
}
```

> 這是**最推薦**的做法 —— 身份跟著資料模型走,最不容易出錯。`Identifiable` 只要求提供一個 `id` 屬性。

## 方式 2:用 KeyPath 指定某個欄位當 id

資料沒有 `Identifiable` 但有「天然唯一鍵」(如後端的主鍵)時:

```swift
ForEach(users, id: \.userID) { user in
    Text(user.name)
}
```

## 方式 3:`id: \.self`(小心使用)

把元素「值本身」當身份。**只適合「值唯一且穩定」的簡單資料**(如固定字串清單):

```swift
ForEach(["紅", "綠", "藍"], id: \.self) { color in
    Text(color)
}
```

⚠️ 風險:若值會重複或會變動,身份就會錯亂(見下一節陷阱)。

---

# 四、用 index 當 id —— 經典陷阱

很常見的錯誤寫法:

```swift
ForEach(0..<items.count, id: \.self) { i in     // ❌ 用位置當身份
    Text(items[i].name)
}
```

問題:**index 是「位置」不是「身份」**。一旦你在中間插入/刪除/重排,每個元素的 index 都變了,SwiftUI 會誤判:

- 動畫亂跳(以為「第 2 列的內容變了」,其實是整串往後移)。
- 列裡的 `@State`(展開狀態、輸入內容)**錯位到別的資料**上。
- 刪除時可能更新到錯的列,甚至崩潰(index 越界)。

✅ 正解:用穩定身份(`Identifiable` 或唯一鍵):

```swift
ForEach(items) { item in        // 身份綁在資料上,重排也不會錯
    Text(item.name)
}
```

> 只有在「資料純唯讀、永不增刪重排、列裡也沒有狀態」時,用 index 才相對安全 —— 但這種情況少,**預設別用 index 當 id**。

---

# 五、`List` 的常用功能

## 刪除、移動

`ForEach` 提供現成的滑動刪除與拖曳重排:

```swift
List {
    ForEach(items) { item in
        Text(item.name)
    }
    .onDelete { offsets in items.remove(atOffsets: offsets) }
    .onMove   { from, to in items.move(fromOffsets: from, toOffset: to) }
}
.toolbar { EditButton() }     // 進入編輯模式
```

## 分區 (Section)

```swift
List {
    Section("進行中") { ForEach(active)  { Text($0.name) } }
    Section("已完成") { ForEach(done)    { Text($0.name) } }
}
```

## 下拉刷新、搜尋

```swift
List(items) { item in Text(item.name) }
    .refreshable { await reload() }            // 下拉刷新(async)
    .searchable(text: $query)                  // 內建搜尋列
```

> `.refreshable` 直接接 `async`,呼應 [[生命週期與 task#三、補充:async / await 與 Task 的最小認識|async/await]]。

---

# 六、`List` vs `LazyVStack` vs `ForEach` 單用

| 用法 | 特性 | 何時用 |
| --- | --- | --- |
| `List` | 內建列樣式、分隔線、滑動刪除、section;**惰性載入**(只建可見列) | 標準清單畫面 |
| `ScrollView { LazyVStack }` | 高度自訂外觀;惰性載入;無內建列功能 | 要完全自訂卡片/排版的捲動列表 |
| `ScrollView { VStack { ForEach } }` | **一次全部建立**(非惰性) | 元素很少時才用 |

關鍵差異:**`List` 和 `LazyVStack` 是「惰性」的** —— 只建立目前看得到的列,捲到才建,適合長列表;`VStack + ForEach` 會一次把所有列都建出來,資料一多就卡。

> 不論哪一種,只要用到 `ForEach`,**identity 的規則都一樣適用**(第三、四節)。

---

# 七、`.id()` 與強制重建

呼應 [[核心架構與機制#View 的識別 (Identity)|核心架構]]:手動改一個 View 的 `.id(...)` 會讓 SwiftUI 當成「全新的 View」,進而**重置它的 `@State`**。

```swift
ProfileView(user: user)
    .id(user.id)        // user 換人時,強制整個 ProfileView 重建、狀態歸零
```

可用來「換資料時重置畫面狀態」,但也要小心別不小心讓狀態無故消失。

---

# 八、總結

- `List` 管捲動與列外觀,`ForEach` 把資料變成 View,職責分開。
- **identity 是核心**:SwiftUI 靠它配對前後兩次的列,才能正確做動畫、留住列狀態、只更新有變的列。
- 取得 identity 的優先序:**`Identifiable`(最佳)> 唯一鍵 KeyPath > `id: \.self`(限唯一穩定值)**。
- **別用 index 當 id** —— 它是位置不是身份,增刪重排會錯亂。
- 長列表用**惰性**容器(`List` / `LazyVStack`),不要 `VStack + ForEach` 全建。
- `.id(...)` 改變 = 強制重建並重置 `@State`,慎用。

---

# 延伸閱讀

- [[核心架構與機制]] —— Identity 與 Render Loop 的根本機制
- [[State 與 Binding 詳解]] —— 為什麼「列裡的狀態」會因 id 錯位而跑掉
- [[導覽 Navigation]] —— List + NavigationLink 的常見組合
- [[佈局系統 Layout]] —— LazyVStack 等捲動容器的佈局
