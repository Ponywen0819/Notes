#note

# 概念

在 C# 中，變數可以分為兩種類別：**值類型 (Value Type)** 與 **引用類型 (Reference Type)**。

# 值類型 (Value Type)

## 包含
- structure (`struct`)
- enum (`enum`)
- 所有內建數值類型（`int`、`double`、`bool`、`char`...）

## 特性
- 直接儲存資料本體
- 配置在堆疊 (stack) 上（區域變數情境下）
- 賦值時複製整個值
- 在 C# 中值類型為 `System.ValueType` 的派生
- 所有**數值類型都為 structure**

# 引用類型 (Reference Type)

## 包含
- `class`
- `interface`
- `delegate`
- `array`
- `string`（特殊：行為類似值類型，但本質是引用類型）
- `object`

## 特性
- 變數存的是「指向堆積 (heap) 上物件的引用」
- 賦值時複製引用，不複製物件本體
- 多個變數可指向同一物件
- 可為 `null`

# 值類型 vs 引用類型 比較表

| 維度 | 值類型 | 引用類型 |
| --- | --- | --- |
| 記憶體位置 | Stack（區域變數時） | Heap |
| 賦值行為 | 複製值 | 複製引用 |
| 預設值 | 0 / false / 結構零值 | `null` |
| 是否可為 null | 否（除非 `T?`） | 可 |
| 比較行為 | 值比較 | 引用比較（除非覆寫 `Equals`） |

# 裝箱 (Boxing) 與拆箱 (Unboxing)

## 裝箱：值類型 → 引用類型

## 拆箱：引用類型 → 值類型

## 效能考量

# 內建型別總覽

## 整數型別
`sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`

## 浮點型別
`float`、`double`、`decimal`

## 邏輯型別
`bool`

## 字元與字串
`char`、`string`

## 物件根型別
`object`

# 特殊型別關鍵字

## `var`（隱式型別）

## `dynamic`（執行期決定型別）

## `object`（所有型別的基底）

## `var` vs `dynamic` vs `object` 比較

# 型別轉換

## 隱式轉換 (Implicit)

## 顯式轉換 (Explicit / Cast)

## `Convert` 類別

## `Parse` / `TryParse`

# 相關概念
- [[C sharp variables and constants]]
- [[C sharp struct and record]]
- [[C sharp class and object]]
- [[C sharp nullable types]]
