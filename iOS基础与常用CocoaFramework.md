# iOS基础与常用CocoaFramework

[toc]

## OC特性

### category		

分类，是一种编译时的手段，允许我们通过给一个类添加方法来扩充它（但是通过category不能添加新的实例变量）。

### extension	

扩展，推荐在自定义类的.m文件中使用扩展,这样就能保证良好的代码封装性，避免把私有接口暴露给外面。例如：

    @interface MyClass : NSObject
    - (float)value;
    @end
    
     MyClass.m
    @interface MyClass () { //注意此处：扩展
        float value;
    }
    - (void)setValue:(float)newValue;
    @end
     
    @implementation MyClass
    - (float)value {
        return value;
    }
    - (void)setValue:(float)newValue {
        value = newValue;
    }
    @end


### 继承

**继承**可以修改父类的已经存在方法，比如初始化方法init，重写它之后你可以得到你想要的初始化对象；而**分类**则可以扩展原有类的方法，比如你想让UIImageView可以从网络设置其image对象，则可以写一个分类，扩展这个原类中没有的方法（或者说功能），但是**不要重写原有类方法** ，CocoaFramework有很多是用Category实现的，重写之后，会导致在Runtime的时候，只有一个方法会被执行，而哪个会被执行是undefined。


### copy和mutablecopy

1. 系统的非容器类对象 指NSString,NSNumber等等一类的对象
对一不可变对象复制，copy是指针复制（浅拷贝）和mutableCopy就是对象复制（深拷贝）。如果是对可变对象复制，都是深拷贝，但是copy返回的对象是不可变的。copy返回不可变对象，mutablecopy返回可变对象。
2. 系统的容器类对象 指NSArray，NSDictionary等。对于容器类本身，上面讨论的结论也是适用的，其元素对象始终是指针复制。如果需要元素对象也是对象复制，就需要实现深拷贝。

### runtime

#### runtime关联key:value动态传参

    objc_setAssociatedObject(btnPostion, @"webTitle", title, OBJC_ASSOCIATION_RETAIN_NONATOMIC);//runtime关联key:value动态传参
    objc_getAssociatedObject(btnPostion, @"webTitle");//获取关联参数


## 常用函数与宏

### 四舍五入

rate = round(c)   不保留小数点
rate = round(c * 100) / 100  保留两位小数点

### 定义nonnull

NS_ASSUME_NONNULL_BEGIN
之间的所有简单指针对象都被假定为nonnull只需要去指定那些nullable的指针
NS_ASSUME_NONNULL_END


### ceil

**无限大取整**
ceil(4.35) 返回的是double
ceilf(4.35) 返回的是float类型5.0000000，做一个强制类型转换就得到5了。

> extern float ceilf(float);
> extern double ceil(double);
> extern long double ceill(long double);



## 单例

    + (UtilityHandler *)shareHandler {
        static UtilityHandler *_shareHandler = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            _shareHandler = [[UtilityHandler alloc]init];
        });
        return _shareHandler;
    }
    
    

## CoreFoundation

### CFStringTransform

iOS在CoreFoundation中提供了CFStringTransform函数，但在Foundation中却没有相对应的方法。它的定义如下：

    Boolean CFStringTransform(CFMutableStringRef string, CFRange *range, CFStringRef transform, Boolean reverse);
    
其中string参数是要转换的string，比如要转换的中文，同时它是mutable的，因此也直接作为最终转换后的字符串。range是要转换的范围，同时输出转换后改变的范围，如果为NULL，视为全部转换。transform可以指定要进行什么样的转换，这里可以指定多种语言的拼写转换。reverse指定该转换是否必须是可逆向转换的。
如果要进行汉字到拼音的转换，我们只需要将transform设定为`kCFStringTransformMandarinLatin`或者`kCFStringTransformToLatin`（它也可适用于非汉字字符串）:

    CFMutableStringRef string = CFStringCreateMutableCopy(NULL, 0, CFSTR("中国"));
    CFStringTransform(string, NULL, kCFStringTransformMandarinLatin, NO);
    NSLog(@"%@", string);

这段代码将输出： zhōng guó

可以看出，CFStringTransform正确的输出了“中国”的拼音，而且还带上了音标。有时候我们不需要音标怎么办？还好CFStringTransform同时提供了将音标字母转换为普通字母的方法kCFStringTransformStripDiacritics。我们在上面的代码基础上再加上这个：

    CFStringTransform(string, NULL, kCFStringTransformStripDiacritics, NO);
    
那么最终将输出： zhong guo

下面的处理也可以去掉音标

    NSString *pinyinString = (NSMutableString *)[mutableString stringByFoldingWithOptions:NSDiacriticInsensitiveSearch locale:[NSLocale currentLocale]];



## CoreAnimation





### CALayer

anchorPoint是layer中相对位置，即（0，0）到（1，1）
position是实际位置。
roundView.layer.anchorPoint = CGPointMake(0.5, 0.5);
roundView.layer.position = CGPointMake(75, 75);
iOS的原点位置是左上角开始，而MacOS中是左下角开始。

