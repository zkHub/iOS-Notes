# Foundation

[toc]

---

## NSString

### 初始化

    NSString *string = [NSString stringWithFormat:@"%d%%",rate];//百分号转义

### 字符串操作

#### string分割

    NSString *str = @",1,2,3";
    NSArray *arr = [str componentsSeparatedByString:@","];//arr.count=4

#### charactersInSet分割

    NSCharacterSet *cs = [NSCharacterSet characterSetWithCharactersInString:@"0123456789X"];
    NSArray *arr = [text componentsSeparatedByCharactersInSet:cs];
    
这个方法常用于限制输入特定字符（**invertedSet**方法是取反集合）

    //invertedSet方法是取反集合
    NSCharacterSet *cs = [[NSCharacterSet characterSetWithCharactersInString:@"0123456789X"] invertedSet];
    NSString*filtered = [[text componentsSeparatedByCharactersInSet:cs] componentsJoinedByString:@""];


替换某些字符

    NSCharacterSet *cs = [NSCharacterSet characterSetWithCharactersInString:@" ,，"];
    NSString*filtered = [[searchBar.text componentsSeparatedByCharactersInSet:cs] componentsJoinedByString:@"|"];

#### 替换某个字符

    NSString *dModel = [device.model stringByReplacingOccurrencesOfString:@" " withString:@""];//去掉string中的空格

#### 去除指定字符集

    //剪切掉收尾空格 set可以自定义
    NSString *temptext = [messageTextField.text stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]];

#### path操作

`stringByAppendingPathComponent` 是添加/号，使之变成一个完整的路径   a/b

`stringByAppendingPathExtension` 是加后缀的意思  a.b

#### NSString -> unichar

    - (unichar)characterAtIndex:(NSUInteger)index;

#### 编码

**UTF8编码**

    urlString = [urlString stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

**gbk编码**

    NSStringEncoding gbk = CFStringConvertEncodingToNSStringEncoding(kCFStringEncodingGB_18030_2000);
    string = [string stringByAddingPercentEscapesUsingEncoding:gbk];


iOS9之后废弃
编码使用`stringByAddingPercentEncodingWithAllowedCharacters: `
解码使用`stringByRemovingPercentEncoding`

    URLFragmentAllowedCharacterSet  "#%<>[\]^`{|}
    URLHostAllowedCharacterSet      "#%/<>?@\^`{|}
    URLPasswordAllowedCharacterSet  "#%/:<>?@[\]^`{|}
    URLPathAllowedCharacterSet      "#%;<>?[\]^`{|}
    URLQueryAllowedCharacterSet     "#%<>[\]^`{|}
    URLUserAllowedCharacterSet      "#%/:<>?@[\]^`




### NSString ->  NSDictionary

    NSString *oddsString = [dataDict objectForKey:@"odds_info"];
    NSData *oddsData = [oddsString dataUsingEncoding:NSUTF8StringEncoding];
    tipInfo.odds_info = [NSJSONSerialization JSONObjectWithData:oddsData options:NSJSONReadingMutableContainers error:nil];



## NSAttributedString

### 使用


#### 基本使用

一般使用其subClass `NSMutableAttributedString` 配合`Attributes`键值达到富文本实现

`Attributes` 中 `NSParagraphStyleAttributeName` 为段落属性

    NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:self.row.intentString];
    NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
    paragraphStyle.lineSpacing = 6.0;
    paragraphStyle.headIndent = 70;//缩进
    paragraphStyle.firstLineHeadIndent = 0;//首行缩进
    [attributedString addAttributes:@{NSParagraphStyleAttributeName:paragraphStyle,NSFontAttributeName:[UIFont systemFontOfSize:14]} range:NSMakeRange(0, [self.row.intentString length])];

#### 图文混排

 `NSTextAttachment` 对应的 `key` 为 `NSAttachmentAttributeName`

    NSTextAttachment * attachment=[[NSTextAttachment alloc] init];
    attachment.image=[UIImage imageNamed:@"xfb_favourable"];
    attachment.bounds=CGRectMake(0, -4, 28*WidthScale, 14*HeightScale);//这里bounds的x值并不会产生影响
    NSAttributedString *attachmentString = [NSAttributedString attributedStringWithAttachment:attachment];
    [self insertAttributedString:attachmentString atIndex:index];

