> Key-Value Coding Programming Guide 官方文档第二部分第3节

[iOS-KVC官方文档第二部分第3节](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/AccessingCollectionProperties.html#//apple_ref/doc/uid/10000107i-CH4-SW1)

# Key-Value Coding Fundamatals--Using Collection Operators
### 使用集合运算符

当你发送向键值编码兼容的对象发送`valueForKeyPath:`消息时, 可以在键路径中嵌入*集合运算符*。集合运算符是前面有 at 符号 (`@`) 的一小部分关键字列表, 它指定了`getter`在返回之前以某种方式操作数据。`NSObject`提供的`valueForKeyPath:`的默认实现实现了此行为。

当键路径包含集合运算符时, 运算符前面的键路径的任何部分 (称为左键路径) 指示相对于消息接收者需要去操作的集合。如果将消息直接发送到集合对象 (如`NSArray`实例), 则可以省略左键路径。

**图 4-1**运算符键路径格式

![ 4-1 Operator key path format.jpg](https://upload-images.jianshu.io/upload_images/126164-76fe78092535649e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


集合运算符展示了三种基本行为类型:

*   **聚合运算符**以某种方式合并集合的对象, 并返回一个通常与在右键路径中命名的属性的数据类型相匹配的单个对象。`@count`运算符是一个特例，它没有右键路径, 并且始终返回`NSNumber`实例。

*   **数组运算符**返回一个`NSArray`实例, 其中包含命名集合中保存的对象的某些子集。

*   **嵌套运算符**处理包含其他集合的集合, 并返回一个`NSArray`或`NSSet`实例 (根据运算符), 它将嵌套集合的对象以某种方式组合在一起。

#### 示例数据

下面的说明包括演示如何调用每个运算符的代码段以及这样做的结果。这依赖于`BankAccount`类 (在[列表 2-1]中显示), 它包含`Transaction`对象的数组。其中每一个都代表一个简单的`checkbook`条目, 如清单 4-1中所声明的那样。

**清单 4-1**Transaction对象的接口声明

```objc
@interface Transaction : NSObject

@property (nonatomic) NSString* payee;   // To whom
@property (nonatomic) NSNumber* amount;  // How much
@property (nonatomic) NSDate* date;      // When

@end
```

为了进行讨论, 假定BankAccount实例具有一个事务数组, 其中填充了表 4-1中显示的数据, 并使示例从BankAccount对象内部调用。

**表 4-1**Transactions对象的示例数据

| payee | amount | date |
| --- | --- | --- |
| Green Power | $120.00 | Dec 1, 2015 |
| Green Power | $150.00 | Jan 1, 2016 |
| Green Power | $170.00 | Feb 1, 2016 |
| Car Loan | $250.00 | Jan 15, 2016 |
| Car Loan | $250.00 | Feb 15, 2016 |
| Car Loan | $250.00 | Mar 15, 2016 |
| General Cable | $120.00 | Dec 1, 2015 |
| General Cable | $155.00 | Jan 1, 2016 |
| General Cable | $120.00 | Feb 1, 2016 |
| Mortgage | $1,250.00 | Jan 15, 2016 |
| Mortgage | $1,250.00 | Feb 15, 2016 |
| Mortgage | $1,250.00 | Mar 15, 2016 |
| Animal Hospital | $600.00 | Jul 15, 2016 |

#### 聚合运算符

聚合运算符处理`array`或`set`属性, 生成一个反映集合的某些方面的单个值。

##### @avg

当指定@avg运算符时, `valueForKeyPath:`读取集合中每个元素的右键路径指定的属性, 将其转换为double (nil值用0替代), 并计算算术平均值。然后返回存储在NSNumber实例中的结果。

获取表 4-1中示例数据之间的平均交易记录金额:

```objc
NSNumber *transactionAverage = [self.transactions valueForKeyPath:@"@avg.amount"];
```

`transactionAverage`的格式化的结果为 $ 456.54。

##### @count

指定`@count`运算符时, valueForKeyPath:返回一个包含集合中的对象个数的NSNumber实例。右键路径 (如果存在) 将被忽略。

在transactions中获取Transaction对象的数目:

```objc
NSNumber *numberOfTransactions = [self.transactions valueForKeyPath:@"@count"];
```

`numberOfTransactions`的值为13。

##### @max

指定`@max`运算符时, `valueForKeyPath:`在由右键路径命名的集合项之间进行搜索, 并返回最大值。搜索使用`compare:`方法进行比较, 许多基础类 (如`NSNumber`类) 中都有定义。因此, 由右键路径指示的属性必须持有对此消息有意义响应的对象。搜索忽略值为`nil`的集合项。

在表 4-1中列出的交易记录中, 获取日期值 (即最新交易记录的日期) 的最大数量:

```objc
NSDate *latestDate = [self.transactions valueForKeyPath:@"@max.date"];
```

`latestDate`的值为 Jul 15, 2016.

##### @min

指定`@min`运算符时, `valueForKeyPath:`在由右键路径命名的集合项之间进行搜索, 并返回最小值。搜索使用`compare:`方法进行比较, 许多基础类 (如`NSNumber`类) 中都有定义。因此, 由右键路径指示的属性必须持有对此消息有意义响应的对象。搜索忽略值为`nil`的集合项。

在表 4-1中列出的事务中, 获取日期值 (即最早的事务的日期) 的最短时间。

```objc
NSDate *earliestDate = [self.transactions valueForKeyPath:@"@min.date"];
```

`earliestDate`的值为 Dec 1, 2015.

##### @sum

指定@sum运算符时, `valueForKeyPath:`读取集合中每个元素的右键路径指定的属性, 将其转换为`double` (`nil`值替换为 0), 并计算总和。然后返回存储在NSNumber实例中的结果。

获取表 4-1中示例数据之间的交易记录金额的总和:

```objc
NSNumber *amountSum = [self.transactions valueForKeyPath:@"@sum.amount"];
```

`amountSum`的结果为 $ 5935.00。

#### 数组运算符

数组运算符使`valueForKeyPath:`返回与右键路径指示的特定对象集相对应的对象数组。

> 重要
> 在使用数组运算符时, 如果有任何叶(`leaf`)对象为nil, 则`valueForKeyPath:`方法将引发异常。

##### @distinctUnionOfObjects

指定`@distinctUnionOfObjects`运算符时, `valueForKeyPath:`将创建并返回一个数组, 其中包含与右键路径指定的属性对应的集合的不同对象。

获取transactions中的交易记录的payee属性值的集合, 但省略了重复值:

```objc
NSArray *distinctPayees = [self.transactions valueForKeyPath:@"@distinctUnionOfObjects.payee"];
```

生成的`distinctPayees`数组包含以下每一个字符串实例：`Car Loan, General Cable, Animal Hospital, Green Power, Mortgage`。

> 注意
> `@unionOfObjects`运算符提供类似的行为, 但不删除重复的对象。

##### @unionOfObjects

指定`@unionOfObjects`运算符时, `valueForKeyPath:`将创建并返回一个数组, 其中包含与由右键路径指定的属性对应的集合的所有对象。与`@distinctUnionOfObjects`不同, 不删除重复对象。

获取`transactions`中的交易记录的payee属性值的集合:

```objc
NSArray *payees = [self.transactions valueForKeyPath:@"@unionOfObjects.payee"];
```

生成的`payees`数组包含以下字符串：`Green Power, Green Power, Green Power, Car Loan, Car Loan, Car Loan, General Cable, General Cable, General Cable, Mortgage, Mortgage, Mortgage, Animal Hospital`.记录了重复值。

> 注意
> `@distinctUnionOfArrays`运算符类似, 但移除重复对象。

#### 嵌套运算符

嵌套运算符对嵌套集合进行操作, 集合本身的每个条目都包含一个集合。

> 重要
> 如果在使用嵌套运算符时, 有任何叶(`leaf`)对象为nil, 则`valueForKeyPath:`方法将引发异常。

对于下面的说明, 请看第二个称为`moreTransactions`的数据数组, 其中填充了表 4-2中的数据, 并与原来的`transactions`数组一起插入嵌套数组:

```objc
NSArray* moreTransactions = @[<# transaction data #>];
NSArray* arrayOfArrays = @[self.transactions, moreTransactions];
```

**表 4-2** `moreTransactions`数组中假设的`Transaction`数据

| payee | amount | date |
| --- | --- | --- |
| General Cable - Cottage | $120.00 | Dec 18, 2015 |
| General Cable - Cottage | $155.00 | Jan 9, 2016 |
| General Cable - Cottage | $120.00 | Dec 1, 2016 |
| Second Mortgage | $1,250.00 | Nov 15, 2016 |
| Second Mortgage | $1,250.00 | Sep 20, 2016 |
| Second Mortgage | $1,250.00 | Feb 12, 2016 |
| Hobby Sho | $600.00 | Jun 14, 2016 |

##### @distinctUnionOfArrays

指定`@distinctUnionOfArrays`运算符时, `valueForKeyPath:`创建并返回一个数组, 其中包含与右键路径指定的属性相对应的所有集合的组合的不同对象。.

在`arrayOfArrays`中的所有数组中获取`payee`属性的不同值:

```
NSArray *collectedDistinctPayees = [arrayOfArrays valueForKeyPath:@"@distinctUnionOfArrays.payee"];

```

生成的`collectedDistinctPayees`数组包含以下值: `Hobby Shop, Mortgage, Animal Hospital, Second Mortgage, Car Loan, General Cable - Cottage, General Cable, Green Power`。

> 注意
> @unionOfArrays运算符类似, 但不移除重复对象。

指定`@unionOfArrays`运算符时, `valueForKeyPath:`创建并返回一个数组, 其中包含与由右键路径指定的属性相对应的所有集合的组合的所有对象, 而不删除重复项。

在`arrayOfArrays`中的所有数组中获取`payee`属性的值:

```
NSArray *collectedPayees = [arrayOfArrays valueForKeyPath:@"@unionOfArrays.payee"];

```

生成的`collectedPayees`数组包含以下值:`Green Power, Green Power, Green Power, Car Loan, Car Loan, Car Loan, General Cable, General Cable, General Cable, Mortgage, Mortgage, Mortgage, Animal Hospital, General Cable`。

> 注意
> `@distinctUnionOfArrays`运算符类似, 但移除重复对象。

##### @distinctUnionOfSets

当指定`@distinctUnionOfSets`运算符时, `valueForKeyPath:`创建并返回一个`NSSet`对象, 其中包含与由右键路径所指定的属性相对应的所有集合组合的不同对象。

此运算符的行为与`@distinctUnionOfArrays`类似, 只是它需要一个`NSSet`实例, 其中包含对象的`NSSet`实例, 而不是`NSArray`实例中包含`NSArray`实例。此外, 它还返回一个`NSSet`实例。假设示例数据已存储在集合而不是数组中, 则示例调用和结果与`@distinctUnionOfArrays`中显示的相同。

 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。