---
layout: post
title: Objective-C 编码规范
tags:
  - iOS
---

# Objective-C 编码规范

## 留白和格式

### 空格 和 制表符

> 缩进四个空格或者一个制表符

在编辑器设置成一个制表符四个空格

### 行宽

尽量将代码保持在 80 列以内,除非超过之后有助于提高可读性.

### 方法声明和定义

> - / + 和返回类型之间必须使用一个空格,参数列表从安徽总只有参数之间可以有空格.

方法应该像这样:

```objc
- (void)doSomethingWithString:(NSString *)theString {
	...
}
```

星号前的空格可选,但是要保持一致.

如果一行有非常多的参数,更好的方式是将每个参数单独拆成一行.如果使用多行,将每个参数前的冒号对齐

```objc
- (void)doSomeThingWith:(YUZFOO *)theFoo
				   rect:(CGRect)theRect
			   interval:(float)theInterval {

}
```

当第一个关键字比其他的短的时候,保证下一行至少有四个空格的缩进.这样可以使关键字垂直对齐:

```objc
- (void)short:(GTMFoo *)theFoo
    longKeyword:(NSRect)theRect
    evenLongerKeyword:(float)theInterval {
    ...

```

### 方法调用

> 方法调用应该尽量保持与方法声明的格式一致,当格式的风格有多种选择的时候,新的代码要与已有代码保持一致.

调用时所有参数应该在同一行:

```objc
[myObject doFooWith:arg1 name:arg2 error:arg3];

```

或者每行一个参数,以冒号对齐:

```objc
[myObject doFooWith:arg1
               name:arg2
              error:arg3];
```

不要使用下面的缩进风格:

```objc
[myObject doFooWith:arg1 name:arg2
              error:arg3];

[myObject doFooWith:arg1
               name:arg2 error:arg3];

[myObject doFooWith:arg1
          name:arg2  // 对齐错误
          error:arg3];
```

### 异常

> 每个@ 标签应该有独立的一行, 在`@` 和 `{}` 之间需要有一个空格,`@catch` 与被捕捉到的异常对象的声明之间也要有一个空格

如果使用了异常处理,使用这个格式.但是一般不推荐使用异常处理

```objc
@try {
  foo();
}
@catch (NSException *ex) {
  bar(ex);
}
@finally {
  baz();
}
```

### 协议名

> 类型标识符和尖括号内的协议名之间,不能有任何空格

这条适用于类声明,实例变量以及方法声明:

```objc
@interface MyProtocoledClass : NSObject<NSWindowDelegate> {
  id<MyFancyDelegate> delegate_;
}

@property (nonatomic, weak) id<MyFancyDelegate> delegate;

- (void)setDelegate:(id<MyFancyDelegate>)aDelegate;
@end
```

### block (闭包)

> `block` 适合用在 target/selector 模式下创建回调方法时.`block`使代码更易读. `block`中的代码应该缩进四个空格.

具体取决于`block` 的长度,一般遵循下列原则:

- 如果一行可以写完,则没必要换行
- 如果不得不换行,关括号应该与 block 声明的第一个字符对齐
- block 内的代码应该以四个空格缩进
- 如果 block 太长,比如超过 20 行,建议把 block 定义为一个局部变量,然后再使用该变量.
- 如果 block 不带参数,`^{` 之间无需空格,如果带有参数 `^(` 之间无需空格,但是 `) {` 之间必须有一个空格.

```objc
// 单行block
[operation setCompletionBlock:^{ [self onOperationDone]; }];

// 对齐大括号
[operation setCompletionBlock:^{
    [self.delegate newDataAvailable];
}];

// 使用C api 的block 时,沿用OC 的规范
dispatch_async(fileIOQueue_, ^{
    NSString* path = [self sessionFilePath];
    if (path) {
      // ...
    }
});

// 先定义实例变量
void (^largeBlock)(void) = ^{
    // ...
};
[operationQueue_ addOperationWithBlock:largeBlock];
```

## 命名

尽可能遵守 Apple 的命名约定,尤其是和内存管理规则相关的地方

推荐使用长的,描述性的方法和变量名

**推荐:**

```objective-c
UIButton *settingsButton;
```

**不推荐:**

```objective-c
UIButton *setBut;
```

### 分类名(类别名)

> 文件名必须反映出实现了什么类.

分类的文件名应该包含被扩展的类名,比如: `NSString+Utils.h`

