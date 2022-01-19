# interface or abstract class

先说结论：
* C#8.0之前，只有多继承（实现）才用`interface`，否则使用`abstract class`。
* C#8.0之后，只有需要实例`Field`成员才用`abstract class`，否则使用`interface`。

为什么以C#8.0为分界，因为8.0为`interface`增加了很多能力，足以和`abstract class`平起平坐，甚至更甚一筹。

## 详细对比

`interface`和`abstract class`都是实现多态的方式之一，两者在作用上很像。

在C#8.0之前，除了多继承（实现）的优势外，`interface`几乎只能被`abstract class`按在地上摩擦，使用起来非常不便。

以下是两者的对比：

| | abstract class | interface |
| :- | :- | :- |
| 多继承（实现） | 不支持 | 支持 |
| Constructor/Destructor | 支持 | 不支持 |
| 修饰符 | 支持所有 | 只支持public |
| Field | 支持 | 不支持 |
| Method、Property、Indexer、Event、Operator | 支持 | 只支持Method和Property |
| 方法实现 | 支持 | 不支持 |
| 内联类型 | 支持 | 不支持 |

从上表可以看出，`interface`只是作为一种类设计准则，即描述`class`需要提供什么样的方法，没有任何其他的功能。
