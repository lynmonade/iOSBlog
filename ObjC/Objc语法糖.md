##使用instancetype
当你在编写alloc方法、initXXX方法、工厂方法时，推荐使用instancetype作为返回值，而不是id类型。

```objc
//MyObject.h
@interface MyObject : NSObject
+(instancetype)factoryA;
+(id)factoryB;
@end

//MyObject.m
@implementation MyObject
+(instancetype)factoryA {
    return [[[self class] alloc] init]; //line0: 使用[self class]
}

+(id)factoryB {
    return [[[self class] alloc] init];
}
@end

//main.m
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        [[MyObject factoryA] count]; //line1: 直接报错
        [[MyObject factoryB] count]; //line2: 编译通过，甚至连警告都没有
    }
    return 0;
}
```

在line0处，推荐使用`[self class]`，而不是直接使用类的名称。这样的好处是:使用工厂方法获得对象后，如果使用错误类型的指针指向对象，编译器会自动提出警告。

在line1和line2中，line1在编译期就会报错，因为factoryA返回的是instancetype类型，明确说明了返回值是MyObject类型，而MyObject并没有count方法，所以会报错。而factoryB返回的是id类型，id可以是任意类型，比如当是NSArray类型时，就会含有count方法。所以在编译期并不会报错，直到运行期才会报错。

因此正确使用instancetype关键字，能帮助我们在编译期就解决很多潜在的问题，而不是在运行期才发现问题。

##两个实用的枚举宏
```objc
//1. NS_ENUM: 单选，且必须是NSInteger类型
//旧的方式
enum {
        UITableViewCellStyleDefault,
        UITableViewCellStyleValue1,
        UITableViewCellStyleValue2,
        UITableViewCellStyleSubtitle
};
typedef NSInteger UITableViewCellStyle;

//新的方式
typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
        UITableViewCellStyleDefault,
        UITableViewCellStyleValue1,
        UITableViewCellStyleValue2,
        UITableViewCellStyleSubtitle
};

/*************************************************/

//2. NS_OPTIONS: 多选，且必须是NSUInteger类型
//旧的方式
enum {
        UIViewAutoresizingNone                 = 0,
        UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
        UIViewAutoresizingFlexibleWidth        = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
        UIViewAutoresizingFlexibleHeight       = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
typedef NSUInteger UIViewAutoresizing;

//新的方式
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
        UIViewAutoresizingNone                 = 0,
        UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
        UIViewAutoresizingFlexibleWidth        = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
        UIViewAutoresizingFlexibleHeight       = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

一般推荐把枚举变量定义在全局变量的位置，这样整个文件的所有方法都可以访问枚举变量。

##Designated Initializer
关于Designated Initializer的介绍，可以看下[Designated Initializer编写规范](http://www.jianshu.com/p/f92972bf7300)。在objc中，可以使用宏`NS_DESIGNATED_INITIALIZER `明确指出Designated Initializer。如果你的编写方式不符合规范，编译器就会提出警告。

```objc
//正确写法
//Person.h
@interface Person : NSObject
@property(nonatomic, strong)NSString *name;
@property(nonatomic, assign)NSInteger age;

-(instancetype)initWithName:(NSString *)name age:(NSInteger)age NS_DESIGNATED_INITIALIZER;
-(instancetype)initWithName:(NSString *)name; //secondary initializer
-(instancetype)init; //secondary initializer
+(instancetype)createPeople; //factory method
@end

//Person.m
@implementation Person
-(instancetype)initWithName:(NSString *)name age:(NSInteger)age {
    if((self=[super init])) {
        _name = name;
        _age = age;
    }
    return self;
}

-(instancetype)initWithName:(NSString *)name {
    return [self initWithName:name age:16]; //line0
}

-(instancetype)init {
    return [self initWithName:@"anonymous" age:16];
}

+(instancetype)createPeople {
    return [[[self class] alloc] init]; //可以使用secondary,也可以直接使用designated
}
@end
```

上面的代码完全符合Designated Initializer的规范。如果把line0的代码改成下面这样，编译器就会给出警告，这样我们就可以在编译期对潜在问题进行排查。

```objc
/*
警告: 
Convenience initializer missing a 'self' call to another initializer
Convenience initializer should not invoke an initializer on 'super'
*/
-(instancetype)initWithName:(NSString *)name {
    if((self=[super init])) { //wrong way
        _name = name;
    }
    return self;
}
```

##Literal Syntax
Literal Syntax可以看做是factory method，并且拥有更简洁的语法格式。

```objc
//line0,line1,line2是等价的!
NSString *str1 = @"Hello, World!"; //line0
NSString *str2 = [NSString stringWithCString:"Hello, World!" encoding:NSUTF8StringEncoding]; //line1
NSString *str3 = [[NSString alloc] initWithCString:"Hello, World!" encoding:NSUTF8StringEncoding]; //line2
    
