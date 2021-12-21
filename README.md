# csharp

C#学习心得

## C#相关

### int or uint（有符号 or 无符号）

> 当我们在设计数据结构的时候，要表示一个数字，用int还是uint

* int能表示负数，uint能表示更大的正数，超过`Math.Pow(2, 31) - 1`
* 两者都不存在默认的隐式转换规则，相互转换都可能存在精度丢失的问题
* 都存在计算结果溢出的情况，对于一般使用到的数值大小来说，int可表示的范围足够进行计算，但uint一旦做减法就可能小于0导致溢出，从这一点来说，对于需要计算的场景，int更合适
    > 以欢乐豆为例，虽然一个玩家拥有的欢乐豆数量必定是个正数，但可能需要计算结算前后的欢乐豆差值，为避免移除，将欢乐豆定义为int比uint更合适

    > 另外注意和0比大小的场景，使用uint也不合适
* CLS不支持uint，在设计公共API的时候使用uint就不太合适
    > 另外java、javascript等也没有无符号数
* int更常见，即便在一些通常看来不可能出现负数的场景
    * var
    ```c#
    var i = 1; // i is int
    var j = 2147483648; // j is uint
    ```
    * Arrat.Length、List.Count等
    * 索引器
    ```c#
    public class List<T>
    {
        public T this[int index] { get; set; }
        public int Count { get; }
    }
    ```
    * IndexOf
        > 本来返回的索引肯定是正数，但还存在-1的情况，所以返回值是int
    * for循环
    ```c#
    for (int i = 0; i < length; i++) // 更常见
    for (uint i = 0; i < length; i++) // 很少见
    for (uint i = length; i >= 0; i--) // 小心死循环
    ```
    * (s)byte、(u)short的数学运算
    ```c#
    byte a = 1;
    byte b = 1;
    var c = a + b; // c is int
    var d = a * b; // c is int
    ```
* 方法参数建议使用int
    > 使用int便于进行参数校验

    > 如索引器，如果使用uint，将可能返回意想不到的结果
    ```c#
    var i = -1;
    var index = (uint)i;
    list[index]; // 4294967295
    ```
* 一些场景使用uint更合适
    * 用于表示id，没有负数，不进行减法操作，不与0比大小
        > 如uin、itemid等
    * enum，且不会和int互相转化的情况

### is null or == null

判空是最常见的代码之一，从C#7.0开始，可以用模式匹配`x is null`来判空，和`x == null`相比，最主要的区别是：
`x == null`受`==`操作符重载影响，所以并不一定总是和预期相符，而`x is null`则不受此影响

> 模式匹配的语法决定了x必须是引用类型，否则存在编译错误，当变量是泛型的时候就需要加一些约束条件才可以，比如`where T : class`

### lambda expression or local function

> At first glance, local functions and lambda expressions are very similar. In many cases, the choice between using lambda expressions and local functions is a matter of style and personal preference.

```c#
public static void Main()
{
    int i = 0;
    // lambda expression
    var add = (int j) => i += j;
    // local function
    void addFunc(int j) => i += j;
}
```

多数场景下，两者确实用谁都可以，在调用次数不大的情况下，性能差别也几乎可以忽略，但`local function`作为`lambda expression`的增强版，具有一些`lambda expression`不具备的能力，意味着在某些场景下，只能选择使用`local function`

* yield return
    > `lambda expression`不支持`yield return`，而`local function`支持，意味着`local function`可以实现为迭代器

    > `lambda expression`无法推断出类型
* 泛型
    > 因为`lambda expression`是匿名函数，不支持泛型，而`local function`支持
    ```c#
    string stringify<T>(T x) => x.ToString();
    ```
