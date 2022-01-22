# sealed class

参考[Why To Use the sealed Keyword on Classes?](https://www.codeproject.com/Articles/239939/Csharp-Tweaks-Why-To-Use-the-sealed-Keyword-on-Cla)

以下代码没有编译错误，但会产生运行时错误：
```C#
interface I { }

class C { }

static void Main()
{
    C c = new();
    // ok
    I i = (I)c;
}
```

因为编译器判断class`C`的派生类可能实现了`I`，所以编译器检查是通过的。

如果给class`C`加上`sealed`则可以提前暴漏问题：
```C#
interface I { }

sealed class C { }

static void Main()
{
    C c = new();
    // 无法将类型“C”转换为“I” [csharp]csharp(CS0030)
    I i = (I)c;
}
```
