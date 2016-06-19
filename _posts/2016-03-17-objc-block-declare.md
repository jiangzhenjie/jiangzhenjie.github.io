---
layout: post
title: Objc Block语法
modified: 2016-03-17
tags: [iOS ObjC]
share: false
---



### 局部变量

    returnType (^blockName)(parameterTypes) = ^(parameters) {...};

例子：

    void (^handler)(NSString *, NSError *) = ^(NSString *string, NSError *error) {
        NSLog(@"%@ %@", string, error);
    };
    [self testWithParams:@{@"key": @"value"} completionHandler:handler];
    
### property

    @property (nonatomic, copy) returnType (^blockName)(parameterTypes);
    
例子：

    @property (nonatomic, copy) void (^handler)(NSString *, NSError *);
    
### 方法声明

    - (void)methodWithBlock:(returnType (^)(parameters))blockName;
    
例子：

    - (void)testWithParams:(NSDictionary *)params completionHandler:(void(^)(NSString *string, NSError *error))handler;
    
### 方法调用

    [obj methodWithBlock:^(parameters) {...}];
    
例子：

    [self testWithParams:@{@"key": @"value"} completionHandler:^(NSString *string, NSError *error) {
        NSLog(@"%@ %@", string, error);
    }];
    
### typedef定义

    typedef returnType (^TypeName)(parameterTypes);
    TypeName blockName = ^(parameters) {...};
    
例子：

    typedef void (^MyHandler)(NSString *, NSError *);
    MyHandler handler = ^(NSString *string, NSError *error) {
        NSLog(@"%@ %@", string, error);
    };
    [self testWithParams:@{@"key": @"value"} completionHandler:handler];
    
### 参考

* [How Do I Declare A Block in Objective-C?](http://fuckingblocksyntax.com/)