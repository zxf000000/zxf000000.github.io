---
layout: post
title: Effective Objective-C 2.0 编写高质量iOS代码的52个有效方法 (3)
tags:
	- iOS
---


Effective Objective-C 2.0 编写高质量iOS与OSX代码的52个有效方法(3)
=====
##用"方法调配技术"调试"黑盒方法"
方法调配 即**Method swizzling**  
类的方法列表会把选择器的名称映射到相关的方法实现上面. 使得"动态消息派发方法"能找到应该调用的方法, 这些方法以函数指针的形式实现, 这种指针叫做IMP

```
id (*IMP)(id, SEL, ...);
```

交换方法实现,用下列函数  

```
void method_exchangeImplementations(Method m1, Method m2);
```

方法实例通过下列函数

```
Method class_getInstanceMethod(Class class, SEL aSelector);
```

这种方法主要在分类中对已有的方法添加实现,而不需要重新创建子类来实现

>要点
>
>* 在运行期,可以向类中新增或替换已有选择器的实现
>* 使用另一种实现替换原有的方法,
>* 只有调试方的时候使用这个方法,一般在生产时期尽量不要用


##理解"类对象"的用意
对象类型并非在编译期绑定好,而是在运行时查找.  

* 每个OC对象都是指向一块内存数据的指针,因此要加 * 
* isa指针就是is a 指针
* 类的isa指针指向元类(metaClass),类就是元类的对象,类方法就是元类的实例方法

> isMenberOfClass 可以查询是否是某各类的实例  
> idKindOfClass 可以查询对象是否为某类或者其派生类的实例
> 应该尽量使用类型信息查询方法来查询某个对象所属的类

#第三章 接口与API设计

##使用前缀避免命名空间冲突

* Apple保留了两个字母前缀的使用权,所以一般应该使用三个子母机前缀

##提供全能初始化方法

也就是提供一个初始化方法,其他初始化方法都要调用他

* 如果自定义的类中有必须赋值的初始化属性,n哪么应该参照下面的两种方法去实现默认的init方法
	
	```
	- (id)init {
		
		return [self initWith.....];
	}
	// 或者抛出异常
	- (id)init {
		@throw [NSException exceptionWithName........];
	}
	
	
	```
	
> 如果子类的全能初始化方法与父类的不同,name子类必须要重写父类的全能初始化方法
> 如果不想重写,最好抛出异常,保证不出错误

##实现description方法
在系统类的对象中使用NSLog打印通常输出的是属性  
但是在自定义类中,通常输出的是内存地址和类名  
只需要重写descript方法,输出对象内容就行了  
debugdescript方法是在LLDB的输出中采用的,也就是断点调试 po 命令

##尽量使用不可变对象
可以在类的声明中声明属性为readonly,然后在扩展中重新声明为readwrite,这样保证只能在类内部修改该属性  
在对象外部(依然可以用KVC来为属性赋值)  

> 不要把可变的collection属性公开,而是应该提供相应的方法去修改其中的数据,对外公布的还是不可变对象

##使用清晰而协调的命名方式


##为私有方法名加前缀

* 有利于调试
* 最好有字母p和_
* 不要使用一个单独的下划线作为前缀(Apple规定的)


##理解Objective-C错误模型

ARC在默认情况下不是"异常安全的",也就是说,抛出异常之后,哪么在本作用域末尾释放的那些资源不会自动释放了,会引起内存泄露.  

OC的推荐做法是: 抛出异常之后,直接退出程序,这样就不需要写"异常安全"代码了  

正常情况下,Objective-C使用返回值nil或者0,或者NSError来返回错误  
NSError对象封装了三条信息

* Error Domain 错误范围.类型为字符串
* error code 类型为整数
* userinfo 类型为字典

> 最好能为自己的库创建一个专用的错误范围字符串,这样,发生错误的时候,就能直接定位


##理解NSCopying协议
使用对象的时候,需要拷贝的情况下,需要自己实现NSCopying协议,这个协议只有一个方法

	- (id)copyWithZone:(NSZone *)zone;

不需要担心zone参数  

	// 实现方法
	- (id)copyWithZone:(NSzone *)zone {
		SomeClass someCopy = [[[self class] allocWithZone:zone] initWithSomeProperty:....];
		return someCopy;
	}

如果还有其他属性,也需要一并拷贝过来  


NSMutableCopying协议.返回可变版本copy

默认进行的都是浅拷贝,如果想进行深拷贝,需要自己实现