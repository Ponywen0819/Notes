#note

# 例外 (Exception) 概念

## 例外 vs 錯誤碼

## `Exception` 類別階層

# `try` / `catch` / `finally`

## 基本語法

## 多個 `catch` 區塊

## 例外篩選器 `when`

## `finally` 的執行時機

# 拋出例外

## `throw new Exception(...)`

## 重新拋出 `throw;` vs `throw ex;`

## 例外鏈接 (Inner Exception)

# 常見內建例外
- `ArgumentException` / `ArgumentNullException`
- `InvalidOperationException`
- `NullReferenceException`
- `IndexOutOfRangeException`
- `FormatException`

# 自訂例外類別

# `using` 陳述句與 `IDisposable`

## 傳統 `using` 區塊

## `using` 宣告 (C# 8+)

# 最佳實務
- 不要吞掉例外
- 例外不應作為控制流程
- 何時需要 catch
