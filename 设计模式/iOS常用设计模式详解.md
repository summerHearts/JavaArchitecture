## 1、设计原则

![](https://upload-images.jianshu.io/upload_images/325120-9111dcfc07dee09f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

前面五种被称为面向对象设计中常用的SOLID原则。

## 1、责任链模式

- 责任链模式就是为一个请求或者一个动作创建一个接收者对象的链,这条链上的每一个对象都可以去响应和处理这个请求和动作,把发送者和接收者进行解耦,在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

举例iOS事件分发和相应：

1.在iOS视图树形结构中找到最终的接收者，也就是触摸事件发生的那个最上层的View上，这一过程称为hit-testing(测试命中),通过一层层的遍历找到最终的命中视图称为hit-test view.
UIView中有两个方法用来确定hit-test view.

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event; 
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;
```

![](https://upload-images.jianshu.io/upload_images/696136-52b8039b06194a89.png?imageMogr2/auto-orient/strip%7CimageView2/2)

从apple官方文档里面的截图，这里所有的显示的View都是加载到主window上，假设我们触摸到屏幕上ViewD的区域,当我们没有重载UIView的hitTest:withEvent:和pointInside:withEvent:这两个方法时，系统默认的处理如下：

- keyWindow调用pointInside:withEvent:判断触摸点是否在其frame范围内，返回Yes，遍历keyWindow的subView->ViewA.

- ViewA调用pointInside:withEvent:判断触摸点是否在其frame范围内，返回Yes，遍历ViewA的subView->ViewB、ViewC(关于ViewB和ViewC先执行哪个，是根据ViewA添加子控件的先后顺序，总是先执行后添加的subView.假设添加ViewB后添加ViewC)
- ViewC调用pointInside:withEvent:判断触摸点是否在其frame范围内，返回Yes，遍历ViewC的subView->ViewD、ViewE
- ViewE调用pointInside:withEvent:判断触摸点是否在其frame范围内，返回NO,ViewE的hitTest:withEvent:返回nil(如果是先执行ViewB的情况，假设ViewB还有子节点subView,由于ViewB的pointInside:withEvent:返回NO,ViewB的hitTest:withEvent:`直接返回nil是不会再去遍历ViewB的子节点的)
- ViewD调用pointInside:withEvent:判断触摸点是否在其frame范围内，返回Yes并且没有子节点subView,ViewD的hitTest:withEvent:返回ViewD本身，即为最终的hit-test view(不会再遍历ViewB)iewB

响应事件。说明一下，对于触摸事件来说，无论View是否处理事件，即使是application通过[application beginIgnoringInteractionEvents]忽略了触摸事件，上面hit-testing的过程依然存在，它只影响第二个步骤事件响应的过程。下面我们将介绍iOS响应者链条(Responder chain)

