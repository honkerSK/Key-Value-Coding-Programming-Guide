> Key-Value Coding Programming Guide 官方文档第二部分第6节
> 2018.9.20 第一次修正 

[iOS-KVC官方文档第二部分第6节](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/SearchImplementation.html#//apple_ref/doc/uid/20000955-CJBBBFFA)

# Key-Value Coding Fundamatals--Accessor Search Patterns

##访问器搜索方式

NSObject默认实现NSKeyValueCoding协议提供的基于键的访问器，使用一组明确定义的规则来调用对象的基础属性。这些协议方法使用键参数在其自己的对象实例中搜索访问器，实例变量以及遵循某些命名约定的相关方法。尽管您很少修改此默认搜索， 但了解它的工作方式会有所帮助，对于跟踪键值编码对象的行为，也可以使您自己的对象兼容。

>注意
>本节中的描述使用`<key>`或`<Key>`作为键字符串的占位符，该键字符串在一个键值编码协议方法中作为参数出现，然后该方法将该键字符串用作间接方法调用或变量名称查找的一部分。映射的属性名称遵循占位符大小写的情况。例如，对于getter`<key>`和`is<KEY >`，名为 `hidden` 的属性映射为`hidden`和`isHidden` .。

### 基本的Getter搜索模式
`valueForKey：`的默认实现，给定一个`key`参数作为输入，在接收`valueForKey：`调用的类实例中操作，执行以下过程。

1.搜索实例与名称，按照该顺序，搜索找到的名称为`get<Key>`、`<key>`、`is<Key>`或`<key>`的第一个访问器方法。如果找到，调用它并继续到步骤5。否则请继续执行下一步。 

2.如果找不到简单取值方法, 则在实例中搜索其方法名形如 `countOf<Key>`和`objectIn<Key>AtIndex:` （相当于`NSArray`类中定义的基本方法）和`<key>AtIndexes`:（相当于`NSArray`类中的`objectsAtIndexes:`方法）的方法。

如果第一个方法和后边两个方法中的至少一个方法被实现了, 则创建一个能够响应所有`NSArray`方法并返回该方法的集合代理对象。否则, 继续执行步骤3。

代理对象随后将它接收的任何NSArray消息转换为`countOf <Key>`，`objectIn <Key> AtIndex：`和`<key> AtIndexes：`的消息，并将其发送给创建它的键值编码兼容对象。如果原始对象还实现了一个名为`get <Key>：range：`的可选方法，则代理对象也会在适当时使用该方法。实际上，与符合键值编码的对象一起工作的代理对象允许底层属性的行为就像它是一样`NSArray`，即使它不是。

3.如果没有找到简单的访问器方法或数组访问方法组，请查找名为`countOf <Key>`，`enumeratorOf <Key>`和`memberOf <Key>：`的方法的三个方法（对应于`NSSet`类定义的原始方法）。

如果找到所有三个方法，请创建一个响应所有`NSSet`方法并返回该方法的集合代理对象。 否则，请继续执行步骤4。

这个代理对象随后将它接收的任何`NSSet`消息转换为`count Of <Key>`，`enumeration of <Key>`和`member Of <Key>：`消息到创建它的对象。实际上，与符合键值编码的对象一起工作的代理对象允许底层属性的行为就像它是一样`NSSet`，即使它不是。


4.如果找不到简单访问器方法或集合访问方法组, 并且消息接收者的类方法`accessInstanceVariablesDirectly`返回YES, 则系统按以下顺序搜索名为:`_<key>`、 `_is<Key>`、 `<key>`或`is<Key>`的实例变量。如果找到, 则直接获取实例变量的值, 然后继续执行步骤5。否则, 继续跳转到步骤6。

5.如果获取到的属性值是对象指针,即获取的是对象, 则直接将对象返回。

如果获取到的属性值是NSNumber支持的数据类型, 则将其存储在NSNumber实例并返回。
如果获取到的属性值不是 NSNumber 支持的类型, 则转换为NSValue对象, 然后返回。

6.如果上述所有方法都没有执行，则调用`valueForUndefinedKey：`。 默认情况下，这会引发异常，但NSObject的子类可以通过重载并根据特定key做一些特殊处理。

---

### 基本的Setter搜索方式
`setValue:forKey:`的默认实现, 给定`key`和`value`参数作为输入, 在接收调用的对象内尝试将名为key的属性设置为value(或者, 对于非对象属性, 则为unwarp value, 详见[Representing Non-Object Values](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/DataTypes.html#//apple_ref/doc/uid/20002171-BAJEAIEE)), 使用以下过程:

+ 1.按照顺序查找第一个名为`set<Key>:`或`_set<Key>`的方法。如果找到, 传入输入值 (或根据需要展开值) 调用它, 然后完成。

+ 2.如果找不到简单访问器, 并且类方法`accessInstanceVariablesDirectly`返回YES, 则按以下顺序查找实例变量: `_<key>`、 `_is<Key>`、` <key>`、`is<Key>` 。如果找到, 则直接使用输入值 (或展开值) 设置变量并完成。

+ 3.如果找不到以上方法或实例变量, 则调用`setValue:forUndefinedKey:`。默认情况下，这会引发异常，但`NSObject`的子类可以通过重载并根据特定key做一些特殊处理。

---

### 可变数组的搜索方式
`mutableArrayValueForKey：`的默认实现，给定一个`key`参数作为输入，使用以下过程为接收访问者调用的对象内的一个名为`key`的属性返回一个可变代理数组：

1.查找一对方法, 名为`insertObject：in <Key> AtIndex：`和`removeObjectFrom <Key> AtIndex：`的方法（分别对应于`NSMutableArray`原始方法`insertObject：atIndex：`和`removeObjectAtIndex：`） ，或者名称类似于`insert <Key>：atIndexes：`和`remove <Key> AtIndexes：`（对应于`NSMutableArrayinsertObjects：atIndexes：`和`removeObjectsAtIndexes：`方法）。

如果对象至少实现一个插入方法和至少一个删除方法，则返回一个响应`NSMutableArray`消息的代理对象，方法是发送`insertObject：in <Key> AtIndex：`，`removeObjectFrom <Key> AtIndex： `，`insert <Key>：atIndexes：`，和`remove <Key> AtIndexes：`消息到`mutableArrayValueForKey：`的原始接收者。

当接收`mutableArrayValueForKey：`消息的对象也实现了一个可选的替换对象方法，其名称如`replaceObjectIn <Key> AtIndex：withObject：`或`replace <Key> AtIndexes：with <Key>：`，代理对象在适合最佳性能时会自动调用该可选方法。

2.如果对象没有可变数组方法，则查找名称与模式`set <Key>：`匹配的访问器方法。 在这种情况下，通过向`mutableArrayValueForKey：`的原始接收者发出`set <Key>：`消息，返回响应NSMutableArray消息的代理对象。

>注意:
>此步骤中描述的机制比上一步的效率要低得多, 因为它可能涉及重复创建新的集合对象, 而不是修改现有的。因此, 在设计自己的键值编码兼容对象时, 通常应避免这种情况。

3.如果既没有找到可变数组方法，也没有找到访问器，并且接收者的类对`accessInstanceVariablesDirectly`响应'YES`，则按照顺序搜索名为`_ <key>`或`<key>`的实例变量。

如果找到这样的实例变量，则返回一个代理对象，该对象将它接收的每个`NSMutableArray`消息转发给实例变量的值，该值通常是`NSMutableArray`的实例或其子类之一。

4.如果所有其他方法都失败了，只要收到`NSMutableArray`消息，就返回一个可变集合代理对象，该对象向`mutableArrayValueForKey：`消息的原始接收者发出`setValue：forUndefinedKey：`消息。

setValue:forUndefinedKey:的默认实现会抛出NSUndefinedKeyException异常, 但子类可能会重写此行为。

---

### 可变有序集的搜索方式

mutableOrderedSetValueForKey的默认实现：将相同的简单访问器方法和有序集访问器方法识别为`valueForKey :`(请参阅 [Default Search Pattern for the Basic Getter](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/SearchImplementation.html#//apple_ref/doc/uid/20000955-138234)
），并遵循相同的直接访问实例变量策略，但始终返回可变集合代理对象 `valueForKey：`返回的不可变集合。 此外，它还执行以下操作：

1.搜索名称类似于以下形式的方法: `insertObject：in <Key> AtIndex：`和`removeObjectFrom <Key> AtIndex：`（对应于`NSMutableOrderedSet`类定义的两个最原始方法），以及`insert <Key>： atIndexes：`和`remove <Key> AtIndexes：`（对应于`insertObjects：atIndexes：`和`removeObjectsAtIndexes：`）。

如果找到至少一个insert方法和至少一个remove方法，返回的代理对象每次接收到 NSMutableOrderedSet的消息后， 会通过以下组合方法给`mutableOrderedSetValueForKey`的原始对象 发送消息: `mutableOrderedSetValueForKey: `  `insertObject:in<Key>AtIndex:`, `removeObjectFrom <Key> AtIndex :`, `insert <Key>：atIndexes：`  `remove<Key>AtIndexes: ` 

代理对象还使用方法名为 `replaceObjectIn <Key> AtIndex：withObject：` 的方法，或`replace<Key>AtIndexes:with<Key>: ` ,当这些方法存在于原始对象中时。

2.如果找不到可变的set方法，请搜索名为`set <Key>：`的访问器方法。 在这种情况下，返回的代理对象每次收到`NSMutableOrderedSet`消息时都会向`mutableOrderedSetValueForKey：`的原始接收者发送一个`set <Key>：`消息。

>注意
>此步骤中描述的机制比前一步骤的效率低得多，因为它可能涉及重复创建新的集合对象而不是修改现有的集合对象。 因此，在设计自己的符合键值编码的对象时，通常应该避免使用它。

3.如果找不到可变集消息和访问器，并且接收者的`accessInstanceVariablesDirectly`类方法返回`YES`，则按顺序搜索名称如`_ <key>`或`<key>`的实例变量。 如果找到这样的实例变量，则返回的代理对象将它接收的任何`NSMutableOrderedSet`消息转发给实例变量的值，该值通常是`NSMutableOrderedSet`或其子类之一的实例。

4.如果所有其他方法都失败了，那么只要收到一个可变的set消息，返回的代理对象就会向`mutableOrderedSetValueForKey：`的原始接收者发送一个`setValue：forUndefinedKey：`消息。

`setValue：forUndefinedKey：`的默认实现引发了一个`NSUndefinedKeyException`，但是对象可能会覆盖此行为。

---

### 可变集的搜索方式

`mutableSetValueForKey：`的默认实现，给定一个`key`参数作为输入，使用以下过程为接收访问者调用的对象内的一个名为`key`的数组属性返回一个可变代理集：

1.搜索方法名称为`add<Key>Object:`和 `remove <Key> Object` 的方法：（分别对应于`NSMutableSet`原始方法`addObject:`和`removeObject:`）以及`add<Key>：`和`remove<Key>: `（对应于`NSMutableSet` 方法`unionSet:`和`minusSet:` ）。如果找到至少一个添加方法和至少一个删除方法，则返回一个NSMutableSet代理对象，该代理对象发送`add <Key> Object：`，`remove <Key> Object：`，`add <Key>：`的某种组合， 和`remove <Key>：`对于它收到的每个`NSMutableSet`消息的`mutableSetValueForKey：`的原始接收者的消息。

代理对象还使用名称为“cross<Key>：”或“set<Key>：”的方法来提高性能（如果可用的话）。

2.如果`mutableSetValueForKey：`调用的接收者是托管对象，则搜索模式不会像非托管对象那样继续。 有关更多信息，请参阅“*[Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html#//apple_ref/doc/uid/TP40001075)*
”中的托管对象访问器方法。

3.如果找不到可变集合方法，并且对象不是托管对象，则搜索名为`set <Key>：`的访问器方法。 如果找到这样的方法，则返回的代理对象将`set <Key>：`消息发送给它接收的每个NSMutableSet消息的`mutableSetValueForKey：`的原始接收者。

> 注意
> 此步骤中描述的机制的效率远低于第一步的机制，因为它可能涉及重复创建新的集合对象而不是修改现有的集合对象。 因此，在设计自己的符合键值编码的对象时，通常应该避免使用它。

4.如果找不到可变的set方法和accessor方法，并且`accessInstanceVariablesDirectly`类方法返回`YES`，则按照顺序搜索名为`_ <key>`或`<key>`的实例变量。 如果找到这样的实例变量，则代理对象将它接收的每个`NSMutableSet`消息转发给实例变量的值，该值通常是`NSMutableSet`的实例或其子类之一。

5.如果所有其他方法都失败了，返回的代理对象通过向`mutableSetValueForKey：'的原始接收者发送`setValue：forUndefinedKey：`消息来响应它收到的任何`NSMutableSet`消息。

 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。




