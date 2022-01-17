# generic or specific

> Return the most specific type, accept the most generic type.

本文从类和集合两方面来说明何时使用具体类型，何时使用宽泛类型。

对类而言，派生类是具体类型，基类是宽泛类型。

对集合而言，如果`List`是具体类型，则`IList`是宽泛类型，如果`IList`是具体类型，则`IEnumerable`是宽泛类型，总之，越基础的类型越是宽泛类型。

## 类

当类类型作为方法参数的时候，应尽可能使用基类类型，作为返回值的时候，尽可能返回派生类型。

方法参数使用基类，是为了让方法更通用，这样可以接受任何派生类作为实参，因为派生类转为基类总是可行的，而反之则可能失败。

同理返回具体的派生类也可以在需要的时候转为基类使用，同时派生类能提供更多的能力。

## 集合（接口）

> 以最常用的`List`和`Dictionary`来说明。

从继承（实现）链可以得出`List : IList : ICollection : IEnumerable`这样的关系。

作为方法参数，当使用`List`的时候，应该认真审视是否`IList`就已足够，是否`ICollection`就已足够，甚至是否`IEnumerable`就已足够。

如果只需要遍历功能，使用`IEnumerable`更合适，好处是任意集合类型都可以作为参数。

如果还需要`Count`、`Add`、`Remove`等方法，使用`ICollection`更合适，可同时支持`List`、`Collection`等类型。

如果还需要`IndexOf`、`RemoveAt`等方法，使用`IList`更合适，也同时支持`List`和`Collection`。

如果确实使用了`List`独有的方法，那么才使用`List`，也意味着方法只为`List`专用。

同样的，对于`Dictionary`有`Dictionary : IDictionary : ICollection : IEnumerable`，当使用`Dictionary`的时候，是否更基本的基类就已足够。

作为返回值，跟类类型同理。

## 总结

越基本的类型，越能接受更多派生类型的参数。

派生类型提供了更多的能力，但更具体的类型意味着更强的限制，带来了更弱的通用性和可扩展性。

为`public`方法和公共API选择尽量基本的类型，以便将来的兼容和扩展。
