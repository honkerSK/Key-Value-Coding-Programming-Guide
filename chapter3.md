> Key-Value Coding Programming Guide 官方文档第二部分第2节

[iOS-KVC官方文档第二部分第2节](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/AccessingCollectionProperties.html#//apple_ref/doc/uid/10000107i-CH4-SW1)

# Key-Value Coding Fundamatals--Accessing Collection Properties

## 访问集合属性

键值编码兼容对象以与公开其他属性相同的方式公开其对多属性。您可以像使用`valueForKey:`和`setValue:forKey:` (或它们的键路径等同方法) 一样获取或设置集合对象。但是, 当您要操作这些集合的内容时, 使用协议定义的可变代理方法通常是最有效的。

该协议为集合对象访问定义了三种不同的代理方法, 每个都具有一个键和一个键路径变体方法:

*   `mutableArrayValueForKey:`和`mutableArrayValueForKeyPath:`

    这些方法返回的代理对象的行为类似于`NSMutableArray`对象。

*   `mutableSetValueForKey:`和`mutableSetValueForKeyPath:`

    这些方法返回的代理对象的行为类似于`NSMutableSet`对象。

*   `mutableOrderedSetValueForKey:`和`mutableOrderedSetValueForKeyPath:`

    这些方法返回的代理对象的行为类似于`NSMutableOrderedSet`对象。

当您对代理对象进行操作、向其添加对象、从其中移除对象或替换其中的对象时, 协议的默认实现将相应地修改基础属性。这比使用`valueForKey:`得到一个不可变集合对象，创建一个修改了内容的可变集合对象, 然后将其存储回带有`setValue:forKey:`消息的对象更有效。在许多情况下, 它也比直接使用可变属性更有效。这些方法提供了对集合对象中保存的对象保持键值观察遵从性的额外好处 (请参见*[Key-Value Observing Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)*以了解详细信息）。

 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。