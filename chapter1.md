# [KVC翻译]1-Key-Value Coding Programming Guide 官方文档第一部分

> Key-Value Coding Programming Guide 官方文档第一部分


[iOS-KVC官方文档第一部分](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107-SW1)

# Key-Value Coding Programming Guide - Getting Started
> 键值编码编程指南-入门

> 该文档苹果官方已不再更新。有关Apple SDK的最新信息，请访问[文档网站](https://developer.apple.com/documentation)。

## 关于键值编码

键值编码是一种机制，通过`NSKeyValueCoding`非正式协议,对象采用这种机制提供对其属性的间接访问。当对象符合键值编码时, 它的属性可以使用字符串参数通过简明、统一的消息接口进行寻址。这种间接访问机制补充了实例变量及其关联的访问器方法提供的直接访问。

通常使用访问器方法获取对对象属性的访问权限。`get` 访问器 (或 `getter`) 返回属性的值。`set` 访问器 (或 `setter`) 设置属性的值。在`Objective-C`中, 还可以直接访问属性内部的实例变量。以上述任何一种方式访问对象属性都非常直截了当, 但需要调用属性特定的方法或变量名。随着属性列表的增长或更改, 必须编写访问这些属性的代码。相反, 键值编码兼容对象提供了一个在其所有属性中一致的简单消息传递接口。

键值编码是许多其他`Cocoa`技术的基础概念, 如`KVO`、`Cocoa bindings`、`Core Data`和 `AppleScript-ability`。在某些情况下, 键值编码还可以帮助简化代码。

### 使用键值编码兼容对象

对象通常采用键值编码, 当它们 (直接或间接) 继承`NSObject`时, 它们都采用了`NSKeyValueCodin`协议, 并为基本方法提供默认实现。此类对象使其他对象能够通过简洁的消息接口执行以下操作:

*   **访问对象属性.** 协议中指定的方法, 例如一般的 getter `valueForKey:`和常用的setter `setValue:forKey:`, 用于通过名称或键访问对象属性 (参数化为字符串)。这些方法和相关方法的默认实现使用key来定位基础数据并与之交互, 详见*访问对象属性*.

*   **操作集合属性.** 配合对象集合属性(如`NSArray`对象)使用的访问器方法的默认实现与对象其他属性的访问器方法一样。此外, 如果对象定义了属性的集合访问器方法, 则它允许对集合的内容进行键值访问。这通常比直接访问效率更高, 并允许你通过标准化的接口处理自定义集合对象, 详见*访问集合属性*.

*   **在集合对象上调用集合运算符.** 在键值编码兼容对象中访问集合属性时, 可以将*集合运算符*插入到键字符串中, 详见*使用集合运算符*。集合运算符指示`NSKeyValueCoding` getter 默认的实现去对集合执行某些操作, 然后返回一个全新的、过滤过的集合版本或单个表示某些集合特性的值 。

*   **访问非对象属性.** 协议的默认实现支持检测非对象属性, 包括标量和结构, 并自动将它们包装成对象和解包,详见*表示非对象值*.此外, 该协议还声明了一种方法, 允许兼容对象通过键值编码接口在非对象属性上设置`nil`值时为该情况提供适当的操作。

*   **按键路径访问属性.** 当你有一个键值编码兼容的对象的层级结构,你可以使用`key path based`方法在单次调用中获取或设置层次结构深处的值。

### 为对象采用键值编码

为了使您自己的对象兼容键值编码, 您可以确保它们遵循`NSKeyValueCoding`非正式协议并实现相应的方法, 如`valueForKey:`作为一般的 getter 和`setValue:forKey:`作为一般的 setter。幸运的是, 如上文所述, `NSObject`遵循此协议, 并为这些和其他基本方法提供了默认实现。因此, 如果从`NSObject` (或它的许多子类) 中派生对象, 则大部分工作已经完成。

为了使默认方法执行其工作, 您可以确保对象的访问器方法和实例变量遵守某些定义良好的模式。这允许默认实现在响应键值编码消息时查找对象的属性。然后, 您可以通过提供验证方法和处理某些特殊情况来扩展和自定义键值编码。

### Swift中的键值编码

默认情况下, 继承自`NSObject`或其子类的 `Swift` 对象，其属性是支持键值编码的。而在`Objective-C` 中, 属性的访问器和实例变量必须遵循某些模式, Swift 中的标准属性声明会自动保证这一点。另一方面, 许多协议的功能要么不相关, 要么使用某些`Objective-C`中没有的`Swift` 原生构造和技术能够更好地处理。例如, 由于所有 `Swift` 属性都是对象, 因此你永远不会使用到默认实现中对非对象属性的特殊处理。

虽然键值编码协议方法直截了当地翻译成`Swift`, 但本指南主要侧重于`Objective-C`, 在这里您需要做更多的事来确保兼容, 而键值编码通常最有用。在整个指南中都需要注意到在 `Swift` 中采取明显不同方法的情况。

有关使用 swift 与Cocoa技术的详细信息, 请参阅*[Using Swift with Cocoa and Objective-C (Swift 4.2)](https://developer.apple.com/documentation/swift#2984801)*。有关 swift 的完整说明, 请阅读*[The Swift Programming Language (Swift 4.2)](https://docs.swift.org/swift-book/index.html#//apple_ref/doc/uid/TP40014097)*.

### 其他依赖于键值编码的Cocoa技术

键值编码兼容的对象可以广泛的应用于依赖这种访问的Cocoa技术, 其中包括:

*   **键值观察.** 此机制使对象可以注册由其他对象属性中的更改驱动的异步通知, 详见*[Key-Value Observing Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)*.

*   **Cocoa bindings.** 此技术集完全实现了模型-视图-控制器范式, 其中模型封装应用程序数据、视图显示和编辑数据以及控制器在两者之间进行协调。阅读*[Cocoa Bindings Programming Topics](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaBindings/CocoaBindings.html#//apple_ref/doc/uid/10000167i)*了解有关Cocoa Bindings的更多信息。

*   **Core Data.** 此框架为与对象生命周期和对象图管理 (包括持久性) 相关的常见任务提供了通用和自动解决方案。您可以在*[Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html#//apple_ref/doc/uid/TP40001075)*中阅读有关*Core Data*的信息。.

*   **AppleScript** 这种脚本语言可以直接控制可脚本化的应用程序和 macOS 的许多部分。Cocoa的脚本支持利用键值编码来获取和设置可脚本化对象中的信息。`NSScriptKeyValueCoding`非正式协议中的方法为使用键值编码提供了额外的功能, 包括按多值键中的索引获取和设置键值, 并将键值强制 (或转换) 为适当的数据类型。*[AppleScript Overview](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptX/AppleScriptX.html#//apple_ref/doc/uid/10000156i)*提供了 AppleScript 及其相关技术的高级别概述。

 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。