### 方法名

> 方法名应当以小写字母开头,并遵循驼峰命名法. 每个参数名也应该以小写字母开头.

方法名应该尽量读起来像句子,应该选择与方法名连在一起读起来通顺的参数名,(例如, `convertPoint:fromRect` 或者 `replaceCharectersInRange:withString`).

访问器方法应该与他们要获取的成员变量的名字一样,不应该以 get 作为前缀:

```objc
- (id)getDelegate;  // 不推荐
- (id)delegate;     // 推荐
```

### 变量名

> 变量名应该以小写字母开头,并使用驼峰格式. 类的成员变量应该以下划线作为后缀. 例如: `myLocalVarable`, `myInstanceVariable_` .

#### 普通变量名

对于静态属性,不要使用匈牙利命名法. 尽量为变量起一个描述性的名字. 可以让代码阅读者立即理解代码. 例如:

- 错误的命名

```objc
int w;
int nerr;
int nCompConns;
tix = [[NSMutableArray alloc] init];
obj = [someObject object];
p = [network port];
```

- 正确的命名

```objc
int numErrors;
int numCompletedConnections;
tickets = [[NSMutableArray alloc] init];
userInfo = [someObject object];
port = [network port];

```

#### 常量

> 常量应该以驼峰命名法命名,并以相关类名作为前缀

**推荐**

```objc
static const NSTimeInterval YUZSignInViewControllerFadeOutAnimationDuration = 0.4;
```

**不推荐**

```objc
static const NSTimeInterval fadeOutTime = 0.4;
```

推荐使用常量来代替字符串字面值和数字,方便复用,而且可以快速修改二而不需要查找和替换. 常量应该用 static 来声明为静态常量,而不要用 #define, 除非他明确的作为一个宏来使用:

**推荐**

```objc
static NSString * const YUZCacheControllerDidClearCacheNotification = @"YUZCacheControllerDidClearCacheNotification";
static const CGFloat YUZImageThumbnailHeight = 50.0f;
```

**不推荐**

```objc
#define CompanyName @"Apple Inc."
#define magicNumber 42
```

如果一个常量需要在多个文件中使用,应该在头文件中这样暴露给外部:

```objc
extern NSString *const YUZCacheControllerDidClearCacheNotification;
```

如果一个常量只在一个文件中使用,可以直接用 k 开头命名:

```objc
static NSString * const kCacheControllerDidClearCacheNotification = @"kCacheControllerDidClearCacheNotification";

```

## 字面值

> 使用字面值来创建不可变的 `NSString` , `NSDictionary`, `NSArray` 和 `NSNumber` 对象.

注意不要将 nil 传进 `NSArray` 和 `NSDictionary` 中,会导致崩溃

**例如:**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
```

**不要:**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
```

## 注释

### 文件注释

> 每个文件的开头以文件的简要描述为起始, 紧接着是作者,最后是版权声明和许可证样板

#### 版权信息和作者

每个文件应该包括如下:

- 文件内容简要描述
- 代码作者
- 版权/许可证样板

#### 声明部分的注释

> 每个接口, 分类 以及协议应该注释,以描述他的目的以及与整个项目的关系.

```objc
// A delegate for NSApplication to handle notifications about app
// launch and shutdown. Owned by the main app controller.
@interface MyAppDelegate : NSObject {
  ...
}
@end
```

如果已经在文件头部详细描述了接口,可以直接说明"完成信息参见文件头部"

另外,公共接口的每个方法,都应该有注释来解释他的作用,参数,返回值以及其他影响

如果有的话,为类的线程安全性做注释. 如果类的实例可以被多个线程访问,注释多线程条件下的使用规则.

#### 实现部分的注释

> 使用 | 来引用注释用的变量以及符号名而不是使用引号

这样会避免二义性.

**例如:**

```objc
// 有时候 |count| 可以为负数.
```

或者当引用已经包含引号的符号

```objc
// Remember to call |StringWithoutSpaces("foo bar baz")|
```

## 条件语句

条件语句体应该总是被大括号包围。尽管有时候你可以不使用大括号（比如，条件语句体只有一行内容），但是这样做会带来问题隐患。比如，增加一行代码时，你可能会误以为它是 if 语句体里面的。此外，更危险的是，如果把 if 后面的那行代码注释掉，之后的一行代码会成为 if 语句里的代码。

**推荐:**

