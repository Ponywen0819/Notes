#note

# Attribute（屬性標記）

## 概念與用途

## 套用語法 `[AttributeName]`

## 多重 attribute

## 帶參數的 attribute

# 常見內建 Attribute
- `[Obsolete]`
- `[Serializable]`
- `[Conditional]`
- `[DllImport]`
- `[AttributeUsage]`

# 自訂 Attribute

## 繼承 `Attribute` 類別

## `[AttributeUsage]` 控制適用範圍

# 反射 (Reflection)

## `typeof` 與 `Type` 物件

## `GetType()` 方法

## 取得型別資訊
- `GetMethods` / `GetProperties` / `GetFields`
- `GetCustomAttributes`

## 動態呼叫成員

## `Activator.CreateInstance`

# 反射效能考量

# 反射典型應用
- 序列化 / 反序列化
- 依賴注入容器
- ORM
- 單元測試框架

# Source Generator（編譯期反射的替代方案）
