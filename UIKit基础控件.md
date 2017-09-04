# UIKit基础控件

[toc]

---
## UIView

### viewWithTag

    UIButton *btn = (UIButton *)[self.view viewWithTag:i + 200];

viewWithTag函数会递归的匹配subviews。

### clipsToBounds

    @property(nonatomic) BOOL clipsToBounds; // When YES, content and subviews are clipped to the bounds of the view. Default is NO.

### layoutSubviews & drawRect

layoutSubviews方便数据计算，drawRect方便视图重绘。两个方法都是异步执行

**layoutSubviews在以下情况下会被调用：**

> 1. addSubview会触发layoutSubviews。所以不要在这个方法中addSubview
> 2. 设置view的Frame会触发layoutSubviews，当然前提是frame的值设置前后发生了变化。
> 3. 滚动一个UIScrollView会触发layoutSubviews。
> 4. 旋转Screen会触发父UIView上的layoutSubviews事件。
> 5. 改变一个UIView大小的时候也会触发父UIView上的layoutSubviews事件。
> 6. 直接调用setLayoutSubviews。

**drawRect在以下情况下会被调用：**

> 1. 如果在UIView初始化时没有设置rect大小，将直接导致drawRect不被自动调用。drawRect 掉用是在Controller->loadView, Controller->viewDidLoad 两方法之后掉用的.
> 2. 该方法在调用sizeToFit后被调用，所以可以先调用sizeToFit计算出size。然后系统自动调用drawRect:方法。
> 3. 通过设置contentMode属性值为UIViewContentModeRedraw。那么将在每次设置或更改frame的时候自动调用drawRect:。
> 4. 直接调用setNeedsDisplay，或者setNeedsDisplayInRect:触发drawRect:，但是有个前提条件是rect不能为0。
> 以上1,2推荐；而3,4不提倡

**drawRect方法使用注意点：**
> 1.  若使用UIView绘图，只能在drawRect：方法中获取相应的contextRef并绘图。如果在其他方法中获取将获取到一个invalidate 的ref并且不能用于画图。drawRect：方法不能手动显示调用，必须通过调用setNeedsDisplay 或 者 setNeedsDisplayInRect，让系统自动调该方法。
> 2. 若使用calayer绘图，只能在drawInContext: 中（类似于drawRect）绘制，或者在delegate中的相应方法绘制。同样也是调用setNeedDisplay等间接调用以上方法
> 3. 若要实时画图，不能使用gestureRecognizer，只能使用touchbegan等方法来调用setNeedsDisplay实时刷新屏幕


## UIButton

### 1. [UIButton中setTitleEdgeInsets和setImageEdgeInsets的使用](http://blog.csdn.net/dfqin/article/details/37813591)

UIEdgeInsetsMake(CGFloat top, CGFloat left, CGFloat bottom, CGFloat right)
setTitleEdgeInsets，setImageEdgeInsets
UIEdgeInsetsMake(top+a, 0, 0, 0)
若a为负数则两者y值减（-a/2） 否则titleLable的宽高不变 y值增长(a/2)
imageView在(y+h)<button.h时y值增长(a/2)  之后y+a，h-a直到h=0，y=button.h y不变h+a

### 2. 代码执行点击UIbutton事件

    [button sendActionsForControlEvents:UIControlEventTouchUpInside];
    
### 3. UIButton title左对齐
    
    button.contentHorizontalAlignment = UIControlContentHorizontalAlignmentLeft;

### 4. button的自定义设置
    
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    button.frame = CGRectMake(0, 0, 70, 30);
    [button setTitle:@"返回" forState:UIControlStateNormal];
    [button setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];
    [button setTitleEdgeInsets:UIEdgeInsetsMake(0, -10, 0, 0)];//titleLabe的宽高一般不会变。
    button.titleLabel.font = [UIFont systemFontOfSize:17];//设置字体大小
    [button setImage:[UIImage imageNamed:@"return"] forState:UIControlStateNormal];
    [button setImageEdgeInsets:UIEdgeInsetsMake(4.5, -20, 4.5, 0)];//imageView的frame根据button边界会变。
    [button addTarget:self action:@selector(backButton:) forControlEvents:UIControlEventTouchUpInside];
    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]initWithCustomView:button];
   
### 5. button的状态

normal		普通状态

highlight	按下时高亮状态

**selected	选中状态 与高亮状态冲突**（需要设置selected，否则只有上面的两种状态）
   
