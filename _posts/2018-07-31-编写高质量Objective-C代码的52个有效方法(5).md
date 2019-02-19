---
layout: post
title: Effective Objective-C 2.0 编写高质量iOS代码的52个有效方法 (5)
tags:
	- iOS
---

Effective Objective-C 2.0 编写高质量iOS和OSX代码的52个有效方法 (5)
====

##以弱引用避免保留环 (循环引用)
unsafe_unretain 当实例变量的引用移除之后,属性仍然指向哪个已经回收的实例  
但是weak属性指向nil  

##以"自动释放池块"降低内存峰值

系统中的线程都是GCD机制中的线程,这些线默认都有自动释放池,每次执行事件循环的时候都将清空

>自动释放池排布在栈中,对象收到autoRelease消息后,系统将其放入最顶端的迟中
>合理运用自动释放池,可以降低应用程序的内存峰值
>@autoreleasepool 新式写法能创建出更为轻便的自动释放池


##用僵尸对象调试内存管理问题

NSZombieEnabled环境变量设置为YES
>
>* 系统在回收对象时,可以不将其真的回收,而是转化为僵尸对象
>* 系统会修改对象的isa指针,令其指向特殊的僵尸类,从而使该对象编程僵尸对象.僵尸类能够响应所有的选择器,打印一条包含消息内容及其接收者的消息,然后终止应用程序

##不要用retainCount

>ARC已经废弃了这个获取引用计数的方法

因为这个方法只会返回某一个时间节点的引用计数,而不会考虑接下来系统会清空自动释放池  
而且这个方法可能永远不会返回0,因为有时候系统会优化对象的释放行为,在计数还是1的时候就释放了.    

#第六章 块(block)与大中枢派发 (Grand Central Display-GCD)

##理解block
block可以实现闭包, 这项语言特性是作为扩展(extension)加入GCC编译器中的,在Clang中都可以使用,Clang是iOS和Mac OS app所用的编译器;

###基础知识
用^表示  

```
^{
	// 实现代码
}
```

block也是个对象 ,

```
void (^someBlock)() = ^{
	// 实现代码
}
```

> 在block声明的范围内,所有变量都可以被他捕获
> 在block外部的变量不可以在内部修改,想要修改,需要加上__block修饰符

####内联block
```
NSarray *array = @[@1,@2];
__block NSIngeter count = 0;
[array enumarateObjectsUsingBlock:^(NSNumber *number, NSUInteger idx, BOOL *stop) {
	///
}];
```

####block的内部结构
block本身也是对象,存放在存放块对象的内存区域中,首个变量是指向class的指针,isa指针  


void*|isa
------|------
int|flags
int|reserved
void(*)(void *, ...)|invoke
struct * | descriptor
捕获到的变量|捕获到的变量

* 块描述符 (descriptor)

unsigned long int | reserved
-------------|-------
unsigned long int | size
void (*)(void *, void *) | copy
void (*)(void *, void *) | dispose

> 捕获的变量一直保存在内存中,那么在其他的作用域调用block的时候,能不能从block中获取其在定义时候捕获的变量

> 定义的时候,所占的内存区域是分配在栈中的,也就是说,只在定义的那个作用域有效;

```
void (^block)();
if () {
	block = ^{
	
	}
} else {
	block = ^{
	
	}
}
block();
```

在if 和else 两个作用域中的block, 很有可能已经释放掉,造成崩溃;  

> 解决这个问题,可以使用copy消息拷贝, 这样会复制到堆上.

```
void (^block)();
if () {
	block = [^{
	
	} copy];
} else {
	block = [^{
	
	} copy];
}
block();
```
> block 是C C++ Objective-C中的词法闭包
> block 可以接收参数,也可以返回值
> 可以分配在栈上面,也可以是全局的.分配在栈上的可以拷贝到堆里面.这样的话,就和Objective-C中的对象一样,具备引用计数了

## 为常用的block类型创建typedef

```
typedef int(^SomeBlock)(BOOL flag, int value);
```

作为方法的参数的时候应该把block的名字放在外面

> 要点
> 
>* 重新定义可以时block用起来更简单  
>* 应该遵循现有的命名习惯   
>* 不放为同一个block定义多个类型别名, 如果要重构代码使用了某个block类型的某个别名,那么只需要修改响应的typedef中的签名即可

##用Handle 块降低代码分散程度

一个页面的请求过多的时候,还是用delegate回调更容易管理

> 一般是网络请求回调之类的异步回调


##使用block的时候避免循环引用


##多用派发队列,少用同步锁(@synchronize)

同步锁能保证在一个线程操作一个数据,但是不能保证同一个线程不同操作的结果   
使用串行队列来避免这种情况,

