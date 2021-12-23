# is null or == null

判空`x == null`是最常见的代码之一，从C#7.0开始，可以用模式匹配`x is null`来判空，和`x == null`相比，最主要的区别是：

`x == null`受`==`操作符重载影响，所以当不知道代码具体实现时，结果就并不一定和预期相符，而`x is null`则不受此影响。

```c#
public static void Main()
{
    A x = new();
    System.Console.WriteLine(x is null);
    System.Console.WriteLine(x == null);
}
private class A
{
    public static bool operator ==(A a, A b)
    {
        return true;
    }
    public static bool operator !=(A a, A b)
    {
        return !(a == b);
    }
}
```
对比编辑器优化后的代码：
```c#
private class A
{
    public static bool operator ==(A a, A b)
    {
        return true;
    }

    public static bool operator !=(A a, A b)
    {
        return !(a == b);
    }
}
public static void Main()
{
    A x = new A();
    // x is null 变成了 (object)x == null
    Console.WriteLine((object)x == null);
    Console.WriteLine(x == null);
}
```
可以看到`x == null`变成了`(object)x == null`，所以实际上调用了`Object`的`==`来进行比较，尽管`class A`重载了`==`操作符。

可以进一步看`IL`代码：
```
IL_0000: newobj instance void CSharp/A::.ctor()

IL_0005: dup
IL_0006: ldnull
IL_0007: ceq
IL_0009: call void [System.Console]System.Console::WriteLine(bool)

IL_000e: ldnull
IL_000f: call bool CSharp/A::op_Equality(class CSharp/A, class CSharp/A)
IL_0014: call void [System.Console]System.Console::WriteLine(bool)

IL_0019: ret
```
`x is null`直接使用`ceq`指令来判断，所以不被任何函数影响。
> Compares two values. If they are equal, the integer value 1 (int32) is pushed onto the evaluation stack; otherwise 0 (int32) is pushed onto the evaluation stack.

所以`x == null`和`(object)x == null`完全等价，其实还有两种写法也与其等价：`x as object == null`和`x is not object`。

有趣的是，通常编辑器自带的语法检查功能，当遇到以上等价写法时，只要不是`x is null`的形式，将会一直提示改成`x is null`，显然这也是最容易书写和最接近自然语言的格式。

> `is`操作符右参数必须是编译常量，所以`static`变量将编译不通过。

> 模式匹配的语法决定了要和`null`比较的话，x必须是引用类型，否则存在编译错误，当x是泛型的时候就需要加一些约束条件才可以通过编译，如`where T : class`。

## 合理使用Equals

`Equals`方法也用来判断相等性，也是可重载的，而`object`本身还有一个`Equals`静态方法，先看看两者的差异。
```c#
System.Console.WriteLine(x.Equals(null));
System.Console.WriteLine(Equals(x, null));
```
对应的`IL`代码：
```c#
IL_001a: dup
IL_001b: ldnull
IL_001c: callvirt instance bool [System.Private.CoreLib]System.Object::Equals(object)
IL_0021: call void [System.Console]System.Console::WriteLine(bool)

IL_0026: ldnull
IL_0027: call bool [System.Private.CoreLib]System.Object::Equals(object, object)
IL_002c: call void [System.Console]System.Console::WriteLine(bool)
```
差别仅在于`callvirt`和`call`，`callvirt`需要查虚函数表，性能上肯定不如直接`call`。

所以如果自定义类不需要重载`Equals`的时候，使用`object`的静态方法`Equals`更合适。

*不要用`Equals`来判断两个值类型，除非迫不得已，因为存在装箱操作。*

*永远不要用`ReferenceEquals`来判断两个值类型，即使是两个常量，因为将永远得到错误答案。*
```c#
var i1 = 1;
var i2 = i1;
System.Console.WriteLine(Equals(i1, i2));
System.Console.WriteLine(ReferenceEquals(i1, i2));
System.Console.WriteLine(ReferenceEquals(1, 1));
```
编译器优化后：
```c#
int num;
int num2 = (num = 1);
Console.WriteLine(num == 1);
Console.WriteLine(num2 == num);
Console.WriteLine(object.Equals(num2, num));
Console.WriteLine((object)num2 == (object)num);
Console.WriteLine((object)1 == (object)1);
```
再来看看`IL`代码：
```
IL_0031: ldc.i4.1

IL_0032: dup
IL_0033: stloc.0

IL_0034: ldloc.0
IL_0035: ldc.i4.1
IL_0036: ceq
IL_0038: call void [System.Console]System.Console::WriteLine(bool)

IL_003d: dup
IL_003e: ldloc.0
IL_003f: ceq
IL_0041: call void [System.Console]System.Console::WriteLine(bool)

IL_0046: dup
IL_0047: box [System.Private.CoreLib]System.Int32
IL_004c: ldloc.0
IL_004d: box [System.Private.CoreLib]System.Int32
IL_0052: call bool [System.Private.CoreLib]System.Object::Equals(object, object)
IL_0057: call void [System.Console]System.Console::WriteLine(bool)

IL_005c: box [System.Private.CoreLib]System.Int32
IL_0061: ldloc.0
IL_0062: box [System.Private.CoreLib]System.Int32
IL_0067: ceq
IL_0069: call void [System.Console]System.Console::WriteLine(bool)

IL_006e: ldc.i4.1
IL_006f: box [System.Private.CoreLib]System.Int32
IL_0074: ldc.i4.1
IL_0075: box [System.Private.CoreLib]System.Int32
IL_007a: ceq
IL_007c: call void [System.Console]System.Console::WriteLine(bool)
```