**取消button高亮状态**

    [priceBtn setAdjustsImageWhenHighlighted:NO];
    [priceBtn setImage:[UIImage imageNamed:@"sequence_up_pressed.png"] forState:UIControlStateHighlighted];


## UILabel

### 1. 计算UILabel高度

在官方文档里可以看到，`- (CGSize)sizeWithFont:(UIFont *)font constrainedToSize:(CGSize)size lineBreakMode:(NSLineBreakMode)lineBreakMode;`这个函数在iOS7里面已经**deprecated**了，取而代之的是`boundingRectWithSize:options:attributes:context:`这个函数。实际上这两个函数在iOS7里面的计算结果是一致的，都是带小数的。boundingRectWithSize:options:attributes:context:的文档中可以看到这么一句话：
> This method returns fractional sizes (in the size component of the returned CGRect); to use a returned size to size views, you must use raise its value to the nearest higher integer using the ceil function.

也就是说，计算出的size需要用ceil函数取整，如：`labelSize.height = ceil(labelSize.height);`

> extern float ceilf(float);
extern double ceil(double);
extern long double ceill(long double);

    CGRect rect = [keyStr boundingRectWithSize:CGSizeMake(500, 50) options:NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:20]} context:nil];
    height = ceil(rect.size.height);



## UIFont

### 系统字体

```
+ (UIFont *)systemFontOfSize:(CGFloat)fontSize;
+ (UIFont *)boldSystemFontOfSize:(CGFloat)fontSize;//粗体
+ (UIFont *)italicSystemFontOfSize:(CGFloat)fontSize;//斜体
```

### 使用自定义字体

    //包括系统内置的其它字体也是这个方法
    UIFont *font = [UIFont fontWithName:@"FZLTXHK--GBK1-0" size:40];    


