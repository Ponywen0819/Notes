#note

# 非同步程式設計概念

## 同步 vs 非同步

## 為什麼需要 async

## I/O bound vs CPU bound

# `Task` 與 `Task<T>`

## 建立 `Task`

## `Task.Run`

## `Task.WhenAll` / `Task.WhenAny`

## `Task.Delay`

# `async` / `await` 語法

## `async` 修飾子

## `await` 運算子

## 返回值類型 (`Task` / `Task<T>` / `void`)

## `async void` 為何要避免

# 同步上下文 (SynchronizationContext)

## `ConfigureAwait(false)`

## UI 執行緒 vs 背景執行緒

# 取消權杖 (CancellationToken)

## `CancellationTokenSource`

## 在 async 方法中傳遞 token

# 例外處理

## `try/catch` 與 `await`

## `AggregateException`

# 進階主題

## `IAsyncEnumerable<T>` / `await foreach`

## `ValueTask` / `ValueTask<T>`

# 常見陷阱

## Deadlock（`.Result` / `.Wait()`）

## Async all the way