![](https://upload-images.jianshu.io/upload_images/696136-2e0a9a2ae8458e0c.png?imageMogr2/auto-orient/strip%7CimageView2/2)

从官方文档里面截取的一张关于响应者链条的截图。我们先看上图左边的情况：标注为①的地方即为步骤1找到的hit-test view 它作为第一响应者来响应这个事件，如果该view没有通过重写或者封装touch系列方法来处理该事件，默认touch的实现就是调用父类的touch方法，将事件传递下去。在这里由1->传递到它的父类2，2是控制器的根view,->传递到vc控制器->传递到窗口window->传递到application

再看上图右边的情况：标注为①的地方即为步骤1找到的hit-test view,同时它是控制器的根view并且还有父视图，事件传递到控制器->再传递到父视->传递到控制器，再传递到父视图窗口->application。其实上图左边部分也可以理解为窗口是控制器根视图的父视图。如果整个响应者链条结束，都没有对事件做处理，那么该事件会被丢弃。

扩展说明：

如何控制控件的点击区域

![](https://upload-images.jianshu.io/upload_images/325120-5f4ee665c225bd28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

## 1.1、责任链模式实战

![](https://upload-images.jianshu.io/upload_images/325120-3c958affaab2fceb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](https://upload-images.jianshu.io/upload_images/325120-e60296345dfb19db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

我们假设第一个对象不处理，然后让第二个去处理。如上所述。

在举个例子

```
#import <Foundation/Foundation.h>

@protocol Leave <NSObject>

- (void)handleLeaveApplication:(NSUInteger)dayCount;

@end

@interface Manager : NSObject<Leave>

@property (nonatomic, strong) id<Leave> superior;

@end

@implementation Manager

- (void)handleLeaveApplication:(NSUInteger)dayCount {
}

@end

@interface PM : Manager

@end

@implementation PM

- (void)handleLeaveApplication:(NSUInteger)dayCount {

    if (dayCount < 10) {
        NSLog(@"dayCount:%ld----PM:请假跟女朋友去旅游啊，调试完这个bug就走吧。过来你看，你的程序报错了：\"找不到对象\"", dayCount);
    } else {
        if (self.superior != nil) {
            NSLog(@"dayCount:%ld----PM:请假跟女朋友去旅游啊，我没权利批假，去问一下我的老大吧", dayCount);
            [self.superior handleLeaveApplication:dayCount];
        }
    }
}

@end

@interface CTO : Manager

@end

@implementation CTO

- (void)handleLeaveApplication:(NSUInteger)dayCount {
    if (dayCount < 10) {
        NSLog(@"CTO:我很忙，这种小事别烦我");
        return;
    }
    if (dayCount < 20) {
        NSLog(@"dayCount:%ld----CTO:又请假相亲啊，去吧去吧~", dayCount);
    } else {
        if (self.superior != nil) {
            NSLog(@"dayCount:%ld----CTO:又请假相亲啊，我没权利批假，去问一下我的老大吧~", dayCount);
            [self.superior handleLeaveApplication:dayCount];
        }
    }
}

@end

@interface CEO : Manager

@end

@implementation CEO

- (void)handleLeaveApplication:(NSUInteger)dayCount {
    if (dayCount < 20) {
        NSLog(@"CEO:我很忙去找你上司");
        return;
    }
    if (dayCount < 30) {
        NSLog(@"dayCount:%ld----CEO:Bug都写完了吗？那就去吧", dayCount);
    } else {
        NSLog(@"dayCount:%ld----CEO:世界那么大你是不是也想出去看看？回去写你的Bug", dayCount);
    }
}

@end

void chainOfResponsibility() {
    CEO *ceo = [[CEO alloc] init];
    CTO *cto = [[CTO alloc] init];
    PM *pm = [[PM alloc] init];
    pm.superior = cto;
    cto.superior = ceo;
    
    NSArray *leaveApplicationArray = @[@"1", @"16", @"25", @"31"];
    for (NSString *string in leaveApplicationArray) {
        [pm handleLeaveApplication:[string integerValue]];
    }
}
```

- 优点： 1.低耦合：将请求和处理分开，请求者可以不用知道是谁处理的。2.新增和修改新的处理类比较容易

- 缺点： 每个请求都是从链头遍历到链尾，如果链比较长会产生一定的性能问题，调试起来也比较麻烦。

## 2、桥接模式

```

 /**
  *  桥接模式：将抽象部分与它的实现部分分离，使它们都可以独立地变化。
  *  在本例中，AbstractRemoteControl是抽象部分，TVProtocol是其实现部分。
  *  抽象部分与实现部分通过detectTVFunction方法来连接。
  */
  
@protocol TVProtocol <NSObject>

@required

- (void)switchChannel; // 切换频道

- (void)adjustVolume;  // 调节音量

- (void)powerSwitch;   // 电源开关

@end

---------------------------------------------------------------
#import "TVProtocol.h"
@interface AbstractRemoteControl : NSObject

@property (nonatomic, weak) id<TVProtocol> tvProtocol;

- (void)detectTVFunction;

@end

---------------------------------------------------------------
@implementation AbstractRemoteControl

- (void)detectTVFunction {
    NSLog(@"检测电视机具备的功能，由子类来进行实现");
}

@end

---------------------------------------------------------------
@interface ConcreteRemoteControl : AbstractRemoteControl

// 重写该方法
- (void)detectTVFunction;

@end

---------------------------------------------------------------
@implementation ConcreteRemoteControl

- (void)detectTVFunction {
    [self.tvProtocol switchChannel];
    [self.tvProtocol adjustVolume];
    [self.tvProtocol powerSwitch];
}

@end

---------------------------------------------------------------
#import "TVProtocol.h"
@interface AbstractTV : NSObject <TVProtocol>

@end

---------------------------------------------------------------
@implementation AbstractTV

- (void)switchChannel {
    NSLog(@"切换频道，由具体的子类来实现");
}

- (void)adjustVolume {
    NSLog(@"调节音量，由具体的子类来实现");
}

- (void)powerSwitch {
    NSLog(@"电源开关，由具体的子类来实现");
}

@end

---------------------------------------------------------------

@interface TVA : AbstractTV

// 重写这三个方法
- (void)switchChannel;
- (void)adjustVolume;
- (void)powerSwitch;

@end

---------------------------------------------------------------
@implementation TVA

- (void)switchChannel {
    NSLog(@"电视机A 具备了切换频道的功能");
}

- (void)adjustVolume {
    NSLog(@"电视机A 具备了调节音量的功能");
}

- (void)powerSwitch {
    NSLog(@"电视机A 具备了电源开关的功能");
}

@end
---------------------------------------------------------------
@interface TVB : AbstractTV

// 重写这三个方法
- (void)switchChannel;
- (void)adjustVolume;
- (void)powerSwitch;

@end

---------------------------------------------------------------
@implementation TVB

- (void)switchChannel {
    NSLog(@"电视机B 具备了切换频道的功能");
}

- (void)adjustVolume {
    NSLog(@"电视机B 具备了调节音量的功能");
}

- (void)powerSwitch {
    NSLog(@"电视机B 具备了电源开关的功能");
}

@end

---------------------------------------------------------------

AbstractRemoteControl *remoteControl = [[ConcreteRemoteControl alloc] init];
TVProtocol tvProtocol = [[TVA alloc] init];
remoteControl.tvProtocol = tvProtocol;

[remoteControl detectTVFunction];

NSLog(@"///////////////////////////////");

tvProtocol = [[TVB alloc] init];
remoteControl.tvProtocol = tvProtocol;
[remoteControl detectTVFunction];
```
桥接模式在UIKit和Foundation中的使用场景,比如，同一界面中在不同的用户等级（如游客模式、普通用户、VIP）时，展示不同的板块

## 3、适配器模式

- 项目比较老了，又需要修改。

```
@interface Target : NSObject

- (void)operation;

@end
---------------------------------------------------------------
@implementation Target

- (void)operation{
    // 原有的具体业务逻辑
}
@end
---------------------------------------------------------------
// 适配对象
@interface CoolTarget : NSObject

// 被适配对象
@property (nonatomic, strong) Target *target;

// 对原有方法包装
- (void)request;

@end
---------------------------------------------------------------
@implementation CoolTarget

- (void)request{
    // 额外处理
    
    [self.target operation];
    
    // 额外处理
}
@end
```

## 4、单例模式

```
@interface Mooc : NSObject

+ (id)sharedInstance;

@end
---------------------------------------------------------------
@implementation Mooc

+ (id)sharedInstance{
    // 静态局部变量
    static Mooc *instance = nil;
    
    // 通过dispatch_once方式 确保instance在多线程环境下只被创建一次
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        // 创建实例
        instance = [[super allocWithZone:NULL] init];
    });
    return instance;
}

// 重写方法【必不可少】
+ (id)allocWithZone:(struct _NSZone *)zone{
    return [self sharedInstance];
}

// 重写方法【必不可少】
- (id)copyWithZone:(nullable NSZone *)zone{
    return self;
}

@end
```

## 4、命令模式

- 1、定义: 命令模式将请求封装成对象，从而可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销和恢复的操作。

- 2、 使用场景： 在某些场合，比如要对行为进行"记录、撤销/重做、事务"等处理的时候。
- 3、具体实现： YTKNetwork就是用的命令模式，推荐大家学习。这里我举了一个吃饭点菜的例子，具体请点击这里查看
- 4、优点： 1.类间解耦：调用者与接收者之间没有任何依赖关系。2.扩展性良好：新的命令可以很容易添加到系统中去。
- 5、缺点： 使用命令模式可能会导致系统有过多的具体命令类。

```
@class Command;
typedef void(^CommandCompletionCallBack)(Command* cmd);

@interface Command : NSObject
@property (nonatomic, copy) CommandCompletionCallBack completion;

- (void)execute;
- (void)cancel;

- (void)done;

@end
---------------------------------------------------------------
@implementation Command

- (void)execute{
    
    //override to subclass;
    
    [self done];
}

- (void)cancel{
    
    self.completion = nil;
}

- (void)done{
    dispatch_async(dispatch_get_main_queue(), ^{
        
        if (_completion) {
            _completion(self);
        }
        
        //释放
        self.completion = nil;
        
        [[CommandManager sharedInstance].arrayCommands removeObject:self];
    });
}

@end
---------------------------------------------------------------
@interface CommandManager : NSObject
// 命令管理容器
@property (nonatomic, strong) NSMutableArray <Command*> *arrayCommands;

// 命令管理者以单例方式呈现
+ (instancetype)sharedInstance;

// 执行命令
+ (void)executeCommand:(Command *)cmd completion:(CommandCompletionCallBack)completion;

// 取消命令
+ (void)cancelCommand:(Command *)cmd;

@end
---------------------------------------------------------------
@implementation CommandManager

// 命令管理者以单例方式呈现
+ (instancetype)sharedInstance{
    static CommandManager *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[super allocWithZone:NULL] init];
    });
    return instance;
}

// 【必不可少】
+ (id)allocWithZone:(struct _NSZone *)zone{
    return [self sharedInstance];
}

// 【必不可少】
- (id)copyWithZone:(nullable NSZone *)zone{
    return self;
}

// 初始化方法
- (id)init{
    self = [super init];
    if (self) {
        // 初始化命令容器
        _arrayCommands = [NSMutableArray array];
    }
    return self;
}

+ (void)executeCommand:(Command *)cmd completion:(CommandCompletionCallBack)completion{
    if (cmd) {
        // 如果命令正在执行不做处理，否则添加并执行命令
        if (![self _isExecutingCommand:cmd]) {
            // 添加到命令容器当中
            [[[self sharedInstance] arrayCommands] addObject:cmd];
            // 设置命令执行完成的回调
            cmd.completion = completion;
            //执行命令
            [cmd execute];
        }
    }
}

// 取消命令
+ (void)cancelCommand:(Command *)cmd{
    if (cmd) {
        // 从命令容器当中移除
        [[[self sharedInstance] arrayCommands] removeObject:cmd];
        // 取消命令执行
        [cmd cancel];
    }
}

// 判断当前命令是否正在执行
+ (BOOL)_isExecutingCommand:(Command *)cmd{
    if (cmd) {
        NSArray *cmds = [[self sharedInstance] arrayCommands];
        for (Command *aCmd in cmds) {
            // 当前命令正在执行
            if (cmd == aCmd) {
                return YES;
            }
        }
    }
    return NO;
}

@end
```

再举一个点餐的例子

执行指令

```
@protocol CommandProtocol <NSObject>

@required

- (void)execute;

@end
```

服务员
 
```

@interface Waiter : NSObject

/**
 点菜
 */
- (void)addOrder:(id <CommandProtocol>)command;

/**
 全点好了
 */
- (void)submmitOrder;

/**
 取消菜
 */
- (void)cancleOrder:(id <CommandProtocol>)command;

@end

@interface Waiter()

@property (nonatomic, strong) NSMutableArray *commandQueue;

@end

@implementation Waiter

- (instancetype)init {
    if (self = [super init]) {
        self.commandQueue = [[NSMutableArray alloc] init];
    }
    return self;
}

- (void)addOrder:(id <CommandProtocol>)command {
    [_commandQueue addObject:command];
}

- (void)submmitOrder {
    for (id <CommandProtocol> command in _commandQueue) {
        [command execute];
    }
    [_commandQueue removeAllObjects];
}

- (void)cancleOrder:(id <CommandProtocol>)command {
    if ([_commandQueue containsObject:command]) {
        [_commandQueue removeObject:command];
        NSLog(@"取消成功");
    } else {
        NSLog(@"已经不可以取消了");
    }
}

@end
```

厨子

```
@interface Cook : NSObject

/**
 制作龙虾
 */
- (void)cookLobster;

/**
 制作鲍鱼
 */
- (void)cookAbalone;

@end

@implementation Cook

- (void)cookLobster {
    NSLog(@"制作好了龙虾");
}
- (void)cookAbalone {
    NSLog(@"制作好了鲍鱼");
}

@end
```

```
@interface Command : NSObject<CommandProtocol>

@property (nonatomic, strong, readonly) Cook *cook;

- (instancetype)initWithReceiver:(Cook *)cook;

@end

@implementation Command

- (void)execute {
}

- (void)cancleCommand {
}

- (instancetype)initWithReceiver:(Cook *)cook {
    if (self = [super init]) {
        _cook = cook;
    }
    return self;
}

@end
```
龙虾
 
```

@interface LobsterCommand : Command

@end

@implementation LobsterCommand

- (void)execute {
    [self.cook cookLobster];
}

@end
```
鲍鱼

```
@interface AbaloneCommand : Command

@end

@implementation AbaloneCommand

- (void)execute {
    [self.cook cookAbalone];
}

@end
```
## 5、中介者模式（Mediator）

- 1、定义: 中介者模式就是用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

- 2、 使用场景： 多个类相互依赖，形成了网状结构的时候可以考虑使用中介者模式。
- 3、具体实现： 一个聊天室的例子
- 4、优点： 1.解耦：通过中介者模式，我们可以将复杂关系的网状结构变成结构简单的以中介者为核心的星形结构，每个对象不再和它与之关联的对象直接发生相互作用，而是通过中介者对象来另一个对象发生相互作用。2.降低了类的复杂度，将一对多转化成了一对一。
- 5、缺点：中介者模式在某些情况会膨胀得很大，而且逻辑复杂，中介类越多越复杂，越难以维护。
- 6、注意事项: 类之间的依赖关系是必然存在的，所以不一定有多个依赖关系的时候就考虑使用中介者模式。中介者模式适用于多个对象之间的紧密耦合的情况，紧密耦合的定义标准是：在类图中出现了蜘蛛网状结构，这种情况就要考虑使用中介者模式，中介者模式可以把蜘蛛网梳理成星型结构，使原本复杂混乱的关系变得清晰简单。

```
@protocol MediatorProtocol <NSObject>

- (void)showMessage:(NSString *)message;

@end
--------------------------------------------------------------------------------------

@interface ChatRoom : NSObject<MediatorProtocol>

- (void)showMessage:(NSString *)message userName:(NSString *)name;

@end

@implementation ChatRoom

- (void)showMessage:(NSString *)message {
    NSLog(@"%@\n",message);
}

- (void)showMessage:(NSString *)message userName:(NSString *)name {
    NSString *string = [NSString stringWithFormat:@"%@:%@", name, message];
    [self showMessage:string];
}

@end
--------------------------------------------------------------------------------------

@interface User : NSObject

- (instancetype)initWithName:(NSString *)name room:(ChatRoom *)room;

- (void)sendMessage:(NSString *)message;

@end

@interface User()

@property (nonatomic, copy) NSString *name;     ///< 用户昵称
@property (nonatomic, strong) ChatRoom *room;   ///< 当前聊天室

@end

@implementation User

- (instancetype)initWithName:(NSString *)name room:(ChatRoom *)room {
    if (self = [super init]) {
        _name = name;
        _room = room;
    }
    return self;
}

- (void)sendMessage:(NSString *)message {
    [_room showMessage:message userName:_name];
}

@end

void mediator() {
    ChatRoom *room = [[ChatRoom alloc] init];
    User *wuJun = [[User alloc] initWithName:@"吴军" room:room];
    User *me = [[User alloc] initWithName:@"SuperMario" room:room];
    [wuJun sendMessage:@"来自硅谷的第一封信"];
    [me sendMessage:@"谢谢，不做伪工作者"];
}
```

## 6、观察者模式（Observer）

- 1、定义: 定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

- 2、 使用场景： 一个对象的状态发生改变，所有的依赖对象都将得到通知的时候。
- 3、 具体实现： Objective-C中的通知以及KVO都是观察者模式的具体实现。这里举了一个找工作订阅的例子，具体请点击这里查看
- 4、 优点： 1.观察者和被观察者是抽象耦合的，扩展比较方便。2.建立一套触发机制。
- 5、 缺点： 1.如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。  2如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。 3.观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

```
@protocol SubjectProtocol <NSObject>

- (void)addObserver:(id <ObserverProtocol>)observer;
- (void)removeObserver:(id <ObserverProtocol>)observer;
- (void)notify;

@end
--------------------------------------------------------------------------------------
@interface JobProvider : NSObject<SubjectProtocol>

@end

@interface JobProvider()

@property (nonatomic, strong) NSMutableArray *observers;

@end

@implementation JobProvider

- (void)addObserver:(NSObject *)observer {
    [self.observers addObject:observer];
}
- (void)removeObserver:(NSObject *)observer {
    [self.observers removeObject:observer];
}
- (void)notify {
    for (id <ObserverProtocol> observer in self.observers) {
        [observer update];
    }
}

- (NSMutableArray *)observers {
    if (!_observers) {
        _observers = [[NSMutableArray alloc] init];
    }
    return _observers;
}

@end
--------------------------------------------------------------------------------------

@protocol ObserverProtocol <NSObject>

- (void)update;

@end
--------------------------------------------------------------------------------------

@interface JobHunter : NSObject<ObserverProtocol>

- (instancetype)initWithName:(NSString *)name;

@end

@interface JobHunter()

@property (nonatomic, copy) NSString *name;

@end

@implementation JobHunter

- (instancetype)initWithName:(NSString *)name {
    if (self = [super init]) {
        _name = name;
    }
    return self;
}


- (void)update {
    NSLog(@"%@:有一个新的职位更新啦",_name);
}

@end
```

## 7、备忘录模式（Memento）

- 1、定义: 在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就开奖对象恢复到原先保存的状态了。

- 2、使用场景： 需要存档的时候，比如说游戏中的存档。
- 3、具体实现： 打游戏时的存档，数据库的事务管理，SVN以及Git代码的版本控制系统等等都可以说成是备忘录模式的实例。这里我简单的举了一下例子，具体请点击这里查看
- 4、优点： 1.给用户提供了一种可以恢复状态的机制，可以使用户能够比较方便地回到某个历史的状态。 2.实现了信息的封装，使得用户不需要关心状态的保存细节。
- 5、缺点： 在一些场景下比较消耗资源。
- 6、注意事项: 不要在频繁建立备份的场景中使用备忘录模式，比如说在for循环中。


```
@interface EditorMemento : NSObject

@property (nonatomic, copy, readonly) NSArray *array;
- (instancetype)initWithArray:(NSArray *)array;

@end

@implementation EditorMemento

- (instancetype)initWithArray:(NSArray *)array {
    if (self = [super init]) {
        _array = array;
    }
    return self;
}

@end
--------------------------------------------------------------------------------------

@class EditorMemento;
@interface Editor : NSObject

- (void)insertContent:(NSString *)string;
- (EditorMemento *)save;
- (void)echo;
- (void)restore:(EditorMemento *)memento;

@end

@interface Editor()

@property (nonatomic, strong) NSMutableArray *array;
@property (nonatomic, strong) EditorMemento *memento;

@end

@implementation Editor

- (instancetype)init {
    if (self = [super init]) {
        _array = [[NSMutableArray alloc] init];
    }
    return self;
}

- (void)insertContent:(NSString *)string {
    [_array addObject:string];
}

- (EditorMemento *)save {
    return [[EditorMemento alloc] initWithArray:[_array copy]];
}

- (void)echo {
    for (NSString *string in _array) {
        NSLog(@"%@", string);
    }
}

- (void)restore:(EditorMemento *)memento {
    _array = [[NSMutableArray alloc] initWithArray:memento.array];
}

- (EditorMemento *)memento {
    if (!_memento) {
        _memento = [[EditorMemento alloc] initWithArray:[_array copy]];
    }
    return _memento;
}

@end
--------------------------------------------------------------------------------------
void memento() {
    Editor *editor = [[Editor alloc] init];
    [editor insertContent:@"总熬夜会带来三个问题"];
    [editor insertContent:@"第一:记忆力会明显下降"];
    [editor insertContent:@"第二:数数经常会数错"];
    EditorMemento *memento = [editor save];
    [editor insertContent:@"第四:记忆力会明显下降"];
    [editor echo];
    NSLog(@"//------------------------------------");
    [editor restore:memento];
    [editor echo];
}
```

## 8、策略模式（Strategy）：

- 1、定义: 定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。

- 2、 使用场景： 1.多个类只有在算法或行为上稍有不同的场景。2.算法需要自由切换的场景。3.需要屏蔽算法规则的场景。
- 3、 具体实现： 具体请点击这里查看
- 4、优点： 1.算法可以自由切换。 2.避免使用多重条件判断。 3.扩展性良好。
- 5、缺点：1.策略类会增多。 2.所有策略类都需要对外暴露。
- 6、注意事项: 如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题。

```
@protocol StrategyProtocol <NSObject>

+ (void)sort:(NSArray *)array;

@end
--------------------------------------------------------------------------------------

@interface BubbleSortStrategy : NSObject<StrategyProtocol>

@end

@implementation BubbleSortStrategy

+ (void)sort:(NSArray *)array {
    NSLog(@"Array's count=%ld, 使用了冒泡排序", array.count);
}

@end
--------------------------------------------------------------------------------------

@interface QuickSortStrategy : NSObject<StrategyProtocol>

@end

@implementation QuickSortStrategy

+ (void)sort:(NSArray *)array {
    NSLog(@"Array's count=%ld, 使用了快速排序", array.count);
}

@end
--------------------------------------------------------------------------------------

@interface Sort : NSObject<StrategyProtocol>

@end

@implementation Sort

+ (void)sort:(NSArray *)array {
    if (array.count > 5) {
        [QuickSortStrategy sort:array];
    } else {
        [BubbleSortStrategy sort:array];
    }
}

@end
```
## 9、访问者模式（Visitor）

- 1、定义: 访问者模式封装了一些作用于某种数据结构中的各元素操作，它可以在不改变数据结构的前提下定义作用于这些元素的新的操作。

- 2、使用场景： 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，使用访问者模式将这些封装到类中。
- 3、具体实现： 这里举了一个悲观的人和乐观的人对待不同事物的反应的实例，具体请点击这里查看，如果想增加Action就比较方便，但是如果想增加一个既悲观又乐观的人就有一点麻烦了。
- 4、优点： 1.符合单一职责原则。 2.优秀的扩展性。 3.灵活性高
- 5、缺点：1.具体元素对访问者公布细节，违反了迪米特原则。 2.具体元素变更比较困难。 3.违反了依赖倒置原则，依赖了具体类，没有依赖抽象。

```
@protocol PersonProtocol <NSObject>

- (void)accept:(id <ActionProtocol>)visitor;

@end
--------------------------------------------------------------------------------------

@interface PositivePerson : NSObject<PersonProtocol>

@end

@implementation PositivePerson

- (void)accept:(id <ActionProtocol>)visitor {
    [visitor positiveConclusion:[[PositivePerson alloc] init]];
}

@end
--------------------------------------------------------------------------------------

@interface NegativePerson : NSObject<PersonProtocol>

@end

@implementation NegativePerson

- (void)accept:(id <ActionProtocol>)visitor {
    [visitor negativeConclusion:[[NegativePerson alloc] init]];
}

@end

@protocol PersonProtocol;
--------------------------------------------------------------------------------------

@protocol ActionProtocol <NSObject>

- (void)positiveConclusion:(id <PersonProtocol>)positive;
- (void)negativeConclusion:(id <PersonProtocol>)negavite;

@end
--------------------------------------------------------------------------------------

@interface HalfCupWater : NSObject<ActionProtocol>

@end

@implementation HalfCupWater

- (void)positiveConclusion:(id <PersonProtocol>)positive {
    NSLog(@"乐观的人:还有半杯水可以喝");
}
- (void)negativeConclusion:(id <PersonProtocol>)negavite {
    NSLog(@"悲观的人:只剩半杯水了啊");
}

@end
--------------------------------------------------------------------------------------

@interface ObjectStructure : NSObject

- (void)add:(id <PersonProtocol>)person;
- (void)remove:(id <PersonProtocol>)person;
- (void)echo:(id <ActionProtocol>)action;

@end

@interface ObjectStructure()

@property (nonatomic, strong) NSMutableArray <PersonProtocol> *array;

@end

@implementation ObjectStructure

- (instancetype)init {
    if (self = [super init]) {
        _array = [[NSMutableArray <PersonProtocol> alloc] init];
    }
    return self;
}

- (void)add:(id <PersonProtocol>)person {
    [_array addObject:person];
}

- (void)remove:(id <PersonProtocol>)person {
    [_array removeObject:person];
}

- (void)echo:(id <ActionProtocol>)action {
    for (id <PersonProtocol>person in _array) {
        [person accept:action];
    }
}

@end
```
## 10、模板方法模式（TemplateMethod）

- 1、定义: 定义一个操作中的算法的框架，而降一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

- 2、使用场景： 1.多个子类有公有的方法，并且逻辑基本相同时。2.有重要、复杂的算法的时候，可以把核心算法设计为模板方法，周边的相关细节功能则由各个子类实现。
- 3、具体实现： 这里简单举了一个Android 和iOS项目的从code到发布的简易过程Demo
- 4、优点： 1.封装不变部分，扩展可变部分。 2.提取公共代码，便于维护。 3.行为由父类控制，子类实现。
- 5、缺点： 每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。

```
@protocol PublishProtocol <NSObject>

- (void)code;
- (void)test;
- (void)pack;
- (void)upload;

    @end
--------------------------------------------------------------------------------------

@interface Publish : NSObject<PublishProtocol>

- (void)publish;

@end

@implementation Publish

- (void)publish {
    [self code];
    [self test];
    [self pack];
    [self upload];
}

- (void)code {
}
- (void)test {
}
- (void)pack {
}
- (void)upload {
}
@end
--------------------------------------------------------------------------------------

@interface iOSPublish : Publish

@end

@implementation iOSPublish

- (void)code {
    NSLog(@"编写iOS代码");
}
- (void)test {
    NSLog(@"测试iOS代码");
}
- (void)pack {
    NSLog(@"xcode archive");
}
- (void)upload {
    NSLog(@"上传到AppStore");
}

@end
--------------------------------------------------------------------------------------

@interface AndroidPublish : Publish

@end

@implementation AndroidPublish

- (void)code {
    NSLog(@"编写Android代码");
}
- (void)test {
    NSLog(@"测试Android代码");
}
- (void)pack {
    NSLog(@"混淆代码，打包");
}
- (void)upload {
    NSLog(@"上传到各种应用商店");
}

@end
```