[IOS导入ttf等文件使用自定义字体](http://www.jianshu.com/p/e29c37639d81)







## UIImage

### 保存image到相册

    UIImageWriteToSavedPhotosAlbum(image, self, @selector(imageSavedToPhotosAlbum:didFinishSavingWithError:contextInfo:), nil);

**回调**

    - (void)imageSavedToPhotosAlbum:(UIImage *)image didFinishSavingWithError:(NSError *)error contextInfo:(void *)contextInfo
    {
        NSString *message =nil;
        if (!error) {
            message = @"成功保存到相册";
        }else
        {
            message = [error description];
        }
        NSLog(@"message is %@",message);
    }

### 九切片

    UIImage* image = [UIImage imageNamed:@"delete_btn.png"];
    //    resizableImage 就是九切片过后的 新图片
    UIImage* resizableImage = [image 
    resizableImageWithCapInsets:UIEdgeInsetsMake(10, 10, 10, 10) // 上，下，左，右 留边尺寸
      resizingMode:UIImageResizingModeStretch];
    // UIImageResizingModeStretch:  变化时拉伸
    // UIImageResizingModeTile :   变化时复制

### -

 [UIImage imageNamed:@""]为空时会log  `CUICatalog: Invalid asset name supplied:`






## UITextField & UITextView

### 1. UITextField的常见设置

    passwordField=[[UITextField alloc]initWithFrame:CGRectMake(25 ,141, fieldBg.size.width, fieldBg.size.height)];
    passwordField.tag = 10;
    passwordField.textColor= [UIColor blackColor];
    passwordField.font= [UIFont systemFontOfSize:16] ;
    passwordField.backgroundColor= [UIColor clearColor] ;
    passwordField.contentVerticalAlignment=UIControlContentVerticalAlignmentCenter;
    passwordField.returnKeyType = UIReturnKeyDone;
    passwordField.delegate=self;
    passwordField.borderStyle = UITextBorderStyleNone;//边框
    [passwordField setBackground:fieldBg];
    passwordField.secureTextEntry = YES;//密文输入
    passwordField.placeholder = [NSString stringWithFormat:@"密码"];
    passwordField.attributedPlaceholder = [[NSAttributedString alloc] initWithString:@"密码" attributes:@{NSForegroundColorAttributeName:[UtilityHelper colorWithHexString:@"ffffff"]}];
    [passwordField setValue:[UIColor whiteColor] forKeyPath:@"_placeholderLabel.textColor"];//待考证
    passwordField.clearButtonMode = UITextFieldViewModeWhileEditing;//输入时可删除
    passwordField.autocorrectionType = UITextAutocorrectionTypeNo;
    passwordField.autocapitalizationType = UITextAutocapitalizationTypeNone;
    [inputView addSubview:passwordField];

在某些控件包含textField私有属性时可以使用`[passwordField setValue:[UIColor whiteColor] forKeyPath:@"_placeholderLabel.textColor"];`设置textFiled的相关属性，是利用runtime获取到，并通过KVC来修改控件的属性见[KVC实现修改控件属性](#KVC_change_property)

### 2. 限制textFiled输入的字符数

    //text限制输入字符数
    #pragma UITextViewDelegate
    //此方法会导致多输入时textField中string不变 而使string多出一位。因此获取string尽量在endEditing中。
    - (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text {
        NSString *resultStr = [textView.text stringByReplacingCharactersInRange:range withString:text];
        if (resultStr.length > 100) {
            [textView resignFirstResponder];
            [UIToastView showToastViewWithContent:@"最多输入100个字" andRect:CGRectZero andTime:2.0 andObject:[self viewController]];
            return NO;
        }
        return YES;
    }
    
    //解决系统输入法备选输入shouldChangeTextInRange不被调用的情况（可以不用上面的方法了吧）
    - (void)textViewDidChange:(UITextView *)textView {
        if (textView.text.length >= 100) {
            textView.text = [textView.text substringToIndex:100];
            [textView resignFirstResponder];
            [UIToastView showToastViewWithContent:@"最多输入100个字" andRect:CGRectZero andTime:2.0 andObject:[self viewController]];
        }
    }




## UISearchBar

### 1. [UISearchController替换UISearchDisplayController](http://blog.csdn.net/u011619283/article/details/52711311)

### 2.设置搜索光标 跟 取消按钮 颜色不一样的方法
***这个应该可以设置大部分使用UIBarButtonItem的控件*** 如：UITabBar，UINavigationBar...

    self.searchbar.tintColor = XFBTHEMECOLOR;//主题色
    [[UIBarButtonItem appearanceWhenContainedIn:[UISearchBar class], nil] setTitleTextAttributes:@{NSFontAttributeName:[UIFont systemFontOfSize:13],NSForegroundColorAttributeName:[XFBUtilityHelper colorWithHexString:@"#ffffff"]} forState:UIControlStateNormal];//按钮单个设置

## UITabBar & UINavigationBar
### 1.iOS隐藏导航条1px的底部横线
    [self.navigationController.navigationBar setBackgroundImage:[[UIImage alloc]init] forBarMetrics:UIBarMetricsDefault];
    self.navigationController.navigationBar.shadowImage = [[UIImage alloc]init];
    //据说：这是唯一一个隐藏这条线的官方用法，但是有一个缺陷-删除了translucency(半透明)

### 2. 隐藏tabbar上面的1px线
    tabBar.backgroundImage = [[UIImage alloc]init];
    tabBar.shadowImage = [[UIImage alloc]init];


### 3. 自定义tabbar | navigationBar

**UITabBar**
    
    [[UITabBarItem appearance]setTitleTextAttributes:[NSDictionary dictionaryWithObjectsAndKeys:[UIFont systemFontOfSize:15], NSFontAttributeName,nil] forState:UIControlStateHighlighted];

**UINavigationBar**

    //ios7之后用barTintColor设置navigationBar背景色
    [self.navigationController.navigationBar setBarTintColor:[UIColor blueColor]];
    self.navigationController.navigationBar.titleTextAttributes = @{NSForegroundColorAttributeName:[UIColor redColor]};
    //全局有效的
    [UINavigationBar appearance].barTintColor = [UIColor blueColor];
    [UINavigationBar appearance].titleTextAttributes = @{NSForegroundColorAttributeName:[UIColor redColor]};



### 4. 隐藏底部的tabbar

`[self.tabBarController.navigationController pushViewController:container animated:YES];` 可以直接自动处理tabbar隐藏，也可以自定义UINavigationController重写push方法

    @interface NavigationController : UINavigationController
    
    @end
    
    
    @implementation NavigationController
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.
    }
    
    /**
     *  重写Push方法(隐藏底部的tabbar)
     */
    - (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
    {
        if (self.viewControllers.count) { //避免一开始就隐藏了
            viewController.hidesBottomBarWhenPushed = YES;
        }
        
        [super pushViewController:viewController animated:animated];
    }
    
    @end

**tabbar跳转的时候使用hiden隐藏有的时候会导致webview计算其frame**可能是在didappear的时候

**下面这是啥我不知道先记录一下，有可能是我瞎说的，因为忘了这个bug了**

    //系统Tabbar可能自动渲染图片导致非原图
    UITabBarItem *ffdxItem = [[UITabBarItem alloc]initWithTitle:@"付费短信" image:[[UIImage imageNamed:@"btn-fufeiduanxin"]imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] selectedImage:[[UIImage imageNamed:@"btn-fufeiduanxin--select"]imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal]];

### 5. UINavigationController

#### 1. 右滑返回手势
backBarButtonItem 是否启用右滑返回手势，默认YES

    self.navigationController.interactivePopGestureRecognizer.enabled = YES;
    
解决自定义返回按钮不支持右滑返回手势

    self.navigationItem.backBarButtonItem = barItem; 

#### 2.解决系统相册导航透明

遵守协议`<UINavigationControllerDelegate>`
设置代理`imagePickerController.delegate = self;`
实现代理方法（prefect）

    -(void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
    {
        if ([navigationController isKindOfClass:[UIImagePickerController class]]) {
            viewController.navigationController.navigationBar.translucent = NO;
        }
        viewController.navigationItem.rightBarButtonItem.width+=10;//这个属性666
    }

### 6. UITabBarController

#### 1. 获取当前导航控制器

有时候会给tabBar中的每个控制器都加一个导航控制器可以用`selectedViewController`属性获取当前的。


## UIAlertController & UIAlertView & UIActionSheet

### 1. UIAlertController的使用（iOS8）

**以UIDatePicker加在alert | actionSheet上为例**

    UIDatePicker *datePicker = [[UIDatePicker alloc]initWithFrame:CGRectMake(0, 0, SCREEN_WIDTH - 105, 200)];//alert
    datePicker.datePickerMode = UIDatePickerModeTime;
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"\n\n\n\n\n\n\n" message:nil preferredStyle:UIAlertControllerStyleAlert];
    [alert.view addSubview:datePicker];
    /*
    UIDatePicker *datePicker = [[UIDatePicker alloc]initWithFrame:CGRectMake(0, 0, SCREEN_WIDTH - 20, 200)];//比alert宽一点
    datePicker.datePickerMode = UIDatePickerModeTime;
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"\n\n\n\n\n\n\n\n\n" message:nil preferredStyle:UIAlertControllerStyleActionSheet];//比alert多两个\n
    [alert.view addSubview:datePicker];
    */
    //创建action，block中可以写按钮相关回调
    UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        NSDateFormatter *formatter = [[NSDateFormatter alloc]init];
        [formatter setDateFormat:@"HH:mm"];
        NSString *dateStr = [formatter stringFromDate:datePicker.date];
        SL_Log(@"%@",dateStr);
    }];
    [okAction setValue:[XFBUtilityHelper colorWithHexString:@"#228ce2"] forKey:@"titleTextColor"];//KVC修改按钮颜色
    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        
    }];
    [alert addAction:okAction];
    [alert addAction:cancelAction];
    [self presentViewController:alert animated:YES completion:nil];

