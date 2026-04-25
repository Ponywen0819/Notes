#note

# `string` 型別本質
- 不可變 (immutable)
- 引用類型，但行為類似值類型

# 建立字串

## 字面值 (literal)

## Verbatim string `@"..."`

## 字串內插 `$"..."`

## Raw string literals `"""...""""` (C# 11+)

# 常用方法
`Length`、`Substring`、`IndexOf`、`Replace`、`Split`、`Trim`、`ToUpper`/`ToLower`、`Contains`、`StartsWith`/`EndsWith`

# 字串比較
`==`、`Equals`、`Compare`、`StringComparison`

# 字串格式化
`string.Format`、`ToString` 格式說明符、文化資訊 (`CultureInfo`)

# `StringBuilder`
- 為什麼需要：可變字串避免大量 `+` 串接
- 常用方法：`Append`、`Insert`、`Remove`、`ToString`

# 字串編碼
`System.Text.Encoding`、UTF-8 / UTF-16 / ASCII

# 與 `char` 的關係
