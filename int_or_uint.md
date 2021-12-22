# int or uint（有符号 or 无符号）

> 本文旨在讨论当我们在设计数据结构或者创建变量的时候，要表示一个数字，用`int`还是`uint`，接下来将对两者进行一些对比，并给出建议的适用场景。

* `int`能表示负数，而`uint`能表示比`int`更大的正数，超过`Math.Pow(2, 31) - 1`。
* 两者都不存在默认的相互隐式转换规则，相互转换都可能存在精度丢失问题，即`(int)uintValue`和`(uint)intValue`都可能和预期不符。
* 两者都存在计算结果溢出的情况，不过对于一般使用到的数值大小而言，`int`可表示的范围已经足够进行计算，但`uint`一旦做减法就很可能小于0导致溢出，从这一点来说，对于需要计算的场景，`int`更合适。
    > 以欢乐豆为例，虽然一个玩家拥有的欢乐豆数量必定是个正数，考虑到可能需要计算结算前后的欢乐豆差值，次值可正可负，为避免溢出，将拥有的欢乐豆定义为`int`比`uint`更合适。

    > 和0比大小的场景，使用`uint`也不合适，因为`uint`始终是>=0的。
* CLS不支持`uint`，在设计公共API的时候应尽量避免使用`uint`。
    > 只有C#代码的项目不受影响。

    > 另外java、javascript等语言也没有无符号数。
* `int`更常见，即便在一些通常看起来不可能出现负数的场景，语言默认实现也是`int`。
    * var类型推测
    ```c#
    var i = 1; // i is int
    var j = 2147483648; // j is uint
    ```
    * Arrat.Length、List.Count等
    * 索引器参数
    ```c#
    public class List<T>
    {
        public T this[int index] { get; set; }
        public int Count { get; }
    }
    ```
    * IndexOf返回值
        > 索引和-1，肯定得用`int`。
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
* 函数参数建议使用`int`。
    > 使用`int`便于进行参数校验，可以和0比大小。

    > 如索引器参数，如果使用`uint`，传入`(uint)负数`将可能返回意想不到的结果。
    ```c#
    var i = -1;
    var index = (uint)i;
    list[index]; // index == 4294967295
    ```
* 尽管`int`很普遍，不过也有一些场景使用`uint`更合适。
    * 用于表示id等数据，没有负数，不进行减法操作，不与0比大小。
        > 如uin、itemid等。
    * `enum`，且不会和`int`互相转化的情况。
