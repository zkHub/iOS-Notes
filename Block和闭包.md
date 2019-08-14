# OC Block、Swift 闭包

[TOC]

## Block

### 1.block的数据结构
苹果开源了 `block` 和 `runtime` 的数据结构， [在线代码查看](http://opensource.apple.com/source/libclosure/libclosure-63/) 和下载苹果 [libclosure](http://www.opensource.apple.com/tarballs/libclosure/) 库的源码。

```
struct Block_descriptor_1 {
    uintptr_t reserved;
    uintptr_t size;
};
 
struct Block_layout {
    void *isa;
    volatile int32_t flags; // contains ref count
    int32_t reserved; 
    void (*invoke)(void *, ...);
    struct Block_descriptor_1 *descriptor;
    // imported variables
};
```
在 `objc` 中，根据对象的定义，凡是首地址是 `*isa` 的结构体指针，都可以认为是对象(id)。这样在 `objc` 中，`block` 实际上就算是对象。

为了查看编译器具体的工作，这里用clang重写一段代码

```
void foo_(){
    int i = 2;
    NSNumber *num = @3;
 
    long (^myBlock)(void) = ^long() {
        return i * num.intValue;
    };
 
    long r = myBlock();
}
```

上面这是一个很简单的 `block`，捕获了两个变量：一个`int`，一个`NSNumber`。
用clang翻译成C++后，稍微简化和调整

```
struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
};
 
struct __foo_block_desc_0 {
    size_t reserved;
    size_t Block_size;
    void (*copy)(struct __foo_block_impl_0*, struct __foo_block_impl_0*);
    void (*dispose)(struct __foo_block_impl_0*);
};
 
//myBlock的数据结构定义
struct __foo_block_impl_0 {
    struct __block_impl impl;
    struct __foo_block_desc_0* Desc;
    int i;
    NSNumber *num;
};
 
//block数据的描述
static struct __foo_block_desc_0 __foo_block_desc_0_DATA = {
    0,
    sizeof(struct __foo_block_impl_0),
    __foo_block_copy_0,
    __foo_block_dispose_0
};
 
//block中的方法
static long __foo_block_func_0(struct __foo_block_impl_0 *__cself) {
    int i = __cself->i; // bound by copy
    NSNumber *num = __cself->num; // bound by copy
 
    return i * num.intValue;
}
 
void foo(){
    int i = 2;
    NSNumber *num = @3;
 
    struct __foo_block_impl_0 myBlockT;
    struct __foo_block_impl_0 *myBlock = &myBlockT;
    myBlock->impl.isa = &_NSConcreteStackBlock;
    myBlock->impl.Flags = 570425344;
    myBlock->impl.FuncPtr = __foo_block_func_0;
    myBlock->Desc = &__foo_block_desc_0_DATA;
    myBlock->i = i;
    myBlock->num = num;
 
    long r = myBlock->impl.FuncPtr(myBlock);
}
```

编译器会根据`block`捕获的变量，生成具体的结构体定义。`block`内部的代码将会提取出来，成为一个单独的C函数。创建`block`时，实际就是在方法中声明一个`struct`，并且初始化该`struct`的成员。而执行`block`时，就是调用那个单独的`C`函数，并把该`struct`指针传递过去。
`block`中包含了被引用的自由变量(由`struct`持有)，也包含了控制成分的代码块(由函数指针持有)，符合闭包(`closure`)的概念。

### 2.block的Copy
`block`中的`isa`指向的是该`block`的`Class`。在`block runtime`中，定义了6种类：
`_NSConcreteStackBlock`    栈上的`block`
`_NSConcreteMallocBlock`    堆上的`block`
`_NSConcreteGlobalBlock`    作为全局变量的`block`（数据域(.data域)）
`_NSConcreteWeakBlockVariable`
`_NSConcreteAutoBlock`
`_NSConcreteFinalizingBlock`
其中我们能接触到的主要是前3种，后三种用于GC不再讨论。
关于前三种的区分以及作用域与 copy 操作可以看[这篇文章](https://www.jianshu.com/p/83e574bea682)

上面代码可以看到，当`struct`第一次被创建时，它是存在于该函数的栈帧上的，其`Class`是固定的`_NSConcreteStackBlock`。其捕获的变量是会赋值到结构体的成员上，所以当`block`初始化完成后，捕获到的变量不能更改。
当函数返回时，函数的栈帧被销毁，这个`block`的内存也会被清除。所以在函数结束后仍然需要这个`block`时，就必须用 `Block_copy()` 方法将它拷贝到堆上。这个方法的核心动作很简单：申请内存，将栈数据复制过去，将`Class`改一下，最后向捕获到的对象发送`retain`，增加`block`的引用计数。详细代码可以直接[点这里](http://opensource.apple.com/source/libclosure/libclosure-63/runtime.c)查看。
(所以 `MRC` 下需要 `copy` 操作，但是 `ARC` 下 `copy` 与 `strong` 其实都一样, 因为 `block` 的 `retain` 就是用 `copy` 来实现的，不过还是习惯用 `copy` 修饰)

```
struct Block_layout *result = malloc(aBlock->descriptor->size);
memmove(result, aBlock, aBlock->descriptor->size);
result->isa = _NSConcreteMallocBlock;
_Block_call_copy_helper(result, aBlock);
return result;
```

### 3.__block类型的变量

默认`block`捕获到的变量，都是赋值给`block`的结构体的，相当于`const`不可改。为了让`block`能访问并修改外部变量，需要加上`__block`修饰词。

```
void foo(){
    __block int count = 3;
    void(^myBlock)(void) = ^{
        count *= 2;
    };
    myBlock();
}
```

`clang`重写一下

```
struct Block_byref { //Block_private.h中的定义
    void *isa;
    struct Block_byref *forwarding;
    volatile int32_t flags; // contains ref count
    uint32_t size;
};
 
//__block count的实现
struct __Block_byref_count_0 {
    void *__isa;
    __Block_byref_count_0 *__forwarding;
    int __flags;
    int __size;
    int count;
};
 
void foo_(){
    __attribute__((__blocks__(byref))) __Block_byref_count_0 count = {(void*)0,(__Block_byref_count_0 *)&count, 0, sizeof(__Block_byref_count_0), 1};
 
    void(*myBlock)(void) = (void (*)())&__foo__block_impl_0((void *)__foo__block_func_0, &__foo__block_desc_0_DATA, (__Block_byref_count_0 *)&count, 570425344);
 
    ((void (*)(__block_impl *))((__block_impl *)myBlock)->FuncPtr)((__block_impl *)myBlock);
}
```

因为加了个`__block`，原本的`int`值的位置变成了一个`struct`（`struct __Block_byref`）。这个`struct`的首地址同样为`*isa`。正是如此，这个值才能被`block`共享、并且不受栈帧生命周期的限制、在`block`被`copy`后，能够随着`block`复制到堆上。
由于`block`中一直有一个指针指向`value`，所以`block`内部对它的修改，可以影响到`block`外部的变量。因为block修改的就是那个外部变量而不是外部变量的副本。
`__block`修饰的变量被封装在结构体中，`block`内部持有对这个结构体的强引用。

### 4.使用注意事项

我们使用一个`block`的时候，其实就等于在一个方法里面调用另外一个方法。

#### 1.内存
`Block_copy()`和`Block_release()`必须一一匹配，否则会内存泄漏或`crash`。
`__block`这个修饰词会将原本的简单类型转化为较大的`struct`，这会给内存、调用带来额外的开销，使用时需要注意。

#### 2.ARC
在开启`ARC`后，`block`的内存会比较微妙。`ARC`会自动处理`block`的内存，不用手动`copy/release`。
但是，和非`ARC`的情况有所不同：

```
void (^aBlock)(void);
aBlock = ^{ printf("ok"); };
```

`block`是对象，所以这个`aBlock`默认是有`__strong`修饰符的，即`aBlock`对该`block`有`strong references`。即`aBlock`在被赋值的那一刻，这个`block`会被`copy`。所以，`ARC`开启后，所能接触到的`block`基本都是在堆上的。

#### 3.循环引用
当`block`被`copy`之后(如开启了`ARC`、或把`block`放入`dispatch queue`)，该`block`对它捕获的对象产生`strong references` (非`ARC`下是`retain`),所以有时需要避免`block copy`后产生的循环引用。
如果用`self`引用了`block`，`block`又捕获了`self`，这样就会有循环引用。因此，需要用 `__weak` 来修饰`self`。
如果捕获到的是当前对象的成员变量对象，同样也会造成对`self`的引用，同样也要避免。
或者是调用完之后将 `block` 置 `nil`，这样只能调用一次。



## Swift 闭包

`OC`中的`__block` 修饰符本质上是通过截获变量的引用（或指针）来达到在闭包内修改被截获的变量的目的。
在`Swift`中闭包默认会截取变量的引用，也就是说所有变量默认情况下都是加了`__block` 修饰符的。

### 捕获列表
写在闭包的方括号之间，紧跟闭包的左括号（并且在闭包的参数或返回类型之前）。
可以在创建闭包时捕获变量的值（而不是变量的引用），如果一个变量是引用类型，闭包并没有真正（强引用）捕获变量的引用，而是捕获了一个针对原始实例的拷贝。

```
var x = 42  
let f = {  
    // [x] in //如果取消注释，结果是42
    print(x)
}
x = 43  
f() // 结果是43  

class A {
    var name: String = "A"
    init(name: String) {
        self.name = name
    }
}

var block: (() -> ())?
if true {
    var a: A = A(name: "old name")
    block = {
        //[a] in //加上这行就会打印 old name 
        print(a.name)
    }
    a = A(name: "new name")
}
block?() //new name
``` 

上面捕获列表 `[a]` 相当于执行了 `let aCopy = a` 而使用的是 `aCopy`。

1. 在 `Swift` 闭包中使用的所有外部变量，闭包会自动捕获这些变量的引用。
2. 在闭包执行时，会根据这些变量引用得到所对应的具体值。
3. 因为我们捕获的是变量的引用（而不是变量自身的值），所以你可以在闭包内部修改变量的值（当然变量要声明为 `var`，而不能是 `let`）。
4. 你可以在闭包创建时获取变量中的值，然后把它存储到本地常量中，而不是捕获变量的引用。我们可以使用带中括号的捕获列表来实现。


### Swift循环引用
不管是否显示的把变量写进捕获列表，闭包都会对对象有强引用。如果闭包是某个对象的属性，而且闭包中截获了对象本身，或对象的某个属性，就会导致循环引用。这和OC中是完全一样的。解决方法是在捕获列表中把被截获的变量标记为`weak`或`unowned`。

为了避免`weak`变量在闭包中提前被释放，我们需要在`block`一开始强引用它。这在`OC`部分已经讲过如何使用了。`Swift`中实现`Weak-Strong Dance`一般有三种方法。分别是最简单的`if let`可选绑定、标准库的`withExtendedLifetime`方法和自定义的`withExtendedLifetime`方法。




## 对比
1. `OC`中默认截获变量，`Swift`默认截获变量的引用
2. `Swift`中没有`__block`修饰符，但是多了截获列表。通过把截获的变量标记为`weak`避免引用循环
3. 两者都有`Weak-Strong Dance`，不过这一点上`OC`的写法更简单。
4. 在使用可选类型时，要明确闭包截获了可选类型还是实例变量。这样才能正确判断是否发生循环引用。





















