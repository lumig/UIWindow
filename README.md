# UIWindow
详解UIWindow

1.首先我们先从官方文档入手吧。
[apple官方文档](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503-CH1-SW2)

UIWindow类是UIView的子类，可以看作是特殊的UIView。一般应用程序只有一个UIWindow对象，即使有多个UIWindow对象，也只有一个UIWindow可以接受到用户的触屏事件，但是这会影响时间的传递。

UIWindow初始化在appDeleDgate里面的 didFinishLaunchingWithOptions方法。

第一、UIWindow的作用

1.它是一个容器，用来盛放视图，它本身不做显示，是UIView视图做绘图操作。

2.它与视图控制器一起协作来呈现数据

3.它为视图和其他控制器对象的触摸起到关键性作用，分发事件给View。



第一、UIWindow的创建

iPhone应用程序通常只有一个UIWindow类的实例，该实例的创建如果是从nib文件创建，则通常有个属性变量，如果是用代码创建，则必须在创建时传入屏幕矩形，屏幕矩形可以通过UIScreen对象来取得，具体代码如下所示：

```
self.window = [[UIWindow alloc]initWithFrame:[[UIScreen mainScreen] bounds]];
```

第三、常用的加载view的方法

　1、addSubview
　
　　　直接将view通过addSubview方式添加到window中，程序负责维护view的生命周期以及刷新，但是并不会为去理会view对应的ViewController，因此采用这种方法将view添加到window以后，我们还要保持view对应的ViewController的有效性，不能过早释放。

```
　self.vc=[[ViewController alloc] initWithFrame:CGRectMake(0, 0, ScreenWidth, ScreenHeight)];
 UIView *view=self.vc.view;
[self.window addSubview:view];
```
　
　　　2、rootViewController
　　　
```
　　　self.vc = [[[AndyViewController alloc] init];
　　　self.window.rootViewController = self.vc;
```
　　　把创建的viewcontrolle付给rootViewController,，UIWindow将会自动将其view添加到当前window中，同时负责ViewController和view的生命周期的维护，防止其过早释放.
　　　

　　　第四、常用属性分析：
　　　
　　　1、windowLevel
　　　
　　　UIWindow有三个层级，分别是Normal，StatusBar，Alert，如下所示：
　　　
　　　
```
　　　　UIKIT_EXTERN const UIWindowLevel UIWindowLevelNormal; //默认，值为0
　　　　UIKIT_EXTERN const UIWindowLevel UIWindowLevelAlert; //值为2000
　　　　UIKIT_EXTERN const UIWindowLevel UIWindowLevelStatusBar ; // 值为1000
```
　　　
　　　如果想改变UIWindow的层级关系，可以通过一下代码：
　　　
```
　　　self.window.windowLevel=UIWindowLevelAlert;
```
　　　此时如果显示UIAlertView，我们发现他们会同时突出显示，如果self.window是normal级别，那别只有alertView会突出显示
　　　
　　　打印输出他们三个这三个层级的值我们发现从左到右依次是0，1000，2000，也就是说Normal级别是最低的，StatusBar处于中等水平，Alert级别最高。而通常我们的程序的界面都是处于Normal这个级别上的，系统顶部的状态栏应该是处于StatusBar级别，UIActionSheet和UIAlertView这些通常都是用来中断正常流程，提醒用户等操作，因此位于Alert级别。
　　　根据window显示级别优先的原则，级别高的会显示在上面，级别低的在下面，我们程序正常显示的view位于最底层，

　　　　　　
　　　常用一下四个方法来操作
```
　　　– makeKeyAndVisible
　　　– becomeKeyWindow
　　　– makeKeyWindow
　　　– resignKeyWindow
　
```
　　　　
　　　　
　四个关于window变化的通知
　　　　
```
　　UIKIT_EXTERN NSString *const UIWindowDidBecomeVisibleNotification; // 当window激活时并展示在界面的时候触发，返回空
　　UIKIT_EXTERN NSString *const UIWindowDidBecomeHiddenNotification;  // 当window隐藏的时候触发，暂时没有实际测，返回空
　　UIKIT_EXTERN NSString *const UIWindowDidBecomeKeyNotification;     // 当window被设置为keyWindow时触发，返回空
　　UIKIT_EXTERN NSString *const UIWindowDidResignKeyNotification;     // 当window的key位置被取代时触发，返回空
```
　　　　　　　
程序启动然后点击弹框出来 keywindow的变化