okAction按钮的颜色利用了[KVC实现修改控件属性](#KVC_change_property)

### 2. UIAlertView & UIActionSheet

**alertView 添加 activityIndicatorView**

    UIAlertView *alert = [[UIAlertView alloc]initWithTitle:nil message:string delegate:delegate cancelButtonTitle:nil otherButtonTitles:nil, nil];
    UIActivityIndicatorView *activity = [[UIActivityIndicatorView alloc]init];
    activity.activityIndicatorViewStyle = UIActivityIndicatorViewStyleWhiteLarge;
    [alert setValue:activity forKey:@"accessoryView"];
    [activity startAnimating];
    [alert dismissWithClickedButtonIndex:0 animated:YES];//并不会触发代理

**有个需求：弹出好多alert，但是还没有点完账号被踢的处理**
原理就是保持alert一个全局的引用，最后可以获取到，并释放。

    @interface UIAlertView (iOSNewShow)
    
    -(void)showNew;
    
    @end
    
    @implementation UIAlertView (iOSNewShow)
    
    -(void)showNew{
        [self show];
        if (!((AppDelegate*)[UIApplication sharedApplication].delegate).alertArray) {
            ((AppDelegate*)[UIApplication sharedApplication].delegate).alertArray = [[NSMutableArray alloc]init];
        }
        [((AppDelegate*)[UIApplication sharedApplication].delegate).alertArray addObject:self];
    }
    
    @end
    
    -(void)removeAlertArray{
        for (UIAlertView* alert in self.alertArray) {
            if (alert) {
                [alert dismissWithClickedButtonIndex:0 animated:YES];
            }
        }
        [self.alertArray removeAllObjects];
    }

iOS7之前是这样的（虽然不会用到了，但是这个idea很好）
学习自：[实现全局关闭所有键盘,actionSheet和alertView](https://www.baidu.com/link?url=iXITY9isMTcrA4vsqOj0k3sfLYSGLpUxzvXivNwDyQ2C296g8X_EMWM48vUhozKjkiZATzEY4kLq-FeMKI1jF_&wd=&eqid=89375914000372fe00000003577a1ebf)

    -(void)removeAlertView{
        
        for (UIView *view in [UIApplication sharedApplication].keyWindow.subviews) {
            for (UIView *subView in view.subviews) {
                if ([subView isKindOfClass:[UIAlertView class]]) {
                   UIAlertView *view1 = (UIAlertView*)subView;
                    [view1 dismissWithClickedButtonIndex:view1.cancelButtonIndex animated:YES];
                }else{
                    [self dismissAlertForView:subView];
                }
            }
        }
        
    }
    -(void)dismissAlertForView:(UIView*)view{
        for (UIView *subView in view.subviews) {
            if ([subView isKindOfClass:[UIAlertView class]]) {
                UIAlertView *view1 = (UIAlertView*)subView;
                [view1 dismissWithClickedButtonIndex:view1.cancelButtonIndex animated:YES];
            }else{
                [self dismissAlertForView:subView];
            }
        }
    }


## UITableView

### 1. cell的创建

    //tableview初始化的时候注册
    [self.tableView registerClass:[CustomCell class] forCellReuseIdentifier:@"cell"];
    
    //cellForRow方法中
    CustomCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell" forIndexPath:indexPath];
    
`dequeueReusableCellWithIdentifier` 和`dequeueReusableCellWithIdentifier: forIndexPath:`的区别就是上下两种写法
上面的需要注册绑定，不用判断cell是否为空

**比较早的写法**

    static NSString *identifier = @"cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];  
    if (!cell) {
       cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:identifier];
    }else{
        //这个是防止在cellForRow中addSubview导致的重复（一般不会这样写了）
       for (UIView *subView in cell.contentView.subviews)
       {
           [subView removeFromSuperview];
       }
    }

### 2. 不显示没内容的多余cell

    //设置tableFooterView
    self.tableView.tableFooterView = [[UIView alloc] init];

### 3. 点击状态栏 tableview/scrollView 回到顶端

同时存在两个或以上 该功能不起作用  应把其他的`scrollView.scrollsToTop = NO;`
（ipad中 可能是对应区域的最上层view起作用）

### 4. tableView scrollTo…

    [_tableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:0 inSection:0] atScrollPosition:UITableViewScrollPositionTop animated:YES];

