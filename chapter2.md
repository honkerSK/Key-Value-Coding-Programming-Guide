> Key-Value Coding Programming Guide 官方文档第二部分第1节

[iOS-KVC官方文档第二部分第1节](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/BasicPrinciples.html#//apple_ref/doc/uid/20002170-BAJEAIEE)


# Key-Value Coding Fundamatals--Accessing Object Properties
> 键值编码基本原理-- 获取对象属性

## 访问对象属性

对象通常在其接口声明中指定属性, 这些属性属于以下几个类别之一:

*   **属性**. 一些简单的值, 例如标量(`scalars`)、字符串或布尔值。值对象(如`NSNumber`)和其他不可变类型(如`NSColor`) 也被视为属性。
*   **对一关系**. 指具有自身属性的可变对象。对象的属性可以在对象本身没有更改的情况下进行更改。
*   **对多关系**. 指集合对象。你通常使用`NSArray`或`NSSet`的实例来保存此类集合, 自定义集合类也是可行的。

清单 2-1中声明的`BankAccount`对象演示了每种类型的属性。

**清单 2-1**`BankAccount`对象的属性

```objc
@interface BankAccount : NSObject

@property (nonatomic) NSNumber* currentBalance;              // An attribute
@property (nonatomic) Person* owner;                         // A to-one relation
@property (nonatomic) NSArray<Transaction*>* transactions;   // A to-many relation

@end
```

为了维护封装, 对象通常为接口中的属性提供了访问器方法。对象的作者可以显式地编写这些方法, 也可以依赖编译器自动合成它们。无论哪种方式, 代码的作者使用这些访问器时都必须在编译代码之前将属性名写到代码中。访问器方法的名称成为使用它的代码的静态部分。例如, 给定清单 2-1中声明的银行帐户对象, 编译器将合成一个可为myAccount实例调用的 setter:

```objc
[myAccount setCurrentBalance:@(100.0)];
```

这很直接, 但缺乏灵活性。另一方面, 一个键值编码兼容对象提供了一个更通用的机制来使用字符串标识符访问对象的属性。

### 使用键和键路径标识对象的属性

键`key`是标识特定属性的字符串。通常, 按照约定, 表示属性的键`key`是属性本身在代码中显示的名称。键`key`必须使用 `ASCII` 编码, 不能包含空格, 通常以小写字母开头 (尽管有例外, 如在许多类中找到的`URL`属性)。

因为清单 2-1中的`BankAccount`类是符合键值编码的, 所以它能识别键(即其属性的名称)`owner`、`currentBalance`和`transactions` 。您可以通过其键来设置值, 而不是调用`setCurrentBalance:`方法:

```objc
[myAccount setValue:@(100.0) forKey:@"currentBalance"];
```

实际上, 可以使用不同的键参数通过相同方法来设置myAccount对象的所有属性。因为参数是字符串类型, 所以它可以在运行时操作变量。

键路径`Key path`是一个用点操作符`.`来分隔键的字符串, 用于指定要遍历的对象属性序列。序列中第一个键的属性相对于接收者, 每个后续键相对于上一个属性的值进行计算。键路径对于使用单个方法深入调用对象的层次结构很有用。

例如, 应用于银行帐户实例的键路径`owner.address.street`是指存储在银行帐户所有者地址中的街道字符串的值, 假设`Person`和`Address`类也符合的键值编码。

> 注意
> 在 Swift 中, 您可以使用`#keyPath`表达式, 而不是使用字符串来指示键或键路径。这提供了编译期间检查的优点, 详见*[Using Swift with Cocoa and Objective-C (Swift 4.2)](https://developer.apple.com/documentation/swift#2984801) *中的*[Keys and Key Paths](https://developer.apple.com/documentation/swift#2984801)*章节。

### 使用键获取属性值

当对象遵循 `NSKeyValueCoding`非正式协议，则该对象支持键值编码。从`NSObject`（提供了协议中基本方法的默认实现）继承的对象, 它会自动采用此协议的某些默认行为。这样的对象至少实现以下基本的`key-based`的 getter:

*   `valueForKey:` 返回由键参数命名的属性的值。如果根据访问器搜索模式中描述的规则无法找到由键命名的属性, 则该对象将自己发送`valueForUndefinedKey:`消息。`valueForUndefinedKey`的默认实现将抛出`NSUndefinedKeyException`异常, 但子类可以重写此行为并更优雅地处理此情况。

*   `valueForKeyPath:` 返回相对于接收者的指定键路径的值。键路径序列中指定键所对应的对象如果不是键值编码兼容的, 即`valueForKey:`的默认实现无法找到访问器方法-那么就会接收`valueForUndefinedKey:`消息。

*   `dictionaryWithValuesForKeys:` 返回相对于接收者的一组键所对应的值。该方法为数组中的每个键调用`valueForKey:` 。返回的`NSDictionary`包含数组中所有键的值。

> 注意
> 集合对象 (如`NSArray`、 `NSSet`和`NSDictionary`) 不能包含nil作为值。而是使用NSNull对象表示nil值。NSNull提供一个表示对象属性的nil值的单个实例。dictionaryWithValuesForKeys:的默认实现和相关的setValuesForKeysWithDictionary:会在NSNull (在字典参数中) 和nil(在存储的属性中)之间进行自动转换 。

当您使用键路径来寻址属性时, 如果键路径中的最后一个键是一对多关系 (即引用集合), 则返回的值是一个集合, 其中包含对多关系的键右侧的键的所有值。例如, 请求键路径 "`transactions.payee`" 的值返回包含所有`transaction`对象中`payee`对象的数组。这也适用于键路径中的多个数组。键路径`accounts.transactions.payee`返回包含所有帐户中所有交易记录的所有收款人对象的数组。

### 使用键设置属性值

与 `getter`一样, 键值编码兼容对象还提供了一小组具有默认行为的广义 `setter`, 它基于在`NSObject`中对`NSKeyValueCoding`协议的实现:

*   `setValue:forKey:` 设置相对于接收到给定值消息的对象的指定键的值。`setValue:forKey:`的默认实现会自动对表示标量和结构的`NSNumber`和`NSValue`对象执行`unwarp`操作，并将它们设置到相应的属性中。有关`warp`和`unwarp`的详细信息, 详见*表示非对象值*。

如果接收`setter`调用的对象中没有对应指定键的属性，该对象将自己发送一个`[setValue:forUndefinedKey:]`消息。`setValue:forUndefinedKey:`的默认实现将抛出`NSUndefinedKeyException`异常。但是, 子类可以重写此方法以自定义方式处理请求。

*   `setValue:forKeyPath:` 在相对于接收者的指定键路径上设置给定值。键路径序列中指定键所对应的对象如果不是键值编码兼容的，将会收到`setValue:forUndefinedKey:`消息。

*   `setValuesForKeysWithDictionary:` 将指定字典中的值设置到接收者的属性中, 使用字典键标识属性。默认实现调用每个键值对的`setValue:forKey:` , 根据需要用`nil`替换`NSNull`对象。

在默认实现中, 当您尝试将非对象属性设置为`nil`值时, 键值编码兼容对象将自己发送一个`setNilValueForKey:`消息。`setNilValueForKey:`的默认实现将抛出`[NSInvalidArgumentException]`异常, 但对象可能会重写此行为以替换默认值或标记值, 详见*处理非对象值*。

### 使用键简化对象访问

想知道基于键的 `getter` 和 `setter` 如何简化代码, 请查看下面的示例。在 macOS 中, `NSTableView`和`NSOutlineView`对象将标识符字符串与每列关联起来。如果表的模型对象不是符合键值编码的, 则表的数据源方法将强制检查每个列标识符, 依次查找要返回的正确属性, 如清单 2-2所示。此外, 在将来, 当您向模型中添加另一个属性时, 在本例中为`Person` 对象, 还必须重新访问数据源方法, 添加另一个条件来测试新属性并返回相关值.

清单 2-2不基于键值编码的数据源方法的实现

```objc
- (id)tableView:(NSTableView *)tableview objectValueForTableColumn:(id)column row:(NSInteger)row {
    id result = nil;
    Person *person = [self.people objectAtIndex:row];

    if ([[column identifier] isEqualToString:@"name"]) {
        result = [person name];
    } else if ([[column identifier] isEqualToString:@"age"]) {
        result = @([person age]);  // Wrap age, a scalar, as an NSNumber
    } else if ([[column identifier] isEqualToString:@"favoriteColor"]) {
        result = [person favoriteColor];
    } // And so on...

    return result;
}

```

另一方面,清单 2-3展示了相同数据源的方法的一个更紧凑的实现, 该数据源方法使用的是键值编码兼容的`Person`对象。仅使用`valueForKey:` getter, 数据源方法将使用列标识符作为键返回适当的值。除了更短的时间外, 它还更通用, 因为在以后添加新列时, 只要列标识符始终与模型对象的属性名称匹配, 它就会继续保持不变。

清单 2-3基于键值编码的数据源方法的实现

```objc
- (id)tableView:(NSTableView *)tableview objectValueForTableColumn:(id)column row:(NSInteger)row {
    return [[self.people objectAtIndex:row] valueForKey:[column identifier]];
}
```

 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。








