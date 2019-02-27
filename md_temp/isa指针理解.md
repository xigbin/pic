
# isa指针理解

## instance对象实例

iOS代码一般使用`id`来声明一个对象，对于`id`的理解，在文件`#import<objc/objc.h>`有以下声明，相关代码如下：

	/// An opaque type that represents an Objective-C class.
	typedef struct objc_class *Class;
	
	/// Represents an instance of a class.
	struct objc_object {
	    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
	};
	
	/// A pointer to an instance of a class.
	typedef struct objc_object *id;

根据对代码的理解不难发现，其实我们创建的对象跟实例其实本质上就是一个`struct obje_object`结构体，`id`也就是这个结构体的指针。而这个结构体中也只有一个`Class`类型的变量`isa`，也是一个`objc_class`结构体指针。
我们知道一个对象的创建都必须依赖于一个类来创建，因此在在这里对象的`isa`指针就指向就是所属类中根据类模板创建出的实例变量和方法。
如这一段代码：

	NSString *str = @"tellmeone";

结合上面对着一段代码的理解就是：对象`str`本质是一个`objc_object`的结构体，而这个结构体的成员变量`isa`指向的就是`NSString`类，而这个`NSString`实质就是个类对象。

## class object（类对象）/metaclass（元类）

查看源码对结构体`objc_class`的成员定义如下：

	struct objc_class {
		Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

	#if !__OBJC2__
		Class _Nullable super_class                              OBJC2_UNAVAILABLE;
		const char * _Nonnull name                               OBJC2_UNAVAILABLE;
		long version                                             OBJC2_UNAVAILABLE;
		long info                                                OBJC2_UNAVAILABLE;
		long instance_size                                       OBJC2_UNAVAILABLE;
		struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
		struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
		struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
		struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
	#endif

	} OBJC2_UNAVAILABLE;

结构体中`obje_class`中存放的数据就是元数据`metadata`，里边包含了指向父类的指针，当前类名，对应版本，对象的大小，对象里边的实例列表、方法列表、缓存、遵守的协议列表。以这些数据来创建一个对象。
其中第一个也是`isa`指针，就说明`Class`本身也是一个对象，这样的对象称之为 **类对象**。一般类对象和类方法的创建就是从`isa`指向的结构体创建的，而此处类对象`isa`指向的称之为元类`mataclass`，元类中保存了创建类方法和类对象的所需的所有信息，以下图帮助理解：

![isa指针结构图](https://github.com/xigbin/pic/blob/master/md_pic/runtime_isa%E6%8C%87%E9%92%88%E7%BB%93%E6%9E%84%E5%9B%BE.png?raw=true)

一个实例对象也就是`objc_object`结构体他的`isa`指向的是类对象，类对象中`isa`指向的是了元类，对应的`super_class`指针则指向了父类的类对象，而元类的`isa`的指向最终指向的是`NSObject`元类，所以问题来了，`NSObject`元类的`isa`指向哪里呢？

![实例类对象的isa指针结构图](https://github.com/xigbin/pic/blob/master/md_pic/runtime_%E5%AE%9E%E4%BE%8B%E7%B1%BB%E5%AF%B9%E8%B1%A1%E7%9A%84isa%E6%8C%87%E9%92%88%E7%BB%93%E6%9E%84%E5%9B%BE.png?raw=true)

通过这个图可以看出整个体系构成了一个自闭环。整个实例，类对象，元类的概念

