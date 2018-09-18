> Key-Value Coding Programming Guide 官方文档第二部分第5节

[iOS-KVC官方文档第二部分第5节](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/ValidatingProperties.html#//apple_ref/doc/uid/10000107i-CH18-SW1)

# Key-Value Coding Fundamatals--Validating Properties

## 验证属性

键值编码协议定义了支持属性验证的方法。正如使用基于键的访问器读取和写入键值编码兼容对象的属性一样, 也可以按键 (或键路径) 验证属性。当您调用`validateValue:forKey:error:`(或`validateValue:forKeyPath:error:`) 方法时, 协议的默认实现将搜索接收验证消息的对象 (或在键路径的末尾的对象), 该方法的名称与模式`validate<Key>:error:`相匹配。如果对象没有此类方法, 则默认情况下验证成功, 默认实现返回`YES`.当存在属性特定的验证方法时, 默认实现将返回调用该方法的结果。

> 注意
> 您通常仅在`Objective-C`中使用此处描述的验证。在 Swift 中, 通过依赖 optionals 和强类型检查的编译器支持, 可以更便捷地处理属性验证, 同时使用内置的 willSet 和 didSet 属性观察器来测试任何运行时 API 协定, 详见
> *[The Swift Programming Language (Swift 4.2)](https://docs.swift.org/swift-book/index.html#//apple_ref/doc/uid/TP40014097)*
> 中*[Property Observers](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-ID262)*章节对`willSet` `didSet`的描述。

由于属性特定的验证方法通过引用的方式接收值和错误参数, 因此验证有三种可能的结果:

1.  验证方法认为值对象有效并返回YES而不改变值或错误。
2.  验证方法认为值对象无效, 但选择不更改它。在这种情况下, 该方法返回NO并将错误引用 (如果调用方提供) 设置为指示失败原因的NSError对象。
3.  验证方法认为值对象无效, 但创建一个新的、有效的替换项。在这种情况下, 该方法返回YES同时使错误对象不被触及。返回之前, 该方法修改值引用以指向新值对象。当它进行修改时, 该方法总是创建一个新对象, 而不是修改旧值, 即使 value 对象是可变的。

清单 6-1显示了如何调用`name`字符串的验证的示例。

```objc
Person* person = [[Person alloc] init];
NSError* error;
NSString* name = @"John";
if (![person validateValue:&name forKey:@"name" error:&error]) {
    NSLog(@"%@",error);
}
```

### 自动验证

通常, 键值编码协议及其默认实现都不定义自动执行验证的任何机制。相反, 您可以在您的应用程序中使用适合的验证方法。

某些其他Cocoa技术在某些情况下会自动进行验证。例如, 当保存托管对象上下文时, `Core Data`自动执行验证 (详见*[Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html#//apple_ref/doc/uid/TP40001075)*)。此外, 在 macOS 中, `Cocoa Bindings` 允许您指定验证是否自动发生 (请阅读*[Cocoa Bindings Programming Topics](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaBindings/CocoaBindings.html#//apple_ref/doc/uid/10000167i)*了解有关Cocoa Bindings的更多信息。)。

 > 由于笔者水平有限，文中如果有错误的地方，或者有更好的方法，还望大神指出。
附上本文的所有 demo 下载链接，[【GitHub】]()。
如果你看完后觉得对你有所帮助，还望在 GitHub 上点个 star。赠人玫瑰，手有余香。