`NSTextAttachment` 可以包含 `NSData` 或者 `NSFileWrapper` 对象，还未使用过😂


#### 21个属性介绍 [iOS之富文本](http://www.2cto.com/kf/201409/334308.html)

#### 大神封装的一个链式编程的分类，觉得挺好使，以后整理补充。



#### 还有CoreText，以后整理补充。




## NSDate

### NSDateFormatter

#### 常用到的dateFormat

     /**
     G 年代标志符（公元）
     y 年
     M 月
     d 日
     h 时 在上午或下午(1~12) 十二小时制
     H 时 在一天中(0~23) 二十四小时制
     m 分
     s 秒
     S 毫秒
     E~EEE 周几 EEEE 星期几
     a 上午 / 下午 标记符
     z 时区
     D 一年中的第几天
     A 今天的多少毫秒
     设置dateFormat属性 
     self.dateFormatter.dateFormat = @"G yyyy-MM-dd a HH:mm:ss.SSS EEEE z";
     */

#### 自定义 NSDateFormatter

实现 `NSString` 与 `NSDate` 相互转换，如下：

    NSDate* now = [NSDate date];
    NSDateFormatter* fmt = [[NSDateFormatter alloc] init];
    fmt.locale = [[NSLocale alloc] initWithLocaleIdentifier:@"zh_CN"];//美国en_US，缺省应该是按系统语言设置
    //默认是系统时区，没怎么设置过😁
    fmt.timeZone = [NSTimeZone localTimeZone]; //[NSTimeZone timeZoneForSecondsFromGMT:0];
    fmt.dateFormat = @"yyyy-MM-dd HH:mm:ss";
    //NSDateFormatter实现NSString与NSDate相互转换
    NSString* dateString = [fmt stringFromDate:now];
    NSDate *date = [fmt dateFromString:dateString];

这里有一篇写的特别多的 [NSDateFormatter整理](http://www.cocoachina.com/bbs/read.php?tid=134821)


### NSDate的使用

    //从现在倒数24个小时
    NSDate *date = [NSDate dateWithTimeIntervalSinceNow:-24*60*60];

`- (NSTimeInterval)timeIntervalSinceReferenceDate;` 以2001/01/01 GMT为基准时间，返回实例保存的时间与2001/01/01 GMT的时间间隔


## NSArray

`NSArray` 不可变数组，只能存放对象，且可以存放任意对象。
`[NSNull null]` 是空对象，不允许将 `nil` 插入到数组中,因为 `nil` 作用是数组的结束标记。

### 数组在Swift与OC中的区别：
Swift数组要求存储、显示或类型推断的是**相同类型**，但不是必须为class类型。
OC中NSAarry可以存储任何类型的对象（实例）而且不提供他们返回对象的任何本质信息。


### 常用的方法

    //下标获取单个元素
    id obj =[arr2 objectAtIndex:0]; 
    //下标集合获取数组    
    NSArray *sub2 = [arr2 objectsAtIndexes:indexs];    
    //修改数组中的内容  
    [arr2 replaceObjectAtIndex:2 withObject:@"aaa"];
    //交换数组中的内容  
    [arr2 exchangeObjectAtIndex:1 withObjectAtIndex:3]; 
    //包含某个元素
    - (BOOL)containsObject:(ObjectType)anObject;


### 排序

**按字母排序**

    NSArray *keys = [dict allKeys];
    NSArray *sortedArray = [keys sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
       return [obj1 compare:obj2 options: NSNumericSearch];//这是字符串的扩展
    }];


**自定义排序**
    
    keyArr = [NSMutableArray arrayWithArray:[keyArr sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
        if ([obj1 intValue] < [obj2 intValue]) {
            return NSOrderedAscending;//升序
        }else {
            return NSOrderedDescending;//降序
        }
    }]];

### 遍历

