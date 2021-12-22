# int or uint（有符号 or 无符号）

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