`[NSIndexPath indexPathForRow:0 inSection:0]` NSIndexPath的(UITableView)类别 

### 5. 获取滚动tableview中响应的textview

    /*
      //首先通过这两行代码获取第一响应者    
      UIWindow * keyWindow = [[UIApplication sharedApplication] keyWindow];
      UIView * firstResponder = [keyWindow performSelector:@selector(firstResponder)];
    */
    
      //上面的方法在弹出alertview的时候，响应者变成alertview，alert消失会导致崩溃，所以尽量用self.view
      UIView * firstResponder = [self.view performSelector:@selector(firstResponder)];
      if ([firstResponder.superview.superview isKindOfClass:[UITableViewCell class]]) {
          UITableViewCell *cell = (UITableViewCell*)firstResponder.superview.superview;
          NSIndexPath *indexPath = [self.CertifyTableView indexPathForCell:cell];
          [self.CertifyTableView scrollToRowAtIndexPath:indexPath atScrollPosition:UITableViewScrollPositionMiddle animated:YES];
      }


### 6. section header在滚动时会默认悬停在界面顶端 

当 UITableView 的 style 属性设置为 Plain 时，这个tableview的section header在滚动时会默认悬停在界面顶端。取消这一特性的方法有两种：

**1. 将 style 设置为 Grouped 。这时所有的section header都会随着scrollview滚动了。不过 grouped 和 plain 的样式有轻微区别，切换样式后也许需要重新调整UI**

