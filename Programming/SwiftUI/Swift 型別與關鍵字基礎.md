#note

# Swift 型別與關鍵字基礎

> 這份筆記整理 Swift 中用來「定義型別」的一族關鍵字 —— `protocol`、`struct`、`class`、`enum`、`extension` 等,以及和它們常一起出現的 `associatedtype`、`some` / `any`、`typealias`。這些是讀懂 [[核心架構與機制|SwiftUI]] 程式碼的前提。全篇用 C# 做對照,方便你快速建立直覺。

---

# 一、先建立全貌

Swift 定義「一個型別」主要有四個關鍵字,加上一個「擴充既有型別」的關鍵字:

| 關鍵字 | 是什麼 | 值/引用 | 最接近的 C# 概念 |
| --- | --- | --- | --- |
| `struct` | 結構(資料的容器) | **值類型** | C# `struct` |
| `class` | 類別 | **引用類型** | C# `class` |
| `enum` | 列舉(但功能強大很多) | 值類型 | C# `enum`(但更強) |
| `protocol` | 規格 / 合約 | —(只是規範) | C# `interface` |
| `extension` | 替既有型別加東西 | —(擴充) | C# 擴充方法 / `partial` |

> 一句話分類:`struct`/`class`/`enum` 是**「真的造出一個型別」**;`protocol` 是**「規定型別要符合什麼條件」**;`extension` 是**「對既有型別補東西」**。

---

# 二、`protocol` —— 規格 / 合約

最接近 **C# 的 `interface`(介面)**。它只規定「符合它的型別必須提供哪些屬性與方法」,本身**不寫實作**。

```swift
protocol Greetable {
    var name: String { get }      // 規定:必須有可讀的 name
    func greet() -> String        // 規定:必須有 greet() 方法
}

struct Person: Greetable {        // 「: Greetable」= 遵循(conform)這個 protocol
    let name: String
    func greet() -> String { "Hi, I'm \(name)" }   // 被強制要補上實作
}
```

- `struct Person: Greetable` 的冒號 = **「Person 遵循 Greetable」**(術語:conform to)。
- protocol 規定的東西,遵循者**一定要實作**,少寫編譯器就報錯 —— 這份「保證」正是 protocol 的價值。
- 一個型別可以同時遵循**多個** protocol:`struct Person: Greetable, Identifiable { ... }`。

## 和 C# interface 的對照

| C# | Swift |
| --- | --- |
| `interface IGreetable` | `protocol Greetable` |
| `class C : IGreetable` | `struct C: Greetable` |
| 「實作介面」 | 「遵循 protocol (conform)」 |

> Swift 的 protocol 比 C# interface 更強:可以有**預設實作**(透過 `extension`)、可作用在 `struct`/`enum` 上、能搭配泛型約束。入門先理解成「介面 / 合約」就完全夠用。

---

# 三、`struct` vs `class` —— 值類型 vs 引用類型

兩者語法幾乎一樣(都能有屬性、方法、初始化),**最關鍵的差別是「複製行為」**:

| | `struct`(值類型) | `class`(引用類型) |
| --- | --- | --- |
| 指派/傳遞時 | **複製一份**(各自獨立) | **共用同一個**(傳的是參考) |
| 繼承 | ❌ 不能繼承 | ✅ 可以繼承 |
| 何時用 | 純資料、輕量、不可共享狀態 | 需要共享、有身份、需繼承 |

```swift
struct PointS { var x = 0 }
class  PointC { var x = 0 }

var a = PointS(); var b = a; b.x = 99   // a.x 仍是 0(b 是複製品)
var c = PointC(); var d = c; d.x = 99   // c.x 變成 99(c、d 指向同一個物件)
```

> 這正是 [[核心架構與機制|SwiftUI]] 裡「View 是 struct(值類型)、可以被頻繁建立丟棄」的根本原因 —— 複製值類型很廉價,而且不會有「兩個地方不小心共用同一份狀態」的問題。

---

# 四、`enum` —— 列舉(比 C# 強大很多)

不只是「一串常數」,Swift 的 enum 可以**帶資料**、有方法、做模式比對,常用來表達「有限的幾種狀態」。

