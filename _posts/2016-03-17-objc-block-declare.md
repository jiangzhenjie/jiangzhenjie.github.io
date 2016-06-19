---
layout: post
title: Objc Block语法
modified: 2016-03-17
tags: [iOS ObjC]
share: false
---



### 局部变量

```objc
returnType (^blockName)(parameterTypes) = ^(parameters) {...};
```

例子：

```objc
void (^handler)(NSString *, NSError *) = ^(NSString *string, NSError *error) {
    NSLog(@"%@ %@", string, error);
};
[self testWithParams:@{@"key": @"value"} completionHandler:handler];
```
    
### property

```objc
@property (nonatomic, copy) returnType (^blockName)(parameterTypes);
```
    
例子：

```objc
@property (nonatomic, copy) void (^handler)(NSString *, NSError *);
```
    
### 方法声明

```objc
- (void)methodWithBlock:(returnType (^)(parameters))blockName;
```
    
例子：

```objc
- (void)testWithParams:(NSDictionary *)params completionHandler:(void(^)(NSString *string, NSError *error))handler;
```
    
### 方法调用

```objc
[obj methodWithBlock:^(parameters) {...}];
```
    
例子：

```objc
[self testWithParams:@{@"key": @"value"} completionHandler:^(NSString *string, NSError *error) {
    NSLog(@"%@ %@", string, error);
}];
```
    
### typedef定义

```objc
typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^(parameters) {...};
```
    
例子：

```objc
typedef void (^MyHandler)(NSString *, NSError *);
MyHandler handler = ^(NSString *string, NSError *error) {
    NSLog(@"%@ %@", string, error);
};
[self testWithParams:@{@"key": @"value"} completionHandler:handler];
```
    
### 参考

* [How Do I Declare A Block in Objective-C?](http://fuckingblocksyntax.com/)