```objective-c
if (!error) {
    return success;
}
```

**不推荐:**

```objective-c
if (!error)
    return success;
```

和

```objective-c
if (!error) return success;
```

在 2014 年 2 月 苹果的 SSL/TLS 实现里面发现了知名的 [goto fail](https://gotofail.com/) 错误。

代码在这里：

```objective-c
static OSStatus
SSLVerifySignedServerKeyExchange(SSLContext *ctx, bool isRsa, SSLBuffer signedParams,
                                 uint8_t *signature, UInt16 signatureLen) {
  OSStatus        err;
  ...

  if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
    goto fail;
  if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
    goto fail;
    goto fail;
  if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
    goto fail;
  ...

fail:
  SSLFreeBuffer(&signedHashes);
  SSLFreeBuffer(&hashCtx);
  return err;
}
```

显而易见，这里有没有括号包围的 2 行连续的 `goto fail;` 。我们当然不希望写出上面的代码导致错误。

此外，在其他条件语句里面也应该按照这种风格统一，这样更便于检查。

### 尤达表达式

不要使用尤达表达式。尤达表达式是指，拿一个常量去和变量比较而不是拿变量去和常量比较。它就像是在表达 “蓝色是不是天空的颜色” 或者 “高个是不是这个男人的属性” 而不是 “天空是不是蓝的” 或者 “这个男人是不是高个子的”

**推荐:**

```objc
if ([myValue isEqual:@42]) { ...
```

**不推荐:**

```objc
if ([@42 isEqual:myValue]) { ...
```

### nil 和 BOOL 检查

类似于 Yoda 表达式，nil 检查的方式也是存在争议的。一些 notous 库像这样检查对象是否为 nil：

```objc
if (nil == myValue) { ...
```

或许有人会提出这是错的，因为在 nil 作为一个常量的情况下，这样做就像 Yoda 表达式了。 但是一些程序员这么做的原因是为了避免调试的困难，看下面的代码：

```objc
if (myValue == nil) { ...
```

如果程序员敲错成这样：

```objc
if (myValue = nil) { ...
```

这是合法的语句，但是很难看出错误来

为了避免这些奇怪的问题，可以用感叹号来作为运算符。因为 nil 是 解释到 NO，所以没必要在条件语句里面把它和其他值比较。同时，不要直接把它和 `YES` 比较，因为 `YES` 的定义是 1， 而 `BOOL` 是 8 bit 的，实际上是 char 类型。

**推荐:**

```objc
if (someObject) { ...
if (![someObject boolValue]) { ...
if (!someObject) { ...
```

**不推荐:**

```objc
if (someObject == YES) { ... // Wrong
if (myRawValue == YES) { ... // Never do this.
if ([someObject boolValue] == NO) { ...
```

同时这样也能提高一致性，以及提升可读性。

### 黄金大道

在使用条件语句编程时，代码的左边距应该是一条“黄金”或者“快乐”的大道。 也就是说，不要嵌套 `if` 语句。使用多个 return 可以避免增加循环的复杂度，并提高代码的可读性。因为方法的重要部分没有嵌套在分支里面，并且你可以很清楚地找到相关的代码。

**推荐:**

```objc
- (void)someMethod {
    if (![someOther boolValue]) {
        return;
    }

    // Do something important
}
```

**不推荐:**

```objc
- (void)someMethod {
    if ([someOther boolValue]) {
        // Do something important
    }
}
```

### 复杂的表达式

当你有一个复杂的 if 子句的时候，应该把它们提取出来赋给一个 BOOL 变量，这样可以让逻辑更清楚，而且让每个子句的意义体现出来。

```objc
BOOL nameContainsSwift  = [sessionName containsString:@"Swift"];
BOOL isCurrentYear      = [sessionDateCompontents year] == 2014;
BOOL isSwiftSession     = nameContainsSwift && isCurrentYear;

if (isSwiftSession) {
    // Do something very cool
}
```

### 三元运算符

三元运算符 ? 应该只用在它能让代码更加清楚的地方。 一个条件语句的所有的变量应该是已经被求值了的。类似 if 语句，计算多个条件子句通常会让语句更加难以理解。或者可以把它们重构到实例变量里面。

**推荐:**

```objc
result = a > b ? x : y;
```

**不推荐:**

```objc
result = a > b ? x = c > d ? c : d : y;
```

当三元运算符的第二个参数（if 分支）返回和条件语句中已经检查的对象一样的对象的时候，下面的表达方式更灵巧：

**推荐:**

```objc
result = object ? : [self createObject];
```

**不推荐:**

```objc
result = object ? object : [self createObject];
```

### 错误处理

有些方法通过参数返回 error 的引用，使用这样的方法时应当检查方法的返回值，而非 error 的引用。

**推荐:**

```objc
NSError *error = nil;
if (![self trySomethingWithError:&error]) {
    // Handle Error
}
```

此外，一些苹果的 API 在成功的情况下会对 error 参数（如果它非 NULL）写入垃圾值（garbage values），所以如果检查 error 的值可能导致错误 （甚至崩溃）。

## Case 语句

除非编译器强制要求，括号在 case 语句里面是不必要的。但是当一个 case 包含了多行语句的时候，需要加上括号。

```objc
switch (condition) {
    case 1:
        // ...
        break;
    case 2: {
        // ...
        // Multi-line example using braces
        break;
       }
    case 3:
        // ...
        break;
    default:
        // ...
        break;
}
```

有时候可以使用 fall-through 在不同的 case 里面执行同一段代码。一个 fall-through 是指移除 case 语句的 “break” 然后让下面的 case 继续执行。

```objc
switch (condition) {
    case 1:
    case 2:
        // code executed for values 1 and 2
        break;
    default:
        // ...
        break;
}
```

当在 switch 语句里面使用一个可枚举的变量的时候，`default` 是不必要的。比如：

```objc
switch (menuType) {
    case YUZEnumNone:
        // ...
        break;
    case YUZEnumValue1:
        // ...
        break;
    case YUZEnumValue2:
        // ...
        break;
}
```

此外，为了避免使用默认的 case，如果新的值加入到 enum，编译器会马上提示 warning

`Enumeration value 'YUZEnumValue3' not handled in switch.（枚举类型 'YUZEnumValue3' 没有被 switch 处理）`

### 枚举类型

当使用 `enum` 的时候，建议使用新的固定的基础类型定义，因为它有更强大的类型检查和代码补全。 SDK 现在有一个 宏来鼓励和促进使用固定类型定义 - `NS_ENUM()`

**例子:**

```objc
typedef NS_ENUM(NSUInteger, YUZMachineState) {
    YUZMachineStateNone,
    YUZMachineStateIdle,
    YUZMachineStateRunning,
    YUZMachineStatePaused
};
```

## 类

### 类名

类名应该以三个大写字母作为前缀(双字母的为 Apple 的类预留).

创建一个子类的时候,应该把说明性的部分放在前缀和父类名的中间.

举个例子：如果你有一个 `YUZNetworkClient` 类，子类的名字会是`YUZTwitterNetworkClient` (注意 "Twitter" 在 "YUZ" 和 "NetworkClient" 之间); 按照这个约定， 一个`UIViewController` 的子类会是 `YUZTimelineViewController`.

### Initializer 和 dealloc

推荐的代码组织方式是将 `dealloc` 方法放在实现文件的最前面, `init` 应该紧跟在 `dealloc` 后面.

如果有多个初始化方法, 指定初始化方法 (designated initializer) 应该放在最前面, 间接初始化方法 (secondary initializer) 跟在后面, ARC 模式下不需要实现 `dealloc` 方法, 但是把他们写在一起, `init`中做的事情要在`dealloc`中撤销.

### Designated 和 Secondary 初始化方法

```objc

@implementation YUZEvent

- (instancetype)initWithTitle:(NSString *)title
                         date:(NSDate *)date
                     location:(CLLocation *)location {
    self = [super init];
    if (self) {
        _title    = title;
        _date     = date;
        _location = location;
    }
    return self;
}

- (instancetype)initWithTitle:(NSString *)title
                         date:(NSDate *)date {
    return [self initWithTitle:title date:date location:nil];
}

- (instancetype)initWithTitle:(NSString *)title {
    return [self initWithTitle:title date:[NSDate date] location:nil];
}

@end
```

`initWithTitle:date:location:` 就是指定初始化方法, 另外的两个是间接初始化方法.

> 一个类应该有且只有一个 `designated` 初始化方法，其他的初始化方法应该调用这个 `designated` 的初始化方法（虽然这个情况有一个例外）

### instancetype

> 使用 `instancetype` 来作为初始化方法的返回值

### 单例

> 使用 `dispatch_once()` 函数来实现单例.

```objc
+ (instancetype)sharedInstance {
    static id sharedInstance = nil;
    static dispatch_once_t onceToken = 0;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
    });
    return sharedInstance;
}
```

### 属性

> 属性尽可能的使用描述性命名.避免缩写.

应该总是使用 setter 和 getter 方法访问属性, 除了 `init` 和 `dealloc` 方法.避免出现副作用.

- 使用`setter` 会遵守定义的内存管理语义(strong, copy, weak ...)
- KVO 通知 会被自动执行
- 更容易 debug
- 允许在一个单独的地方为设置属性添加额外的逻辑

#### 属性定义

推荐按照下面格式来进行属性定义:

```objc
@property (nonatomic, readwrite, copy) NSString *name;
```

属性的参数应该按照: 原子性, 读写 和内存管理 排列.

属性可以存储一个`block` ,为了存活到定义 block 的结束,必须使用 `copy` ,用来拷贝到堆里面去.

如果要创建一个公有的 `getter` 和一个私有的 `setter` ,应该声明公开的属性为 `readonly` 并且在类的扩展中重新定义通用的属性为 `readwrite`.

```objc
// .h文件中
@interface MyClass : NSObject
@property (nonatomic, readonly, strong) NSObject *object;
@end
// .m文件中
@interface MyClass ()
@property (nonatomic, readwrite, strong) NSObject *object;
@end

@implementation MyClass
// Do Something cool
@end
```

描述 `BOOL`属性的词如果是形容词, 那么 setter 不应该带 `is` 前缀, 但是对应的 `getter`访问器应该带上这个前缀. 如:

```objc
@property (assign, getter=isEditable) BOOL editable;
```

#### 私有属性

> 私有属性应该定义在类的实现文件的扩展中,不允许在有名字的 category 中定义属性. 除非扩展其他类.

### 可变对象

> 任何可以用一个可变的对象设置的(NSArray , NSString, NSURLRequest)属性的内存管理类型必须是`copy`的.

同时应该尽量避免暴露在公开接口中的可变对象. 可以提供只读属性来返回对象的不可变的副本.

## 方法

### 参数断言

> 方法可能要求一些参数来满足特定条件,这个时候最好使用 `NSParameterAssert()` 来断言条件是否成立或者抛出一个异常.

### 私有方法

> 不要在私有方法前加 `_` 前缀. 这个前缀是 Apple 保留的.

## Categories

> 在 category 方法前加上自己的小写前缀以及下划线, 比如 `- (void)yuz_myCategoryMethod` .

## NSNotification

> 定义`notification`的时候应该将通知的名字定义成一个字符串常量.类似暴露给其他类的字符串常量.

同时,用一个 `did/will`这样的动词以及用 Notifications 后缀来命名.

例如:

```objc
// Foo.h
extern NSString * const YUZFooDidBecomeBarNotification

// Foo.m
NSString * const YUZFooDidBecomeBarNotification = @"YUZFooDidBecomeBarNotification";
```

## 代码组织

### pragma

> 利用 pragma 来组织一个类内部的代码

分离:

- 不同功能组的方法
- protocols 的实现
- 对父类方法的重写

```objc
- (void)dealloc { /* ... */ }
- (instancetype)init { /* ... */ }

#pragma mark - View Lifecycle （View 的生命周期）

- (void)viewDidLoad { /* ... */ }
- (void)viewWillAppear:(BOOL)animated { /* ... */ }
- (void)didReceiveMemoryWarning { /* ... */ }

#pragma mark - Custom Accessors （自定义访问器）

- (void)setCustomProperty:(id)value { /* ... */ }
- (id)customProperty { /* ... */ }

#pragma mark - IBActions

- (IBAction)submitData:(id)sender { /* ... */ }

#pragma mark - Public

- (void)publicMethod { /* ... */ }

#pragma mark - Private

- (void)YUZ_privateMethod { /* ... */ }

#pragma mark - UITableViewDataSource

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath { /* ... */ }

#pragma mark - YUZSuperclass

// ... 重写来自 YUZSuperclass 的方法

#pragma mark - NSObject

- (NSString *)description { /* ... */ }
```

## 对象之间的通讯

### Block

- 使用 copy 修饰
- 避免循环引用
  例子:

```objc
__weak __typeof(self) weakSelf = self;
[self executeBlock:^(NSData *data, NSError *error) {
    [weakSelf doSomethingWithData:data];
}];
```

多个语句的时候,避免`self` 提前释放

```objc
__weak __typeof(self)weakSelf = self;
[self executeBlock:^(NSData *data, NSError *error) {
    __strong __typeof(weakSelf) strongSelf = weakSelf;
    if (strongSelf) {
        [strongSelf doSomethingWithData:data];
        [strongSelf doSomethingWithData:data];
    }
}];
```

在 ARC 条件中，如果尝试用 -> 符号访问一个实例变量，编译器会给出非常清晰的错误信息：

```objc
Dereferencing a __weak pointer is not allowed due to possible null value caused by race condition, assign it to a strong variable first. (对一个 __weak 指针的解引用不允许的，因为可能在竞态条件里面变成 null, 所以先把他定义成 strong 的属性)
```

可以用下面的代码展示:

```objc
__weak typeof(self) weakSelf = self;
myObj.myBlock = ^{
    id localVal = weakSelf->someIVar;
};
```

## 其他 Cocoa 和 Objective-C 特性

### 初始化

> 不要在 init 方法中,将成员变量初始化为 0 或者 nil

### 不要使用 `+new` 方法

> 不要调用 new 方法,也不要重载. 使用 `alloc` 和 `init`方法创建并初始化对象.

### 保持 公共 api 简单

> 如果一个函数没必要公开,就不要公开,用私有分类保证公共头文件简洁.

### 使用根框架

> `#import` 根框架而不是单独的文件

_推荐_

```objc
#import <Foundation/Foundation.h>     // good
```

_不推荐_

```objc
#import <Foundation/NSArray.h>        // avoid
#import <Foundation/NSString.h>
```

### 在 setter 方法中 copy NSString

> 接受`NSString` 作为参数的`setter` ,应该总是`copy` 传入的字符串

### BOOL 若干陷阱

> 将普通整形转换成 `BOOL` 时要小心。不要直接将 `BOOL` 值与 `YES` 进行比较。

Ojbective-C 中把 `BOOL` 定义成无符号字符型，这意味着 `BOOL` 类型的值远不止 `YES`(1)或 `NO`(0)。不要直接把整形转换成 `BOOL`。常见的错误包括将数组的大小、指针值及位运算的结果直接转换成 `BOOL` ，取决于整型结果的最后一个字节，很可能会产生一个 `NO` 值。当转换整形至 `BOOL` 时，使用三目操作符来返回 `YES` 或者 `NO`。

对 `BOOL` 使用逻辑运算符（&&，|| 和 !）是合法的，返回值也可以安全地转换成 `BOOL`，不需要使用三目操作符。

错误的用法:

```objc
- (BOOL)isBold {
  return [self fontTraits] & NSFontBoldTrait;
}
- (BOOL)isValid {
  return [self stringValue];
}
```

正确的用法:

```objc
- (BOOL)isBold {
	return ([self fontTraits] & NSFontBoldTrait) ? YES : NO;
}
- (BOOL)isValid {
	return [self stringValue] != nil;
}
- (BOOL)isEnabled {
	return [self isValid] && [self isBold];
}

```

同样，不要直接比较 YES/NO 和 BOOL 变量。不仅仅因为影响可读性，更重要的是结果可能与你想的不同。

错误的用法:

```objc
BOOL great = [foo isGreat];
if (great == YES)
  // ...be great!
```

正确的用法：

```objc
BOOL great = [foo isGreat];
if (great)
  // ...be great!
```

### 文件导入(import)

> 如果在.h 文件中不需要用到类的属性,只需要声明, 使用 `@class` 声明, 在.m 文件中 使用 `#import` 导入

## Cocoa 模式

### 委托模式

> 委托对象不应该被 `retain` (用 `weak` 修饰)

### MVC

> 分离模型和试视图. 分离控制器和模型, 视图. 回调 API 用 `@protocol` 或者 `block` .

# 参考

- [Google Objective-C Style Guide](https://zh-google-styleguide.readthedocs.io/en/latest/google-objc-styleguide/)
- [objc-zen-book-cn](https://github.com/oa414/objc-zen-book-cn/#%E5%B8%B8%E9%87%8F)

# 推荐

- [iOS App ProgrammingGuide](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html)
- [NYTimes/objective-c-style-guide](https://github.com/NYTimes/objective-c-style-guide)
- [Coding Guide for Cocoa](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