* 递归
    > 在看递归之前先看一个例子
    ```c#
    int i;
    // error: 使用了未赋值的局部变量“i” [csharp]csharp(CS0165)
    var add = (int j) => i += j;
    ```

    > `lambda expression`虽然代码跟函数体类似，但其本质还是表达式，函数在声明的时候并不检查函数体，而表达式则不同
    ```c#
    // error: 本地变量“f”在声明之前无法使用 [csharp]csharp(CS0841)
    var f = (int n) => n == 1 ? n : n * f(n - 1);
    ```

    > 我们在使用`lambda expression`的时候，其实绝大多数时候是通过创建`delegate`来使用，这一点容易造成两者混淆

`lambda expression`和`local function`还有一个最大的区别在于捕获变量的内存分配方式上

`lambda expression`因为可以作为函数参数或者返回值（`First-class citizen`），因此其生命周期可以相当长，并不受闭包函数控制

为了可以随时访问到捕获变量，很显然需要将捕获变量分配到堆上，即使捕获变量是闭包函数的临时变量
```c#
public void Main()
{
    int i = 1;
    var add = (int j) => i += j;
}
```
反编译后的代码，可以看到通过`class`来保存捕获变量`i`，内存分配在堆上而非栈上
```c#
[CompilerGenerated]
private sealed class <>c__DisplayClass0_0
{
    public int i;

    internal int <Main>b__0(int j)
    {
        return i += j;
    }
}

public void Main()
{
    new <>c__DisplayClass0_0().i = 1;
}
```

再来看看`local function`的分配方式
```c#
public void Main()
{
    int i = 1;
    void add(int j) => i += j;
}
```
反编译后
```c#
[StructLayout(LayoutKind.Auto)]
[CompilerGenerated]
private struct <>c__DisplayClass0_0
{
    public int i;
}

public void Main()
{
    int i = 1;
}

[CompilerGenerated]
private static void <Main>g__add|0_0(int j, ref <>c__DisplayClass0_0 P_1)
{
    P_1.i += j;
}
```
通过分配在栈上的`struct`来保存捕获变量，并且通过`ref`的方式来减少拷贝，这一点上`local function`比`lambda expression`有更好的性能

不过这一特性也带来了限制，`local function`捕获的变量随着闭包函数执行完便会释放，所以`local function`不是`First-class citizen`，无法作为函数参数和返回值

前文提到`local function`是`lambda expression`的增强版，怎么作为`First-class citizen`这一最强大的功能却没了呢？答案肯定是否定的
```c#
public Action<int> GetAdd()
{
    int i = 1;
    void add(int j) => i += j;
    return add;
}
```
实际上这时候`local function`已经被当做`delegate`了，从反编译的代码可以看出与`lambda expression`并无二致
```c#
[CompilerGenerated]
private sealed class <>c__DisplayClass0_0
{
    public int i;

    internal void <GetAdd>g__add|0(int j)
    {
        i += j;
    }
}

public Action<int> GetAdd()
{
    <>c__DisplayClass0_0 <>c__DisplayClass0_ = new <>c__DisplayClass0_0();
    <>c__DisplayClass0_.i = 1;
    return new Action<int>(<>c__DisplayClass0_.<GetAdd>g__add|0);
}
```
用作函数参数的时候同理，也是被当作`delegate`了

再看一个例子
```c#
public void Main()
{
    int i = 1;
    void add(int j) => i += j;
    var add2 = (int j) => i += j;
}
```
反编译
```c#
[CompilerGenerated]
private sealed class <>c__DisplayClass0_0
{
    public int i;

    internal void <Main>g__add|0(int j)
    {
        i += j;
    }

    internal int <Main>b__1(int j)
    {
        return i += j;
    }
}

public void Main()
{
    new <>c__DisplayClass0_0().i = 1;
}
```
所以当捕获的变量被其他`local function`或者`delegate`也捕获了，则将直接使用堆分配

总结起来，当`local function`所有捕获的变量（`static`除外）没有被其他`lambda expression`或者当做`delegate`的`local function`捕获时，则使用栈分配，否则使用堆分配，从性能上来说，相比`local function`就没了优势

* Array or List
* IEnumerable or List
* 何时用var
* object or dynamic
* interface or abstract class
* sealed class
* namespace
* for or foreach
* LINQ or method

## Unity UI Toolkit
