Designated Initializer翻译过来就是`指定初始化函数。先看一则关于UIViewController的小故事。

#如何创建UIViewController

如何创建一个UIViewController呢？不懂？老老实实翻API去...

![](http://ww2.sinaimg.cn/mw690/0065Y1avgw1f3w0biu15kj30op02q0ti.jpg)

图中的Designated Initializer标志说明，系统推荐你使用`initWithNibName:bundle:`方法创建UIViewController及其子类，不要用其他什么乱七八糟的方法创建对象。因此通常你会这么写: 

```objc
//MyViewController.h
//MyViewController.m is clean
@interface MyViewController : UIViewController
@end

//client
NSBundle *bundle = ...;
MyViewController *vc = [MyViewController alloc] initWithNibName:@"vcNib" bundle:bundle];
```

#更高逼格
如果想逼格高一点，自己给MyViewController创建一个新的指定初始化函数可以吗？可以！比如在初始化时，把作者名字也传进去。那你得这么写: 

```objc
//MyViewController.h
@interface MyViewController : UIViewController
@property (nonatomic, strong) NSString *author;
@end
- (instancetype)initWithNibName:(NSString *)nibName bundle:(NSBundle *)nibBundle author:(NSString *)author;

//MyViewController.m
@implementation MyViewController
- (instancetype)initWithNibName:(NSString *)nibName bundle:(NSBundle *)nibBundle author:(NSString *)author {
	if((self=[super initWithNibName:nibName bundle:nibBundle])) { //调用父类的Designated Initializer
		_author = author;
	}
	return self;
}

//自动继承父类的- (instancetype)initWithNibName: bundle:方法
@end
```

此时`- (instancetype)initWithNibName:bundle:author:`变成了:MyViewController的指定初始化函数。

`- (instancetype)initWithNibName:bundle:`则变成了MyViewController的便利初始化函数(Convenience Initializer/Secondary Initializer)。

也就是说，从此以后，我们应该使用`- (instancetype)initWithNibName:bundle:author:`来创建MyViewController对象。

#手痒怎么办
虽然`- (instancetype)initWithNibName:bundle:`已经被降级为便利初始化函数。但如果你哪天手痒，仍然使用它来创建MyViewController，就会造成author属性没有值了！如果你在指定初始化函数中做了额外的操作，也没有机会得到调用了(比如初始化了一些私有变量)！

为了避免这样的问题，我们应该把指定初始化函数看做是初始化的统一出口。我们需要重写所有便利初始化函数，让便利初始化函数在内部转调指定初始化函数。

```objc
//MyViewController.m
@implementation MyViewController

//指定初始化函数
- (instancetype)initWithNibName:(NSString *)nibName bundle:(NSBundle *)nibBundle author:(NSString *)author {
	if((self=[super initWithNibName:nibName bundle:nibBundle])) { //调用父类的Designated Initializer
		_author = author;
		_address = @"Kings Road"; //额外操作，初始化私有变量
	}
	return self;
}

//重写从父类继承过来的便利初始化函数
- (instancetype)initWithNibName:(NSString *)nibName bundle:(NSBundle *)nibBundle {
	if((self=[self initWithNibName:nibName bundle:nibBundle author:@"anonymous"])) { //调用MyViewController的Designated Initializer
		_address = @"Kings Road"; //额外操作，初始化私有变量
	}
	return self;
}
@end
```

搞掂！此时即使你调用便利初始化函数，在内部也会转调指定初始化函数，确保初始化过程的完整性。以后我们只需保证指定初始化函数是正确的就行了。

#自己创建一个便利初始化函数
```objc
//MyViewController.m
@implementation MyViewController
//指定初始化函数
...

//自己创建的便利初始化函数
//这次我们不要bundle了..真的好吗..只是举个栗子
- (instancetype)initWithNibName:(NSString *)nibName {
    if((self=[self initWithNibName:nibName bundle:nil author:nil])) { //调用MyViewController的Designated Initializer
        
    }
    return self;
}
@end
```

其实做法和之前的方式是完全一样的: 只要确保便利初始化函数在内部转调指定初始化函数就可以了。

#只能有一个Designated Initializer吗？
![](http://ww1.sinaimg.cn/mw690/0065Y1avgw1f3w1tr0qdaj30pt094gqz.jpg)
这是API中关于`initWithNibName:bundle:`方法的讨论。高亮部分说到: 如果使用storyboard创建VC，则系统自动调用`initWithCoder:`创建VC，而不是使用`initWithNibName:bundle:`创建VC。

...这脸打得好疼。之前还说推荐使用`initWithNibName:bundle:`，为毛系统自己却用`initWithCoder:`创建VC。

公布答案: 这两个方法其实都是UIViewController的指定初始化函数，但隶属于不同的初始化数据源。`initWithNibName:bundle:`使用nibName的名字和bundle来创建VC。而`initWithCoder:`使用一个已被序列化的VC对象作为参数，并将VC对象反序列化。

> 科普: 序列化是指把一个对象实例转为二进制数据，然后保存硬盘文件中。反序列化则把一个二进制数据读取出来，重新解析为一个对象。这和平常直接用文本值初始化对象是不一样的。
> 
> 在objc中，如果希望对象支持序列化和反序列化，则必须实现NSCoding接口，并实现其中的两个方法。

**结论: 在同一个类中，我们可以拥有多个Designated Initializer, 每一个Designated Initializer负责使用一种类型的数据源进行初始化。**

#总结
1. Designated Initializer表示指定初始化函数(唯一的初始化出口)。
2. 如果需要创建新的指定初始化函数，则**新的指定初始化函数**在内部调用父类的指定初始化函数。而**旧的指定初始化函数**降级为便利初始化函数，你必须重写便利初始化函数，并在里面转调新的指定初始化函数。
3. 如果需要自己创建便利初始化函数，则在里面必须转调指定初始化函数。(和第二点一样)
4. 一个类可以拥有多个指定初始化函数。你需要根据初始化数据源选择其中一个指定初始化函数。(不建议为一种初始化数据源创建多个指定初始化函数)。

#References
* [iOS: 聊聊 Designated Initializer（指定初始化函数）](http://www.cnblogs.com/smileEvday/p/designated_initializer.html)
* [正确编写Designated Initializer的几个原则](http://www.cocoachina.com/programmer/20140421/8204.html)









