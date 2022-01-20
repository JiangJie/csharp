# interface or abstract class

先说建议：
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

不过随着C#8.0新加入的功能，`interface`已非吴下阿蒙，功能上完全向`abstract class`看齐。

再看对比：

| | abstract class | interface | interface#8.0 |
| :- | :- | :- | :- |
| 多继承（实现） | 不支持 | 支持 | 支持 |
| Constructor/Destructor | 支持 | 不支持 | 支持static |
| 修饰符 | 支持所有 | 只支持public | 支持所有 |
| Field | 支持 | 不支持 | 支持static |
| Method、Property、Indexer、Event、Operator | 支持 | 只支持Method和Property | 支持 |
| 方法实现 | 支持 | 不支持 | 支持 |
| 内联类型 | 支持 | 不支持 | 支持 |

8.0的`interface`除了实例相关的成员，其他都支持了，其根本原因还是因为`interface`无法实例化，最关键支持了方法默认实现，从此不必再为多个`class`写同样的实现代码了。

引用官方对于扩展`interface`能力的动机：
> The principal motivations for this feature are

> Default interface methods enable an API author to add methods to an interface in future versions without breaking source or binary compatibility with existing implementations of that interface.

> The feature enables C# to interoperate with APIs targeting Android (Java) and iOS (Swift), which support similar features.

> As it turns out, adding default interface implementations provides the elements of the "traits" language feature (https://en.wikipedia.org/wiki/Trait_(computer_programming)). Traits have proven to be a powerful programming technique (http://scg.unibe.ch/archive/papers/Scha03aTraits.pdf).

增加方法默认实现，是为了解决`interface`更新导致的不兼容问题，比如增加一个`Method`，需要所有实现的`class`都改一遍，这样的升级是破坏性的。

以前的做法只能新建一个包含新`Method`的`interface`来代替，现在可以直接增加默认实现，而不用改实现类了。