**这两种遍历都是按顺序来的**
    
    [sortedKeys enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
       NSLog(@"%ld",(long)idx);
    }];
    
    for (NSString *string in sortedKeys) {
        NSLog(@"%@",string);
    }


#### 数组倒序遍历

`reverseObjectEnumerator` 获得一个倒序的枚举器

    + (MB_INSTANCETYPE)HUDForView:(UIView *)view {
    	NSEnumerator *subviewsEnum = [view.subviews reverseObjectEnumerator];
    	for (UIView *subview in subviewsEnum) {
    		if ([subview isKindOfClass:self]) {
    			return (MBProgressHUD *)subview;
    		}
    	}
    	return nil;
    }


## NSPredicate

[iOS中的谓词（NSPredicate）使用](http://www.cocoachina.com/ios/20160111/14926.html)

### 常用匹配

    //验证手机号
     - (BOOL)checkPhoneNumber:(NSString *)phoneNumber
    {
        NSString *regex = @"^[1][3-9]\\d{9}$";
        NSPredicate *pred = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", regex];
        return [pred evaluateWithObject:phoneNumber];
    }

    //验证身份证号
    NSString *regex2 = @"^(\\d{14}|\\d{17})(\\d|[xX])$";

    //车牌号判断
    NSString *carRegex = @"^[\u4e00-\u9fa5]{1}[a-zA-Z]{1}[a-zA-Z_0-9]{4}[a-zA-Z_0-9_\u4e00-\u9fa5]$";

    //判断邮箱是否正确
    NSString *emailRegex = @"[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,4}";

     //军官证
     phoneRegex=@"([南|北|沈|兰|成|济|广|参|政|后|装|海|空])字第(\\d{8})号";
    
     //护照号码格式：
     //        因私普通护照号码格式有:14/15+7位数,G+8位数；因公普通:P.+7位数；
     //        公务：S.+7位数 或者 S+8位数,D开头外交护照.D=diplomatic
     phoneRegex=@"^(P\\d{7}|G\\d{8}|S\\d{7,8}|D\\d+|1[4,5]\\d{7})$";
    
     //回乡证前面一个字母，后面10个数字
     phoneRegex = @"^[A-Z]\\d{10}";
    


## NSNotification

### NSNotificationCenter

#### 三件套

    - (void)addObserver:(id)observer selector:(SEL)aSelector name:(nullable NSNotificationName)aName object:(nullable id)anObject;
    
    - (void)postNotificationName:(NSNotificationName)aName object:(nullable id)anObject userInfo:(nullable NSDictionary *)aUserInfo;
    
    - (void)removeObserver:(id)observer;
    - (void)removeObserver:(id)observer name:(nullable NSNotificationName)aName object:(nullable id)anObject;

**时机**

    //键盘通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShow:) name:UIKeyboardWillChangeFrameNotification object:nil];

    - (void)dealloc {
        [[NSNotificationCenter defaultCenter] removeObserver:self];
    }



## NSUserdefaults

  	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];

`-setObject: forKey:` NSUserDefaults支持存储的对象 **NSData，NSString，NSNumber,NSDate,NSArray,NSDictionary,BOOL** 且必须是不可变的。
set完之后会异步持续执行数据同步，一般设置完就调用 `[defaults synchronize];` 手动同步。
  


## NSFileManager

    NSFileManager * fileManager = [NSFileManager defaultManager];

文件是否存在

    - (BOOL)fileExistsAtPath:(NSString *)path; 
    - (BOOL)fileExistsAtPath:(NSString *)path isDirectory:(nullable BOOL *)isDirectory;


文件大小 

    [[fileManager attributesOfItemAtPath:filePath error:nil] fileSize]
    
复制，移动，删除等
    
    [fileManager removeItemAtPath:path error:nil];
    //moveItemAtPath,copyItemAtPath,linkItemAtPath

### 常用path

`/Documents`

    NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject

`/Library`

    NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES).firstObject
    
`/Library/Caches`

    NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject



### NSUUID

    [[NSUUID UUID] UUIDString]