NSNumber *boolNumber = @YES;
NSNumber *floatNumber = @3.14f;
NSNumber *intNumber = @42;
NSNumber *longNumber = @42L;
NSNumber *unsignedNumber = @42u;
NSNumber *doubleNumber = @3.1415926535;
NSNumber *someChar = @'T';
    
    
//create and query array
NSArray *someArray = @[firstObject, secondObject, thirdObject];
id object = someArray[0];
    
//create and query dictionary
NSDictionary *dictionary = @{@"key1":@"value1",
                             @"key2":@"value2",
                             @"key3":@"value3"};
id value1 = dictionary[@"key1"];

```

##变量
静态局部变量、全局变量、静态全局变量经常会搞混，下面分别讲解下。
###静态局部变量
静态局部变量在静态存储区分配空间，其生命周期与程序相同(即只要程序未关闭，静态局部变量就一直存在)。因此即使methodA调用完毕后，a变量并不会销毁。所以第二次调用方法时将打印a=11。此外，静态局部变量的作用域是该方法内部。

```objc
//ObjectA.m
/*
methodA第一次调用时,a=10;
methodA第二次调用时,a=11;
...以此类推
 */
-(void)methodA {
    static int a = 10; //静态局部变量
    NSLog(@"a=%d", a);
    a++;
}

//Client
ObjectA *objectA = [[ObjectA alloc] init];
[objectA methodA]; //a=10
[objectA methodA]; //a=11
```

###全局变量
全局变量对于整个程序的所有源文件均可见(即作用域是程序内)。所有源文件均可以访问和修改全局变量。全局变量也在静态存储区分配空间，生命周期与整个程序相同。

如果文件B需要使用定义在文件A中的全局变量，则必须在文件B中使用extern关键字重新声明这个全局变量。extern关键字会告诉编译器: 这个全局变量是存在的，但并没有定义在文件B中，而是存在于其他外部(extern)文件中，在编译链接时就会找到该全局变量了！

```objc
//ObjectA.m
/*
全局变量的位置
@implementation
...
@end
*/
int globalVar = 20; //定义全局变量。
@implementation ObjectA

-(void)changeGlobalVar {
    globalVar++;
    NSLog(@"globalVar=%d", globalVar);
}
@end

//ObjectB.m
extern int globalVar; //重新声明全局变量。注: 这里不需要import "ObjectA.m"
@implementation ObjectB

-(void)changeGlobalVar {
    globalVar++;
    NSLog(@"globalVar=%d", globalVar);
}
@end

//client
[[ObjectA new] changeGlobalVar]; //globalVar=21
[[ObjectA new] changeGlobalVar]; //globalVar=22
```

如果你希望全局变量为常量(不能被修改)，则可以在定义全局变量时，加入const关键字: 

```objc
const int globalVar = 20;
NSString * const cell = @"cell"; //注意const的位置
```
建议把一个程序中所需的全局变量统一定义在一个文件中。这样可以有效避免全局变量命名冲突，更好地管理所有全局变量。

###静态全局变量
静态全局变量的作用域是文件作用域(仅本文件内有效)。假设静态成员变量定义在文件A中，则文件A中的所有方法都可以使用这个静态成员变量。但文件B则无法直接使用这个静态成员变量。

静态全局变量也在静态存储区分配空间，生命周期与整个程序相同。

```objc
//ObjectA.m
static NSString *identifer = @"cell"; //静态全局变量
@implementation ObjectA
-(void)methodA {
    NSLog(@"%@", identifer); //cell
    identifer = @"cell2";
    NSLog(@"%@", identifer); //cell2
}
```

###变量小结
1. 局部变量在栈上分配空间。
2. 静态局部变量、全局变量、静态全局变量都是在静态存储区分配空间(注意不是堆噢！)
3. 给局部变量加上static关键字后，将变成静态局部变量。其作用域保持不变，但生命周期发生变化，与程序生命周期相同。
4. 给全局变量加上static关键字后，将变成静态全局变量，其生命周期保持不变，但作用域变成文件作用域。



#References
* [Adopting Modern Objective-C](https://developer.apple.com/library/ios/releasenotes/ObjectiveC/ModernizationObjC/AdoptingModernObjective-C/AdoptingModernObjective-C.html#//apple_ref/doc/uid/TP40014150-CH1-SW1)
* [Programming with Objective-C](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011210-CH1-SW1)




