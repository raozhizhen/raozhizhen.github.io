---
date: 2016-01-19 12:00
status: public
title: '属性声明在@implementation里与extension里的区别'
---

当你新建一个类的时候，Xcode会自动给你写上以下代码。

	#import <Foundation/Foundation.h>

	@interface Car : NSObject

	@end

	
	#import "Car.h"

	@implementation Car

	@end
	
Objective-C编译器指令是以@打头，它通常用来描述文件中的内容。.h文件中@interface指令用来标识文件的接口代码的起始位置，而@end指令标示该段的结束位置。在.m文件中，@implementation指令用来标识实现的起始位置，@end标识结束位置

@interface用于定义类的公共接口，通常，接口被称为API（application programming interface）而真正使对象能够运行的代码，位于@implementation中。

当我们要给一个Car类声明一个发动机属性的时候，如果对外公开，则代码为

	#import <Foundation/Foundation.h>

	@interface Car : NSObject
	
	@property (nonatomic, strong) Engine *engine;

	@end

如果不对外公开，则在.m里的代码为

	@interface Car ()

	@property (nonatomic, strong) Engine *engine;

	@end
	
@interface Car ()看起来和.h里的@interface Car : NSObject很像，其实@interface Car ()是一个特殊的匿名 Category，即扩展（extension）。

类别（Category）是一种为现有的类添加新方法的方式。

利用Objective-C的动态运行时分配机制，Category提供了一种比继承（inheritance）更为简洁的方法来对class进行扩展，无需创建对象类的子类就能为现有的类添加新方法，可以为任何已经存在的class添加方法，包括那些没有源代码的类（如某些框架类），申明的方法不需要在@implementation里实现。

但Category无法向类中添加新的实例变量，类别没有空间容纳实例变量。（也有一些技术可以克服类别无法增加新实例变量的局限。例如，使用全局字典来存储对象与你想要关联的额外变量之间的映射。）

而extension可以添加新的实例变量

@property是以@开头，所以它也是Objective—C编译器指令，用于声明属性，并为它自动创建一个带下划线的实例变量，及实例变量的setter和getter方法。
	
而直接声明实例变量的写法，即

	@interface Car () {
	
	     Engine *_engine;
	}
	
	@end
	
和

	@implementation Car {
	
	    Engine *_engine;
	}

	@end
	
从语法上说它们等效。

如果只是声明一个@implementation里需要用到的全局变量，自然是放在@implementation里声明，但如果是声明一个不对外公开的属性呢，比如engine，既然是属性，好像是需要在extension里声明，但如果我使用_engine来访问成员变量，则并不会用到它的setter和getter方法。如果我使用点语法来访问成员变量呢，点语法其实是调用了getter方法[Car engine]，而这种默认的隐藏在代码中多了，会影响代码的阅读和维护。

但engine明明是Car的一个属性，却声明在@implementation里作为一个变量，其实实例变量也是这个对象的构成元素，和属性除了名字并没有涵义上的区别。所以在@implementation里声明的变量也是这个对象的属性，只是为了区分两种声明方式的叫法不同而已。

另一个用@property和@implementation声明属性的区别就是，@property可以给属性添加属性标识符，即assign，copy，weak，strong，nonatomic，但其实大部分的属性标识符都有对应的所有权修饰符，assign对应\_\_unsafe_unretained，copy对应\_\_strong修饰符（但copy赋值的是被复制的对象），strong对应\_\_strong，weak对应\_\_weak。id和对象类型在没有明确指定所有权修饰符时，默认为\_\_strong修饰符，而@property声明属性的默认属性标识符为readwrite，assign, atomic。atomic的确没有对应的所有权修饰符，id和对象类型自然是没有原子性的，在iOS开发，除非特殊需要，我们都会给属性标识符添加nonatomic，所以在这点上，@property和@implementation声明属性倒是没什么区别。

在@interface里使用@property声明属性的时候，如果属性类型为NSString，它的属性标识符是需要添加copy的，原因就在与，设置方法的新值有可能指向一个NSMutableString类的实例，那么设置完属性之后，字符串的值就可能会在对象不知情的情况下遭人更改，那在@implementation里声明一个NSString会不会有这个顾虑呢？copy不是简单的赋值，对应的__strong并不会通过copyWithZone:方法复制赋值源所生产的对象，所以@implementation里声明的NSString没有copy作用的修饰符，但在@implementation里声明即这个属性是不对外公开的，即不会被其它对象直接修改这个属性，那你既然声明了一个NSString类型的属性，自然用意就是使用一个不可变的字符串，自然自己不会去修改它，如果你无意中修改了它，我只能说这是你的代码写错了。所以不需要使用copy作用的修饰符。同理，在extension里使用@property声明NSString，也是不需要copy属性标识符的。所以NSString在@implementation里声明并不会有所影响。

总结
在@implementation里声明并没有缺点，但在extension里使用@property声明属性，会有不带来价值的隐藏代码，以及_engine比self.engine更简短易读，最后还有可以避免在init和dealloc中会去调用self.engine。

参考资料

《iOS与OS X多线程和内存管理》

《Objective-C基础教程（第二版）》

[iOS 开发中的争议（一）](http://blog.devtang.com/blog/2015/03/15/ios-dev-controversy-1/)