这四个通知对象中的object都代表当前已显示（隐藏），已变成keyWindow（非keyWindow）的window对象，其中的userInfo则是空的。于是我们可以注册这个四个消息，再打印信息来观察keyWindow的变化以及window的显示，隐藏的变动 . 变成keywindow 的流程是这样的（默认的window -->点击弹出AlertView）

1.程序默认的window先显示出来
2.默认的window再变成keywindow
3.AlertView 的window显示出来
4.默认的window变成keywindow
5.最终AlertView的window变成keywindow
　　　　　　　
根据测试我们同时可以知道默认的window的level是0，即normal级别；AlertView的window的level是1996，比Alert级别稍微低了一点儿。同时我们可以看出ActionSheet的window的level是2001； 键盘window 的level是最高的在一切之上（我测试的是不管level 设置为多少都在键盘window 的下面）
　　　　　　　
当我们点击ActionSheet cancel的时候，我们会看到流程
　　　　　　
1.首先ActionSheet 的window变成非keyWindow
2.程序默认的window变成keywindow
3.ActionSheet 的window隐藏掉
　　　　　　　
弹出AlertView和ActionSheet的时候系统会帮你改变keyWindow  但是当弹出键盘的时候keyWindow是不变的！
　　　　　　　
下面有说keyWindow是用来接收键盘以及非触摸类的消息（文档有误 是指点击事件等 不是keywindow 也是可以接受事件的消息的）
　　　　　　　
keyWindow

是唯一一个可以接受响应的Window，在一个应用程序中只有唯一一个keyWindow
　　　　
当前app可以打开的多个window 如系统状态栏其实就是一个window ,程序启动的时候创建的默认的window ，弹出键盘也是一个window ，alterView 弹框也是window 。但是keyWindow只有一个 ，一般情况下就是我们程序启动时设置的默认的window
　　　　　　　
官方文档中是这样解释的 “The key window is the one that is designated to receive keyboard and other non-touch related events. Only one window at a time may be the key window." 翻译过来就是说，keyWindow是指定的用来接收键盘以及非触摸类的消息，而且程序中每一个时刻只能有一个window是keyWindow。

文档有误：app可以打开的多个window 每个里面都加入UITextField  和点击事件 发现 都可以处理事件和接受键盘消息
　　　　　　　
问题一：一个应用程序只能有一个主窗口，如果程序中创建了两个Window，那么谁是主窗口？
　　　　　　　
①iOS 7 以后，主窗口和次窗口是没有区别的
②iOS 7 之前，如果后面的窗口设置为主窗口，会把之前设置的主窗口覆盖掉
　　　　　　　
问题二：只有主窗口才能响应键盘的输入事件？

在ios9.3的模拟器中，主窗口和非主窗口中的输入框都能输入文字，但是在ios6.1的模拟器中，
非主窗口的输入框不能输入文字。
　　　　　　　
获取keyWindow的方式
UIWindow *keyWindow = [UIApplication sharedApplication].keyWindow;
UIViewController *rootViewController = keyWindow.rootViewController;
　　　　　　　
注意：keyWindow不是一成不变的，当你创建alertView或者ActionSheet的时候，它们所在的window会变成keyWindow。也就是说系统默认创建的window首先变成keywindow，而当弹框的时候，alertView所在的window变成keywindow，默认的keywindow变成非keywindow。
　　　　　　　
@property(nonatomic,readonly) NSArray  *windows;
　　　　　　　
在windows数组里面，window是根据windowLevel来排列的，最后一个覆盖在最上面。这里的windows数组不包括系统提供的window,比如说状态栏就是在一个系统创建的window里面。


