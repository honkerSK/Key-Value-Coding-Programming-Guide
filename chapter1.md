> Key-Value Coding Programming Guide 官方文档第一部分

[iOS-KVC官方文档第一部分](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107-SW1)

# Key-Value Coding Programming Guide - Getting Started
> 键值编码编程指南-入门

> 该文档苹果官方已不再更新。有关Apple SDK的最新信息，请访问[文档网站](https://developer.apple.com/documentation)。

## 关于键值编码
补充: key-value coding 翻译为 键值编码 , 简称KVC.

键值编码是一种由`NSKeyValueCoding`非正式协议启用的机制，对象采用该机制提供对其属性的间接访问。当对象符合键值编码时，其属性可通过字符串参数通过简洁，统一的消息传递接口寻址。这种间接访问机制补充了实例变量及其相关访问​​器方法提供的直接访问。

您通常使用访问器方法来访问对象的属性。get访问器（或getter）返回属性的值。set访问器（或setter）设置属性的值。在Objective-C中，您还可以直接访问属性的基础实例变量。以任何这些方式访问对象属性都很简单，但需要调用特定于属性的方法或变量名称。随着属性列表的增长或变化，访问这些属性的代码也必须如此。相反，符合键值编码的对象提供了一个简单的消息传递接口，该接口在其所有属性中都是一致的。

键值编码是一个基本概念，是许多其他Cocoa技术的基础，例如键值观察(key-value observing)，Cocoa绑定(Cocoa bindings)，Core Data和AppleScript-ability。在某些情况下，键值编码还有助于简化代码。


### 使用键值编码兼容对象
对象通常在`NSObject`（直接或间接）继承时采用键值编码，它们都采用`NSKeyValueCoding`协议并为基本方法提供默认实现。这样的对象通过紧凑的消息传递接口使其他对象能够执行以下操作：

*   **访问对象属性。**该协议指定方法，例如getter [`valueForKey:`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/EOF/EOControl/Classes/NSObjectAdditions/Description.html#//apple_ref/occ/instm/NSObject/valueForKey:) 和setter [`setValue:forKey:`](https://developer.apple.com/documentation/objectivec/nsobject/1415969-setvalue)，用于通过名称或键访问对象属性，参数为字符串。这些和相关方法的默认实现使用键来定位基础数据并与其交互，如[Accessing Object Properties](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/BasicPrinciples.html#//apple_ref/doc/uid/20002170-BAJEAIEE)。

*   **操纵集合属性。**访问方法的默认实现和对象的集合属性（如`NSArray`对象）一样，也和任何其他属性一样。此外，如果对象定义属性的集合访问器方法，则它允许对集合内容进行键值访问。这通常比直接访问更有效，并允许您通过标准化界面使用自定义集合对象，如[Accessing Collection Properties](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/AccessingCollectionProperties.html#//apple_ref/doc/uid/10000107i-CH4-SW1)。

*   **在集合对象上调用集合运算符。**在符合键值编码的对象中访问集合属性时，可以将*集合运算符*插入到键字符串中，如[Using Collection Operators](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/CollectionOperators.html#//apple_ref/doc/uid/20002176-BAJEAIEE)。集合运算符根据默认的`NSKeyValueCoding`getter实现对集合执行操作，然后返回集合的新的过滤版本或表示集合的某些特征的单个值。

*   **访问非对象属性。**协议默认实现检测非对象属性，包括标量和结构体，并自动将它们包装和解包为协议接口上使用的对象，如[Representing Non-Object Values](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/DataTypes.html#//apple_ref/doc/uid/20002171-BAJEAIEE)。此外，该协议声明了一种方法，该方法允许兼容对象`nil`通过键值编码接口在非对象属性上设置值时为该情况提供合适的作用。

*   **key path访问属性。**如果具有符合键值编码的对象层次结构，则可以使用基于key path的方法调用，使用单个调用在层次结构内深入查看，获取或设置值。



### 采用对象的键值编码

为了使您自己的对象键值编码符合要求，您需要确保它们采用`NSKeyValueCoding`非正式协议并实现相应的方法，例如作为[`valueForKey:`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/EOF/EOControl/Classes/NSObjectAdditions/Description.html#//apple_ref/occ/instm/NSObject/valueForKey:) 通用getter和[`setValue:forKey:`](https://developer.apple.com/documentation/objectivec/nsobject/1415969-setvalue) 通用setter。幸运的是，如上所述，[`NSObject`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Protocols/NSObject/Description.html#//apple_ref/occ/intf/NSObject)  采用此协议并为这些和其他基本方法提供默认实现。因此，如果您从`NSObject`（或其许多子类中的任何一个）派生对象，那么大部分工作已经完成。

为了使默认方法完成其工作，您需要确保对象的访问器方法和实例变量遵循某些明确定义的模式。这允许默认实现找到对象的属性以响应键值编码消息。然后，您可以选择通过提供验证方法和处理某些特殊情况来扩展和自定义键值编码。

### 使用Swift进行键值编码
`NSObject`从其子类或其子类之一 继承的Swift对象默认情况下是符合其属性的键值编码。而在Objective-C中，属性的访问器和实例变量必须遵循某些模式，Swift中的标准属性声明会自动保证这一点。另一方面，协议的许多功能要么不相关，要么使用Objective-C中不存在的本机Swift构造或技术来更好地处理。例如，因为所有Swift属性都是对象，所以您永远不会使用默认实现对非对象属性的特殊处理。

因此，虽然键值编码协议方法直接转换为Swift，但本指南主要关注Objective-C，您需要做更多工作以确保合规性，以及键值编码通常最有用的地方。整个指南中都提到了需要在Swift中采用明显不同方法的情况。

有关使用Swift和Cocoa技术的更多信息，请阅读*将Swift与Cocoa和Objective-C一起使用（Swift 3）*。有关Swift的完整描述，请阅读*Swift编程语言（Swift 3）*。

### 使用键值编码的其他Cocoa技术
符合键值编码的对象可以参与依赖于此类访问的各种Cocoa技术，包括：

*   **键值观察(Key-value observing)。**此机制使对象能够注册异步通知监听另一个对象属性的改变，如“ *[Key-Value Observing Programming](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)*。

*   **Cocoa绑定(Cocoa bindings)。**这一系列技术完全实现了Model-View-Controller范例，其中模型(Model)用于封装应用程序数据，视图(View)用于显示和编辑数据，控制器(Controller)在两者之​​间进行调解。阅读*[Cocoa Bindings Programming Topics](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaBindings/CocoaBindings.html#//apple_ref/doc/uid/10000167i)*以了解有关Cocoa绑定的更多信息。

*   **核心数据(Core Data)。**该框架为与对象生命周期和对象图形化管理相关的常见任务（包括持久性）提供通用和自动化解决方案。您可以在*[Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html#//apple_ref/doc/uid/TP40001075)*阅读Core Data 。

*   **AppleScript。**这种脚本语言可以直接控制脚本化应用程序和macOS的许多部分。Cocoa的脚本支持利用键值编码来获取和设置脚本化对象中的信息。`NSScriptKeyValueCoding`非正式协议中的方法提供了使用键值编码的附加功能，包括通过多值键中的索引获取和设置键值，以及将键值强制（或转换）为适当的数据类型。*[AppleScript Overview](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptX/AppleScriptX.html#//apple_ref/doc/uid/10000156i)*提供了AppleScript及其相关技术的高级概述。


 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。