```swift
enum LoadState {
    case idle                      // 單純的狀態
    case loading
    case success(data: String)     // 可以「夾帶」關聯值
    case failure(error: String)
}

let state = LoadState.success(data: "OK")

switch state {                     // 用 switch 拆解
case .idle, .loading:      print("等待中")
case .success(let data):   print("拿到 \(data)")
case .failure(let err):    print("錯誤 \(err)")
}
```

- `case success(data: String)` 這種「帶關聯值」的能力是 C# enum 沒有的(比較像 C# 的 record + 模式比對的綜合體)。
- 在 SwiftUI 裡很常用 enum 表達畫面狀態(載入中 / 成功 / 失敗),搭配 `switch` 決定要顯示哪個畫面。

---

# 五、`extension` —— 替既有型別補東西

可以對**任何既有型別**(連系統的 `Int`、`String` 都行)再加方法、計算屬性,或讓它額外遵循某個 protocol。

```swift
extension Int {
    var isEven: Bool { self % 2 == 0 }   // 替 Int 加一個計算屬性
}
print(4.isEven)   // true

extension Person: Equatable { }          // 讓既有型別「額外遵循」某 protocol
```

對照 C#:大致等於 **擴充方法 (extension method)**,但更廣 —— 還能加計算屬性、加 protocol 遵循。也常被拿來**整理程式碼**(把不同職責拆到不同 extension)。

## protocol + extension = 預設實作

這是 Swift 很常見的組合:在 protocol 上用 extension 提供**預設行為**,遵循者沒寫就用預設,寫了就覆蓋。

```swift
extension Greetable {
    func greet() -> String { "Hi, I'm \(name)" }  // 預設實作
}
struct Robot: Greetable { let name: String }       // 沒寫 greet() 也能用,套用預設
```

---

# 六、常一起出現的關鍵字

## `associatedtype` —— protocol 裡的「待定型別」

protocol 裡先佔一個型別的位子,實際是什麼由遵循者決定。

```swift
protocol Container {
    associatedtype Item            // 「裝什麼」之後再說
    func add(_ item: Item)
}
```

對照 C#:類似 interface 的泛型參數 `interface IContainer<Item>`,但 Swift 把它寫在內部當「關聯型別」。你在 [[核心架構與機制|SwiftUI]] 看到的 `protocol View { associatedtype Body: View }` 就是這個 —— 每個 View 的 `body` 具體型別不同,所以用 associatedtype 留待各自決定。

## `some` 與 `any` —— 兩種「不講死是哪個型別」的方式

當回傳/儲存的東西「只知道它遵循某 protocol,但不想/不能寫出確切型別」時:

| 寫法 | 意思 | 特性 |
| --- | --- | --- |
| `some View` | **不透明型別**:其實是某個**確定且單一**的型別,只是對外隱藏 | 效能好,編譯期決定 |
| `any View` | **存在型別**:可裝任何遵循 View 的型別,執行期才知道 | 較彈性,但有額外開銷 |

```swift
func makeView() -> some View { Text("Hi") }   // 確定是 Text,只是不寫出來
let views: [any View] = [Text("a"), Image(systemName: "star")]  // 混裝不同型別
```

> 在 SwiftUI,`var body: some View` 用 `some` 是因為「body 的型別其實是固定的某一種(編譯器算得出來),只是又臭又長不想手寫」。預設優先用 `some`,需要「同一個容器裝多種型別」時才用 `any`。

## `typealias` —— 取別名

純粹給某個型別一個更好懂的名字,不造新型別。對照 C# 的 `using Name = ...;`。

```swift
typealias UserID = Int
func find(_ id: UserID) { }   // 讀起來比 Int 清楚
```

---

# 七、速查總結

- **造型別**:`struct`(值、不可繼承)、`class`(引用、可繼承)、`enum`(可帶資料的狀態)。
- **定規格**:`protocol`(≈ C# interface),遵循者必須實作。
- **補東西**:`extension`,還能配 protocol 做「預設實作」。
- **泛用配件**:`associatedtype`(protocol 內待定型別)、`some`/`any`(隱藏具體型別的兩種方式)、`typealias`(別名)。

讀懂這些,再回頭看 [[核心架構與機制|SwiftUI 核心架構與機制]] 裡的 `struct ContentView: View { var body: some View { ... } }`,每個字的意思就都清楚了。

---

# 延伸閱讀

- [[核心架構與機制]] —— SwiftUI 的整體架構與機制(本筆記是它的語言基礎)