[UIApplication sharedApplication].delegate.window和 [UIApplication sharedApplication].keyWindow的区别

它们在iOS上可能相同。当它们不同时，通常您会呈现除应用程序代理的主窗口之外的另一个窗口。你的应用程序可以有很多窗口，但只有keyWindow窗口可以在屏幕上看到并接收事件（例如可见UIAlert，当接收事件时它是关键窗口）参考：https：//developer.apple.com/library /content/documentation/WindowsViews/Conceptual/WindowAndScreenGuide/WindowScreenRolesinApp/WindowScreenRolesinApp.html
从文档：
- 为[[[UIApplication sharedApplication] delegate] window];

展示故事板时使用的窗口。此属性包含用于在设备主屏幕上呈现应用程序可视内容的窗口。

即这是window您在appDelegate.h文件中的属性

对于 [[UIApplication sharedApplication] keyWindow];
该属性在最近发送makeKeyAndVisible消息的windows数组中保存UIWindow对象。

你在iOS里发送makeKeyAndVisible你的appDelegate.m内部
application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions

你创建的appDelegate窗口是keyWindow。通常银行应用程序会在应用程序放在后台时切换关键窗口，以在主页按钮被双击时保护用户敏感信息，并在应用程序处于前景时切换回主委托窗口。


UIWindow的方法与属性

```
NS_CLASS_AVAILABLE_IOS(2_0) @interface UIWindow : UIView
//window的屏幕,默认是 [UIScreen mainScreen] ,不能更改，否则没有界面
@property(nonatomic,strong) UIScreen *screen NS_AVAILABLE_IOS(3_2);
//window的视图层级,默认是0.0
@property(nonatomic) UIWindowLevel windowLevel;
//是否是keyWindow
@property(nonatomic,readonly,getter=isKeyWindow) BOOL keyWindow;
//该方法不应该被手动调用，当window变为keyWindow时会被自动调用来通知window。可以继承UIWindow重写此方法来实现功能
- (void)becomeKeyWindow;
// 该方法不应该被手动调用，当window不再是keyWindow时(例如其他window实例调用了- makeKeyWindow或- makeKeyAndVisible方法)会被自动调用来通知window。可以继承UIWindow重写此方法来实现功能。
- (void)resignKeyWindow;
//该方法使window变为keyWindow，但不影响显示状态
- (void)makeKeyWindow;

//显示主window并将window设为key并显示，该方法让window显示并变为keyWindow。调用该方法会让window排在其level组中的最上面。如果只想改变window的显示而不影响keyWindow状态，可以直接设置window的hidden属性为NO。
- (void)makeKeyAndVisible;
//根控制器
@property(nullable, nonatomic,strong) UIViewController *rootViewController NS_AVAILABLE_IOS(4_0);
//UIApplication调用window的该方法给window分发事件，window再将事件分发到合适的目标，比如将触摸事件分发到真正触摸的view上。可以直接调用该方法分发自定义事件。
- (void)sendEvent:(UIEvent *)event;
// 把该window中的一个坐标转换成在目标window中时的坐标值
- (CGPoint)convertPoint:(CGPoint)point toWindow:(nullable UIWindow *)window;
// 把目标window中的一个坐标转换成在该window中时的坐标值
- (CGPoint)convertPoint:(CGPoint)point fromWindow:(nullable UIWindow *)window;
// 把该window中的一个矩阵转换成在目标window中时的矩阵值
- (CGRect)convertRect:(CGRect)rect toWindow:(nullable UIWindow *)window;
// 把目标window中的一个矩阵转换成在该window中时的矩阵值
- (CGRect)convertRect:(CGRect)rect fromWindow:(nullable UIWindow *)window;

　　```











