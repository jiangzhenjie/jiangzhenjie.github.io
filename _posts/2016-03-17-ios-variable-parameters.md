---
layout: post
title: iOS可变参数
modified: 2016-03-17
tags: [iOS]
share: false
---


今天在写一个可变参数方法时，遇到一些问题，故此做记录。

以下面的例子说明：

    - (NSString *)testVariableParameters:(NSString *)param1, ... {
    
        NSMutableString *query = [NSMutableString string];
        if (param1) {
        
            va_list args;
            va_start(args, param1);
        
            for (NSString *arg = param1; arg != nil; arg = va_arg(args, NSString *)) {
                [query appendFormat:@"%@&", arg];
            }
        
            [query deleteCharactersInRange:NSMakeRange(query.length - 1, 1)];
        
            va_end(args);
        }
    
        return query;
    }

调用上面的函数：

    [self testVariableParameters:@"param1", @"param2", @"param3", @"param4"];

会发现程序直接crash了，crash的信息是`EXC_BAD_ACCESS`。
在上面的`for`循环中打印一下取得参数的情况，会得到如下信息：

    2016-03-16 23:12:50.057 Demo[1009:38552] param1
    2016-03-16 23:12:50.058 Demo[1009:38552] param2
    2016-03-16 23:12:50.059 Demo[1009:38552] param3
    2016-03-16 23:12:50.059 Demo[1009:38552] param4
    2016-03-16 23:12:50.060 Demo[1009:38552] <UIButton: 0x7faf0ad22230; frame = (164.5 318.5; 46 30); opaque = NO; autoresize = RM+BM; layer = <CALayer: 0x7faf0ad21810>>
    
从上面的信息可以看出，传进来的可变参数全部都取出来了，但是`for`循环并没有结束，于是出现了类似`越界访问`的问题。

> 那么该如何解决呢？

答案是**添加哨兵**。

如下，在参数末尾添加一个`nil`：

    [self testVariableParameters:@"param1", @"param2", @"param3", @"param4", nil];
    
运行，程序正常。

> 那么，每次都需要调用方主动在末尾添加`nil`吗？

研究了一下`+[NSArray arrayWithObjects:]`方法，发现其后面有一个`NS_REQUIRES_NIL_TERMINATION`宏，看来像是在末尾自动添加`nil`。

    + (instancetype)arrayWithObjects:(ObjectType)firstObj, ... NS_REQUIRES_NIL_TERMINATION;

在上面的方法后面加上`NS_REQUIRES_NIL_TERMINATION`宏，如下：

    - (NSString *)testVariableParameters:(NSString *)param1, ... NS_REQUIRES_NIL_TERMINATION { ... }

然后调用，发现在可变参数末尾自动补上了`nil`:

    [self testVariableParameters:<#(NSString *), ...#>, nil];

> 另外一个问题，在我们调用`+[NSString stringWithFormat:]`方法的时候，发现可变参数末尾并没有`nil`，
> 可是在调用`+[NSArray arrayWithObjects:]`的时候，可变参数末尾就需要`nil`，这是为什么？

答案是**当参数个数明确知道的时候，就不需要`nil`，参数个数不明确知道的时候，就需要`nil`作为哨兵**

`+[NSString stringWithFormat:]`在使用的时候，因为有`formatter`的缘故，所以参数个数是明确的。

    NSString * string = [NSString stringWithFormat:@"%@, %@, %@", @1, @2, @3];

当然，在使用可变参数的时候，也可以将第一个参数规定为参数的个数，第一个参数以后才是真正的参数，这样也可以避免末尾加`nil`。