**2. 重载scrollview的delegate方法**

    - (void)scrollViewDidScroll:(UIScrollView *)scrollView {
        CGFloat sectionHeaderHeight = 40;
        if (scrollView.contentOffset.y<=sectionHeaderHeight&&scrollView.contentOffset.y>=0) {
            scrollView.contentInset = UIEdgeInsetsMake(-scrollView.contentOffset.y, 0, 0, 0);
        } else if (scrollView.contentOffset.y>=sectionHeaderHeight) {
            scrollView.contentInset = UIEdgeInsetsMake(-sectionHeaderHeight, 0, 0, 0);
        }
    }


## UIScrollView

### UIScrollView或其子类上部有一10px左右的小黑条
当UIViewController.view的subView是scrollview或子类时其属性automaticallyAdjustsScrollViewInsets默认增加了填充，修改为NO

    self.automaticallyAdjustsScrollViewInsets =NO;

### 滚动时动态设置背景色

**判断两个颜色相同CGColorEqualToColor**

    - (void)scrollViewDidScroll:(UIScrollView *)scrollView {
        if (scrollView.contentOffset.y < 0) {//滚动时动态设置背景色
            if (!CGColorEqualToColor(self.CertifyTableView.backgroundColor.CGColor, [UtilityHelper colorWithHexString:@"#228CE2"].CGColor)) {
                self.CertifyTableView.backgroundColor = [UtilityHelper colorWithHexString:@"#228CE2"];
            }
        }else{
            if (!CGColorEqualToColor(self.CertifyTableView.backgroundColor.CGColor, [UtilityHelper colorWithHexString:@"#ffffff"].CGColor)) {
                self.CertifyTableView.backgroundColor = [UtilityHelper colorWithHexString:@"#ffffff"];
            }
        }
        
    }



## UIWebView

###     清除cookies

    NSHTTPCookie *cookie;
    NSHTTPCookieStorage *storage = [NSHTTPCookieStorage sharedHTTPCookieStorage];
    for (cookie in [storage cookies]) {
        [storage deleteCookie:cookie];
    }
      
###     清除缓存
    
    [[NSURLCache sharedURLCache] removeAllCachedResponses];
    [[NSURLCache sharedURLCache] setDiskCapacity:0];
    [[NSURLCache sharedURLCache] setMemoryCapacity:0];


### webView页面间的返回

    if (self.mywebView.canGoBack) {
        [self.mywebView goBack];
        return;
    }

### 存写cookie

一种是通过设置进request头文件（这个设置cookie会优先选择）

    NSHTTPCookie *cookie1 = [self makeCookieWithCookieName:@"sfut" withValue:self.sfut_cookie];
    NSDictionary *headers = [NSHTTPCookie requestHeaderFieldsWithCookies:@[cookie1]];//可以设置多个
    [urlRequest setValue:[headers objectForKey:@"Cookie"] forHTTPHeaderField:@"Cookie"];

    - (NSHTTPCookie *)setCookieWithCookieValue:(NSString *)cookieValue cookieName:(NSString *)cookieName  {
        NSMutableDictionary *properties = [[NSMutableDictionary alloc]init];
        [properties setValue:cookieValue forKey:NSHTTPCookieValue];
        [properties setValue:cookieName forKey:NSHTTPCookieName];
        [properties setValue:@"" forKey:NSHTTPCookieDomain];
        [properties setValue:[NSDate dateWithTimeIntervalSinceNow:60*60] forKey:NSHTTPCookieExpires];
        [properties setValue:@"/" forKey:NSHTTPCookiePath];
        NSHTTPCookie *cookie = [[NSHTTPCookie alloc] initWithProperties:properties];
        return cookie;
    }

一种是通过NSHTTPCookieStorage共享cookie（这个是会在全局生效的一个单例，优先级低于request的headerField）

    - (void)writeCookieWithCookieValue:(NSString *)cookieValue cookieName:(NSString*)cookieName {
        
        NSDictionary *cookieDic = [NSDictionary dictionaryWithObjectsAndKeys:
                                   @"", NSHTTPCookieDomain,
                                   @"/", NSHTTPCookiePath,
                                   cookieName,  NSHTTPCookieName,
                                   cookieValue, NSHTTPCookieValue,
                                   [NSDate dateWithTimeIntervalSinceNow:24*60*60], NSHTTPCookieExpires,
                                   nil];
        
        NSHTTPCookie *HTTPCookie = [NSHTTPCookie cookieWithProperties:cookieDic];
        [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:HTTPCookie];
        
    }



