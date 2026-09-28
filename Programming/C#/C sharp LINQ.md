#note

# LINQ (Language Integrated Query) 概觀

## 為什麼需要 LINQ

## 兩種語法：查詢語法 vs 方法語法

# 查詢語法

```csharp
from x in source
where x > 0
select x
```

# 方法語法

`source.Where(x => x > 0).Select(x => x)`

# 常用運算子

## 篩選：`Where`

## 投影：`Select`、`SelectMany`

## 排序：`OrderBy`、`ThenBy`、`OrderByDescending`

## 分組：`GroupBy`

## 連接：`Join`、`GroupJoin`

## 集合：`Union`、`Intersect`、`Except`、`Distinct`

## 聚合：`Count`、`Sum`、`Average`、`Min`、`Max`、`Aggregate`

## 元素：`First`、`FirstOrDefault`、`Single`、`ElementAt`

## 量詞：`Any`、`All`、`Contains`

## 分頁：`Skip`、`Take`、`SkipWhile`、`TakeWhile`

## 轉換：`ToList`、`ToArray`、`ToDictionary`、`Cast`、`OfType`

# 延遲執行 (Deferred Execution)

## 概念與時機

## 立即執行的觸發

# `IEnumerable<T>` vs `IQueryable<T>`

# LINQ to Objects vs LINQ to SQL/EF

# 自訂 LINQ 運算子（擴充方法）
