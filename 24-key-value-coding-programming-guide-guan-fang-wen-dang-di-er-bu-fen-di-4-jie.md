> Key-Value Coding Programming Guide 官方文档第二部分第4节
> 2018.9.18 第一次修正 Key-Value Coding 修正为 KVC

[iOS-KVC官方文档第二部分第4节](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/DataTypes.html#//apple_ref/doc/uid/20002171-BAJEAIEE)

# Key-Value Coding Fundamatals--Representing Non-Object Values
# 表示非对象值

`NSObject`提供的KVC协议方法的默认实现与对象和非对象属性一起使用。 默认实现自动在对象参数或返回值以及非对象属性之间进行转换。 这允许即使存储的属性是标量或结构体，基于key的getter和setter的命名也保持一致。

> 注意
> 因为Swift中的所有属性都是对象，所以本节仅适用于Objective-C属性。

当你调用协议的其中一个getter时，例如`valueForKey：`，默认实现根据[Accessor Search Patterns](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/SearchImplementation.html#//apple_ref/doc/uid/20000955-CJBBBFFA)中描述的规则 确定为指定键提供值的特定访问器方法或实例变量。 如果返回值不是对象，则getter使用此值初始化`NSNumber`对象（对于标量）或`NSValue`对象（对于结构）并返回该值。

类似地，默认情况下，像`setValue：forKey:`这样的setter在给定特定键的情况下确定属性的访问器或实例变量所需的数据类型。 如果数据类型不是对象，则setter首先向传入值对象发送适当的`<type> Value`消息以提取基础数据，并存储该数据。

> 注意
> 当您使用非对象属性的nil值调用其中一个KVC协议`setter`时，setter没有明显的一般操作过程。 因此，它向接收setter调用的对象发送`setNilValueForKey：`消息。 此方法的默认实现引发`NSInvalidArgumentException`异常，但子类可能会覆盖此行为，如[Handling Non-Object Values](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/HandlingNon-ObjectValues.html#//apple_ref/doc/uid/10000107i-CH5-SW1)中所述，例如设置标记值或提供有意义的默认值。

### 包装和展开标量类型

Table 5-1 列出默认KVC实现使用`NSNumber`实例包装的标量类型。 对于每种数据类型，该表显示了用于从基础属性值初始化`NSNumber`以提供getter返回值的创建方法。 然后显示用于在设置操作期间从setter输入参数中提取值的访问器方法。

**Table 5-1** 标量类型包含在`NSNumber`对象中


| Data type | Creation method | Accessor method |
| ---| --- | --- |
| `BOOL` | `numberWithBool:` | `boolValue` (in iOS)    `charValue` (in macOS)*|
| `char`| `numberWithChar:` | `charValue` |
| `double`| `numberWithDouble:` | `doubleValue` |
| `float` | `numberWithFloat:` | `floatValue` |
| `int` | `numberWithInt:` | `intValue` |
| `long` | `numberWithLong:` | `longValue` |
| `long long` | `numberWithLongLong:` | `longLongValue` |
| `short` | `numberWithShort:` | `shortValue` |
| `unsigned char`  | `numberWithUnsignedChar:` | `unsignedChar` |
| `unsigned int` | `numberWithUnsignedInt:`| `unsignedInt` |
| `unsigned long` | `numberWithUnsignedLong:` | `unsignedLong` |
| `unsigned long long` | `numberWithUnsignedLongLong:`|`unsignedLongLong` |
| `unsigned short` | `numberWithUnsignedShort:` | `unsignedShort` |

> 注意
> *在macOS中，由于历史原因，BOOL的类型定义为`signed char`，而KVC不区分这些。 因此，当`key`为`BOOL`时，不应将字符串值（例如@“true”或@“YES”）传递给`setValue：forKey：`。 KVC将尝试调用`charValue`（因为`BOOL`本身就是一个`char`），但是`NSString`没有实现这个方法，这会导致运行时错误。 相反，当`key`为`BOOL`时，只传递一个`NSNumber`对象，如`@(1)` 或 `@(YES)`，作为`setValue：forKey:` 的值参数。 此限制不适用于iOS，其中BOOL的类型定义为本机布尔类型bool，KVC调用boolValue，它适用于`NSNumber`对象或格式正确的`NSString`对象。

### 包装和展开结构体

[Table 5-2](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/DataTypes.html#//apple_ref/doc/uid/20002171-184580-BCIEDECF) 下面列出了可以包装成`NSPoint`，`NSRange`，`NSRect`和`NSSize`的常用结构体的转化方法和取值方法

**Table 5-2** 包装成`NSValue`的常用结构类型

| Data type | Creation method | Accessor method |
| --- | --- | --- |
| NSPoint | [valueWithPoint:](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSValue/Description.html#//apple_ref/occ/clm/NSValue/valueWithPoint:) | pointValue|
| NSRange | [valueWithRange:](https://developer.apple.com/documentation/foundation/nsvalue/1410315-valuewithrange) | rangeValue |
| NSRect | [valueWithRect:](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSValue/Description.html#//apple_ref/occ/clm/NSValue/valueWithRect:) (macOS only). | rectValue |
| NSSize | [valueWithSize:](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSValue/Description.html#//apple_ref/occ/clm/NSValue/valueWithSize:) | sizeValue |

不仅`NSPoint`，`NSRange`，`NSRect`和`NSSize`的可以自动包装和展开。 结构体类型（即，Objective-C类型中字符串以{开头编码的类型）可以包成NSValue对象中。 例如，参考5-1表格中声明的结构体和类接口。

**Listing 5-1** 使用自定义结构体的类

```objc
typedef struct {
    float x, y, z;
} ThreeFloats;
 
@interface MyClass
@property (nonatomic) ThreeFloats threeFloats;
@end
```

使用名为`myClass`的类的实例，可以使用KVC获取threeFloats值

```objc
NSValue* result = [myClass valueForKey:@"threeFloats"];
```

`valueForKey：`的默认实现调用`threeFloats` 的getter方法，然后返回包含在`NSValue`对象中的结果。

同样，您可以使用KVC设置`threeFloats`值：

```objc
ThreeFloats floats = {1., 2., 3.};
NSValue* value = [NSValue valueWithBytes:&floats objCType:@encode(ThreeFloats)];
[myClass setValue:value forKey:@"threeFloats"];
```

The default implementation unwraps the value with a [getValue:](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSValue/Description.html#//apple_ref/occ/instm/NSValue/getValue:)  message, and then invokes `setThreeFloats:`with the resulting structure.


 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。
