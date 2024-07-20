在 C# 專案中，會存在一個或多個[命名空間]，包含了所有使用者撰寫之內容，範例如下

```C#
// A skeleton of a C# program
using System;

// Your program starts here:
Console.WriteLine("Hello world!");

namespace YourNamespace
{
    class YourClass
    {
    }

    struct YourStruct
    {
    }

    interface IYourInterface
    {
    }

    delegate int YourDelegate();

    enum YourEnum
    {
    }

    namespace YourNestedNamespace
    {
        struct YourStruct
        {
        }
    }
}
```

而程式之進入點可以分成兩種方法
- 最上層陳述式 ( Top-level statements )
- Main 靜態方法

## 最上層陳述句

為最簡單的執行方式，不需要定義任何命名空間，範例如下
```C#
using System;

Console.WriteLine("Hello World!");
```

最上層陳述句有幾點特點
- 任何專案中只能有單一檔案使用最上層陳述句
- 隱含的全局命名空間
- 依舊能夠定義狀態與命名空間
- 可以取用 `args` 參數

### 單一檔案最上層陳述句
在 C# 專案中，只能存在單一進入點，而最上層陳述句隱含的表示進入點，因此每份專案中只能有一份檔案使用最上層陳述句。

### 隱含的全局命名空間
在使用最上層陳述句的檔案中所做的定義都屬於全局的命名空間。

### 可以取用 `args` 參數

可以透過 `args` 參數取的使用者透過 cmd 輸入之參數，範例如下
```C#
if (args.Length > 0)
{
    foreach (var arg in args)
    {
        Console.WriteLine($"Argument={arg}");
    }
}
else
{
    Console.WriteLine("No arguments");
}
```

## Main 靜態函數
`main` 函式定義在命名空間內，並且同一專案中只能有單一型別擁有 main 函式，範例如下
```C#
class TestClass
{
    static void Main(string[] args)
    {
        // Display the number of command line arguments.
        Console.WriteLine(args.Length);
    }
}
```