## UIGestureRecognizer 手势

### 手势与点击冲突

当`cancelsTouchesInView`为NO的时候，会分别触发`tapAction:`和`btnAction:`方法；而当`cancelsTouchesInView`为YES的时候，只会触发`tapAction:`方法。
因此可以设置为NO，并在手势代理中截取到点击view时取消手势事件。

    UITapGestureRecognizer *tapGr = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(viewTapGr:)];
    tapGr.cancelsTouchesInView = NO;
    tapGr.delegate = self;
    [self.view addGestureRecognizer:tapGr];
    
    -(void)viewTapGr:(UITapGestureRecognizer*)tapGr{
        [self.tableView endEditing:YES];
    }
    
    -(BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch{
        NSLog(@"输出点击的view的类名 = %@", NSStringFromClass([touch.view class]));
        
        // 若为UITableViewCellContentView（即点击了tableViewCell），则不截获Touch事件
        if ([NSStringFromClass([touch.view class]) isEqualToString:@"UITableViewCellContentView"]) {
            return NO;
        }
        
        if ([touch.view isKindOfClass:[UIButton class]]) {
            return NO;
        }
        return YES;
    }



## UIResponder

### 获取当前view所在的controller
- (UIViewController *)viewController {
    for (UIView* next = [self superview]; next; next = next.superview) {
        UIResponder *nextResponder = [next nextResponder];
        if ([nextResponder isKindOfClass:[UIViewController class]]) {
            return (UIViewController *)nextResponder;
        }
    }
    return nil;
}

### window层级
    
    for (UIView *view in [self.window subviews]) {
      if ( [NSStringFromClass([view class]) isEqualToString: @"UILayoutContainerView"]) {
          for (UIView *subView in view.subviews) {
              if ([NSStringFromClass([subView class]) isEqualToString:@"UIView"]) {
                  [subView removeFromSuperview];
              }
          }
      }
    }





## KVC实现修改控件属性 <a name="KVC_change_property"></a>

```
## <a name="KVC_change_property">KVC实现修改控件属性</a>
## <a name="KVC_change_property"></a>KVC实现修改控件属性
```



### 1. searchBar
    //获取searchBar里面的TextField
    UITextField*searchField = [_searchBar valueForKey:@"_searchField"];
    //更改searchBar 中PlaceHolder 字体颜色
    [searchField setValue:[UIColor blackColor] forKeyPath:@"_placeholderLabel.textColor"];
    //更改searchBar输入文字颜色
    searchField.textColor= [UIColor blackColor];

### 2. UIAlertController
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"标题" message:@"内容" preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
      //Add image 这里可以给button添加图片
    UIImage *accessoryImage = [UIImage imageNamed:@"selectRDImag.png"];
    accessoryImage = [accessoryImage imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
    [cancelAction setValue:accessoryImage forKey:@"image"];
    //设置cancelAction的title颜色
    [cancelAction setValue:[UIColor lightGrayColor] forKey:@"titleTextColor"];
    //设置cancelAction的title的对齐方式
    [cancelAction setValue:[NSNumber numberWithInteger:NSTextAlignmentLeft] forKey:@"titleTextAlignment"];
    [alertController addAction:cancelAction];
    UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:nil];
     //设置okAction的title颜色
    [okAction setValue:[UIColor greenColor] forKey:@"titleTextColor"];
    [alertController addAction:okAction];
    //Custom Title,使用富文本来改变title的字体大小
    NSMutableAttributedString *hogan = [[NSMutableAttributedString alloc] initWithString:@"Presenting the Hulk Hogan!"];
    [hogan addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:20] range:NSMakeRange(13, 11)];
    [alertController setValue:hogan forKey:@"attributedTitle"];
    NSMutableAttributedString *hogan1 = [[NSMutableAttributedString alloc] initWithString:@"hjasdghjdfsgkfdghfdgsgdsfgdsfg"];
    [hogan1 addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:20] range:NSMakeRange(13, 11)];
    [alertController setValue:hogan1 forKey:@"attributedMessage"];
    [self presentViewController:alertController animated:YES  completion:nil];


