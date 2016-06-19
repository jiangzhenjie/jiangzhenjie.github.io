---
layout: post
title: iOS Keychain Sharing
modified: 2016-04-06
tags: [iOS]
share: false
---



在iOS系统中，通常使用钥匙串（`Keychain`）存储一些敏感信息，比如密码，证书等等。每个应用程序只能访问自己的`Kechain`条目。

## Keychain 结构

每个`Keychain`包含多个`Keychain item`，每个`Keychain item`包含数据（data）和一系列属性（attributes）。
对于一些需要被保护的`Keychain item`，比如密码或者用于加密的私钥，`Keychain`会对其数据进行加密保护；
而对一些不需要被保护的数据，比如证书，`Keychain`不会将其数据加密。
`Keychain item`的属性(attributes)根据不同的类型item而不同，最常用的两种item的类型是：因特网密码（Internet passwords）和 通用密码（generic passwords）。
因特网密码item包含安全域名，协议类型和路径等属性。
通用密码item包含服务(service)，账户(account)等属性。通常在开发中，使用通用密码item来存储用户密码或者一些敏感的数据。

## Keychain 访问

如上节所述，每个`Keychain item`包含数据和一系列属性，因此访问`Keychain item`是通过定义一个查询字典（query dictionary）,然后传递给系统提供的操作方法（增删改查）来完成。

### 查询字典（query dictionary）

查询字典是访问`Keychain item`的依据，因此对于访问`Keychain item`，查询字典的构建为首要操作。

通常，一个典型的查询字典包含：

* 一个类别键值对，用于指定`Keychain item`的类型。比如 `kSecClass: kSecClassGenericPassword` 指定了需要查询的item的类型为通用密码类型。
* 一个或多个键值对，用于指定`Keychain item`满足的属性。比如 `kSecAttrService: "Baidu"`和`kSecAttrAccount: "Jboy92"` 指定了需要查询的item的服务名称是`Baidu`，账户是`Jboy92`。
* 一个或多个查询键值对，用于指定进一步细化搜索。比如 `kSecMatchCaseInsensitive: kCFBooleanTrue` 指定了查询是忽略大小写的。
* 一个返回结果类型键值对，用于指定期望返回的结果类型。比如 `kSecReturnData: kCFBooleanTrue` 指定了需要返回item的数据。

不同类型的`Keychain item`的属性有所不同，应该按照`Keychain item`的类型来指定相应的查询属性。

举个例子，假设一个场景是从`Keychain`里面取出`Jboy92`这个帐号登录`Baidu`的密码（假设这个密码存储在`Baidu`这个service下）。
如下代码：

```swift
var query = [String: AnyObject]()
query[kSecClass as String] = kSecClassGenericPassword
query[kSecAttrService as String] = "Baidu"
query[kSecAttrAccount as String] = "Jboy92"
query[kSecMatchCaseInsensitive as String] = kCFBooleanTrue
query[kSecReturnAttributes as String] = kCFBooleanTrue
query[kSecReturnData as String] = kCFBooleanTrue
```

上面代码中，`kSecReturnAttributes`指定了需要返回item的属性，`kSecReturnData`指定了需要返回item的数据，即这里的密码。

### 操作函数

对于`Keychain item`的操作，系统提供了四个方法，分别是增删改查：

* `SecItemAdd`
* `SecItemDelete`
* `SecItemUpdate`
* `SecItemCopyMatching`

这几个方法的使用比较简单，最主要还是定义好查询字典。继续以上面的场景为例，代码如下：

```swift
func addPassword() {
    
    var query = [String: AnyObject]()
    query[kSecClass as String] = kSecClassGenericPassword
    query[kSecAttrService as String] = "Baidu"
    query[kSecAttrAccount as String] = "Jboy92"
    query[kSecValueData as String] = "test_password".dataUsingEncoding(NSUTF8StringEncoding)

    let status = SecItemAdd(query, nil)
    
    if status == errSecSuccess {
        print("Add password succeed")
    } else {
        print("Add password failed")
    }
}
    
func deletePassword() {
    
    var query = [String: AnyObject]()
    query[kSecClass as String] = kSecClassGenericPassword
    query[kSecAttrService as String] = "Baidu"
    query[kSecAttrAccount as String] = "Jboy92"
    
    let status = SecItemDelete(query)
    
    if status == errSecSuccess {
        print("Delete password succeed")
    } else {
        print("Delete password failed");
    }
    
}
    
func updatePassword() {
    
    var query = [String: AnyObject]()
    query[kSecClass as String] = kSecClassGenericPassword
    query[kSecAttrService as String] = "Baidu"
    query[kSecAttrAccount as String] = "Jboy92"
    
    var updateAttrs = [String: AnyObject]()
    updateAttrs[kSecValueData as String] = "update_password".dataUsingEncoding(NSUTF8StringEncoding)
    
    let status = SecItemUpdate(query, updateAttrs)
    
    if status == errSecSuccess {
        print("Add password succeed")
    } else {
        print("Add password failed")
    }
    
}
    
func queryPassword() {
    
    var query = [String: AnyObject]()
    query[kSecClass as String] = kSecClassGenericPassword
    query[kSecAttrService as String] = "Baidu"
    query[kSecAttrAccount as String] = "Jboy92"
    query[kSecMatchLimit as String] = kSecMatchLimitOne
    query[kSecReturnData as String] = kCFBooleanTrue
    
    var result: AnyObject?
    let status = withUnsafeMutablePointer(&result) {
        SecItemCopyMatching(query, UnsafeMutablePointer($0))
    }
    
    if status == errSecSuccess {
        if let result = result as? NSData {
            print(String(data: result, encoding: NSUTF8StringEncoding))
        }
    } else {
        print("No password");
    }
    
}
```

一般会封装一个工具类来处理`Keychain`的操作，如[Keychain](https://github.com/jiangzhenjie/KeychainDemo/blob/master/KeychainDemo/Keychain.swift)或者Github上其他[开源库](https://github.com/search?o=desc&q=keychain&s=stars&type=Repositories&utf8=%E2%9C%93)

### 使用Keychain实现应用间共享数据(todo)

## 参考

* [Keychain Services Programming Guide](https://developer.apple.com/library/mac/documentation/Security/Conceptual/keychainServConcepts/01introduction/introduction.html#//apple_ref/doc/uid/TP30000897-CH203-TP1)
* [Keychain Services Reference](https://developer.apple.com/library/mac/documentation/Security/Reference/keychainservices/index.html#//apple_ref/doc/uid/TP30000898)
* [SecItem.h](https://opensource.apple.com/source/libsecurity_keychain/libsecurity_keychain-38259/lib/SecItem.h)