画图(导致过崩溃，具体原因忘了。。。)

    CALayer *layer = [[CALayer alloc]init];
    layer.frame = _headerView.bounds;
    layer.delegate = self;//这里在代理释放上有问题，如僵尸对象
    layer.backgroundColor = [UIColor clearColor].CGColor;
    [layer setNeedsDisplay];
    [_headerView.layer addSublayer:layer];

    #pragma mark --CALayerDelegate

    -(void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx{
        CGContextSaveGState(ctx);
        CGContextAddArc(ctx, SCREEN_WIDTH/2, 55, 40.5, 0, 2 * M_PI, 0);
        CGContextSetLineWidth(ctx, 1);
        CGContextSetStrokeColorWithColor(ctx, [[UIColor whiteColor]colorWithAlphaComponent:1.0].CGColor);
        CGContextStrokePath(ctx);
        CGContextAddArc(ctx, SCREEN_WIDTH/2, 55, 45.5, 0, 2 * M_PI, 0);
        CGContextSetLineWidth(ctx, 0.5);
        CGContextSetStrokeColorWithColor(ctx, [[UIColor whiteColor]colorWithAlphaComponent:0.35].CGColor);
        CGContextStrokePath(ctx);
        CGContextAddArc(ctx, SCREEN_WIDTH/2, 55, 50.5, 0, 2 * M_PI, 0);
        CGContextSetLineWidth(ctx, 0.5);
        CGContextSetStrokeColorWithColor(ctx, [[UIColor whiteColor]colorWithAlphaComponent:0.1].CGColor);
        CGContextStrokePath(ctx);
        CGContextRestoreGState(ctx);
    }




#### CAGradientLayer实现渐变色

    CAGradientLayer *gradientLayer = [CAGradientLayer layer];
    gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor,(__bridge id)[UIColor greenColor].CGColor];
    //gradientLayer.locations = @[@0.33,@0.66];//分界点，三种颜色需要两个点吧
    gradientLayer.startPoint = CGPointMake(0, 0);
    gradientLayer.endPoint = CGPointMake(1, 0);
    gradientLayer.frame = CGRectMake(100, 100, 100, 10);
    gradientLayer.cornerRadius = 5.0;
    [self.view.layer addSublayer:gradientLayer];




#### CAShapeLayer

配合UIBezierPath实现复杂的遮罩、动画，下面是一个动画的实现：

    CAShapeLayer *shapeLayer = [CAShapeLayer layer];
    shapeLayer.frame = _demoView.bounds;

    UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:_demoView.bounds];

    shapeLayer.path = path.CGPath;
    shapeLayer.fillColor = [UIColor clearColor].CGColor;
    shapeLayer.lineWidth = 2.0f;
    shapeLayer.strokeColor = [UIColor redColor].CGColor;
    [_demoView.layer addSublayer:shapeLayer];

    CABasicAnimation *pathAnima = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
    pathAnima.duration = 3.0f;
    pathAnima.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    pathAnima.fromValue = [NSNumber numberWithFloat:0.0f];
    pathAnima.toValue = [NSNumber numberWithFloat:1.0f];
    pathAnima.fillMode = kCAFillModeForwards;
    pathAnima.removedOnCompletion = NO;
    [shapeLayer addAnimation:pathAnima forKey:@"strokeEndAnimation"];


## TouchID Authenticate

导入  `LocalAuthentication.framework`
`#import <LocalAuthentication/LocalAuthentication.h>`


    //TouchID验证对象
    LAContext *context = [[LAContext alloc]init];
    context.localizedFallbackTitle = @"验证登录密码";//撤销按钮title
    
    NSError *error = nil;
    if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error]) {
        //验证回调
        [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:@"通过Home键验证已有手机指纹" reply:^(BOOL success, NSError * _Nullable error) {
            
            if (success) {
                
                NSLog(@"authentication succeeded");
                
             } else if (error) {
                
                NSLog(@"%ld",error.code);
                
                switch (error.code) {
                        
                    case LAErrorAuthenticationFailed: {
                    }
                        break;
                        
                    case LAErrorUserCancel: {
                    }
                        break;
                        
                    case LAErrorUserFallback: {
                    }
                        break;
                        
                    case LAErrorSystemCancel:{
                    }
                        break;
                        
                    case LAErrorTouchIDNotEnrolled: {
                    }
                        break;
                        
                    case LAErrorPasscodeNotSet: {
                    }
                        break;
                        
                    case LAErrorTouchIDNotAvailable: {
                    }
                        break;
                        
                    case LAErrorTouchIDLockout: {
                    }
                        break;
                        
                    case LAErrorAppCancel:  {
                    }
                        break;
                    case LAErrorInvalidContext: {
                    }
                        break;
                }
             }else{
                 
             }
        }];
        
    } else {
        NSLog(@"This device isn't supported");
    }



## CGContext
   
[CGContext 解释](http://www.cnblogs.com/rambo-q/p/5624329.html)


## AVFoundation

### MPMoviePlayerController
  
MPMoviePlayerController必须是全局的属性级别的

    NSString *string = @"http://videos.cctvnews.cn/publish/app/data/2015/03/30/408156/b5737e32-790c-432c-83aa-df61132fd97c.m3u8";
    NSURL *url = [NSURL URLWithString:string];
    self.player_1 = [[MPMoviePlayerController alloc]initWithContentURL:url];
    self.player_1.view.frame = CGRectMake(0, 20, 375, 300);
    self.player_1.controlStyle = MPMovieControlStyleEmbedded;
    [self.view addSubview:self.player_1.view];
    [self.player_1 play];



### AVAudioSession

    [[AVAudioSession sharedInstance] setActive:NO withFlags:AVAudioSessionSetActiveFlags_NotifyOthersOnDeactivation error:nil];