```
_synchronizedQueue = dispatch_queue_creat(@"com.zxf.synchronizedQueue",NULL);
-(NSString *)someString {
	__block NSString *localSomeString;
	dispatch_sync(_synchronizedQueue, ^{
		localString = _someString;
	});
	return localString;
}

- (void)setSomeString:(NSString *)someString {
	
	dispach_sync(_synchronizedQueue, ^{
	
		_someString = someString;
	});
	
}

```
以上的思路是,吧设置操作和读取操作都放在序列化的队列中执行, 这样的话,所有对于这个属性的访问都同步了;
但是可以做进一步优化,即设置方法可以设置成异步执行, 因为设置方法不需要返回什么值;  
但是异步之后可能会变慢,因为异步派发需要拷贝块,执行任务的时间可能比拷贝的时间要短  
多个获取方法可以并发执行,但是设置方法和获取方法之间不能异步执行,  

使用并发队列 + barrie

```
void dispatch_barrie_async(dispatch_queue_t queue, dispatch_block block);
void dispatch_barrie_sync(dispatch_queue_t queue, dispatch_block block);

```


barrie总是单独执行,这个只对并发队列有意义,因为串行队列总是单独执行每一个任务.  
并发队列如果发现接下来要执行的是一个barrie,则会等待当前的所有任务都执行完之后,单独执行这个block, 完毕之后,在执行后面的任务.  

> 要点
> 
> * 使用派发队列可以用来描述同步语义, 这种方式要比直接使用同步锁或者NSLock要更简单
> * 将同步和异步结合起来,可以实现与普通加锁一样的同步行为,而却这么做不会阻塞执行异步派发的线程
> * 使用同步队列和barrie_block, 可以领同步行为更加高效


##多用GCD, 少用performSelector方法
 
* 编译器会提示可能有内存泄露
* performSelector方法的返回值只能是id类型,如果想要其他类型,就必须要进行复杂的转换工作
* 参数个数和参数类型也是同样受限的


##掌握GCD操作队列的使用时机

从iOS4开始 NSOperationQueue的底层就开始用GCD来实现
虽然是重量级的对象,但是仍然有一些好处

* 取消某个操作
* 指定操作之间的依赖关系
* 通过KVO监测NSOperation对象的属性(isCanceled, isFinished)
* 指定操作的优先级, GCD的优先级是针对队列来说的, 并不是针对单个操作
* 一些系统提供的NSOperation对象可以重复使用


##通过dispatch group机制,根据系统资源状况来执行任务

dispatch group是GCD的一箱特性,能够吧任务分组  
创建

```
dispatch_group_t dispatch_group_creat();
```

两种方法将任务分组:

```
void dispatch_group_async(dispatch_group_t group,dispatch_queue_t queue, dispatch_block_t block);
```

或者下面这种

```
void dispatch_group_enter(dispatch_group_t group);
void dispatch_group_leave(dispatch_group_t group);

```

下面这个函数哦用来等待group执行完毕, 

```
long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeOut);
```

时间参数表示执行完group需阻塞多久,如果执行的时间小于time,则返回0,否则返回非0,这个参数也可以取常量 **DISPATCH\_TIME\_FOREVER** 表示一直等待group执行结束  
除此之外,也可以用下面这个方法等待group执行完毕  

```
void dispatch_group_notify(group, queue, block);
```

* 同一个group中的任务也可以放到不同的优先级的线程上面


dispatch_apply 可以实现for循环的效果,反复执行block一定的次数

```
//
//    dispatch_queue_t queue = dispatch_queue_create("com.zxf.queue", NULL);
//    dispatch_apply(10, queue, ^(size_t i) {
//        NSLog(@"%zd",i);
//    });
    
    // 如果采用并发队列
    dispatch_queue_t queue1 = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_apply(20, queue1, ^(size_t i) {
        NSLog(@"%zd",i);
    });
}

```



##使用Dispatch_once来执行只需要执行一次的安全代码

> token应该声明为global作用域或者static保证每次传进去都是相同的值

##不要使用dispatch_get_currentQueue

>* 因为队列有层级关系,有肯能判断的队列并不是当前队列,但是这个队列的父队列或者上层队列刚好是当前队列,或者当前队列的福队列,这样就会造成死锁
>* 解决上述问题,使用GCD提供的"队列特有数据"(queue-specify data) 可以把任意数据以键值对的形式关联到队列中



#第七章 系统框架
##熟悉系统框架
将一系列代码封装成动态库,并在其中放入描述接口的头文件,这样的东西就叫做框架  


##多用block枚举,少用for循环

NSEnumator拥有反向枚举器  reverseEnumator
主要就是高效
  




