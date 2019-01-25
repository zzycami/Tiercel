<div align=center>
<img src="https://github.com/Danie1s/Tiercel/blob/master/Images/logo.png"/>
</div>

[![Version](https://img.shields.io/cocoapods/v/Tiercel.svg?style=flat)](http://cocoapods.org/pods/Tiercel)
[![Platform](https://img.shields.io/cocoapods/p/Tiercel.svg?style=flat)](http://cocoapods.org/pods/Tiercel)
[![Language](https://img.shields.io/badge/language-swift-red.svg?style=flat)]()
[![Support](https://img.shields.io/badge/support-iOS%208%2B%20-brightgreen.svg?style=flat)](https://www.apple.com/nl/ios/)
[![License](https://img.shields.io/cocoapods/l/Tiercel.svg?style=flat)](http://cocoapods.org/pods/Tiercel)


Tiercel是一个简单易用且功能丰富的纯Swift下载框架。最大的特点就是拥有强大的任务管理功能和可以直接获取下载速度等常见的下载信息，只要加上一些简单的UI，就可以实现一个下载类APP的大部分功能。


- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Example](#example)
- [Usage](#usage)
  - [基本用法](#基本用法)
  - [TRManager](#trmanager)
  - [TRDownloadTask](#trdownloadtask)
  - [TRCache](#trcache)
- [License](#license)



## WARNING:

这是Tiercel 1版本的分支，下载实现基于`URLSessionDataTask`，不支持后台下载，已经不再更新，如果需要后台下载和更强大的功能，请使用`master`的最新版本



## Features:

- [x] 支持大文件下载
- [x] 支持离线断点续传，APP关闭后依然可以恢复所有下载任务
- [x] 精细的任务管理，每个下载任务都可以单独管理操作和状态回调
- [x] 支持多个下载模块，每个模块拥有一个管理者，每个模块互不影响
- [x] 下载模块的管理者拥有总任务的状态回调
- [x] 内置了下载速度等常见的下载信息，并且可以选择是否持久化下载任务信息
- [x] 链式语法调用
- [x] 支持控制下载任务的最大并发数
- [x] 线程安全

## Requirements

- iOS 8.0+
- Xcode 10.0+
- Swift 4.2+

## Installation

### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

> CocoaPods 1.1+ is required to build Tiercel.

To integrate Tiercel into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
use_frameworks!

target '<Your Target Name>' do
    pod 'Tiercel', :git => 'https://github.com/Danie1s/Tiercel.git', :branch => 'dataTask'
end
```

Then, run the following command:

```bash
$ pod install
```

### Manually

If you prefer not to use any of the aforementioned dependency managers, you can integrate Tiercel into your project manually.

## Example

To run the example project, clone the repo, and run `Tiercel.xcodeproj` .

<img src="https://github.com/Danie1s/Tiercel/blob/master/Images/3.gif" width="50%" height="50%">

<img src="https://github.com/Danie1s/Tiercel/blob/master/Images/4.gif" width="50%" height="50%">

## Usage

### 基本用法

只需要简单的几行代码即可开启下载

```swift
let URLString = "http://api.gfs100.cn/upload/20171219/201712191530562229.mp4"
let downloadManager = TRManager()
// 创建下载任务并且开启下载，同时返回可选类型的TRDownloadTask实例，如果URLString无效，则返回nil
let task = downloadManager.download(URLString)

// 批量创建下载任务并且开启下载，返回有效URLString对应的任务数组，URLStrings需要跟fileNames一一对应
let tasks = downloadManager.multiDownload(URLStrings)
```

如果需要设置回调

```swift
// 可以在创建下载任务的时候设置

// 回调闭包的参数是TRDownloadTask实例，可以得到所有相关的信息
// 回调闭包都是在主线程运行
// progress 闭包：如果任务正在下载，就会触发
// success 闭包：任务已经下载过，或者下载完成，都会出发，这时候task.status == .completed
// failure 闭包：只要task.status != .completed，就会触发：
//    1. 暂停任务，这时候task.status == .suspend
//    2. 任务下载失败，这时候task.status == .failed
//    3. 取消任务，这时候task.status == .cancel
//    4. 移除任务，这时候task.status == .remove
downloadManager.download(URLString, fileName: "视频.mp4", progressHandler: { (task) in
    let progress = task.progress.fractionCompleted
    print("下载中, 进度：\(progress)")
}, successHandler: { (task) in
    print("下载成功")
}) { (task) in
    print("下载失败")
}

// 也可以拿到下载任务后，再对它进行设置
let task =  downloadManager.download(URLString)
                                      
task.progress { (task) in
     let progress = task.progress.fractionCompleted
     print("下载中, 进度：\(progress)")
    }
    .success({ (task) in
      	print("下载完成")
    })
    .failure({  (task) in
		print("下载失败")
    })
```

下载任务的管理和操作。**在Tiercel中，URLString是下载任务的唯一标识，如果需要对下载任务进行操作，则使用TRManager实例对URLString进行操作。**

```swift
// 创建下载任务并且开启下载，同时返回可选类型的TRDownloadTask实例，如果URLString无效，则返回nil
let task = downloadManager.download(URLString)
// 根据URLString查找下载任务，返回可选类型的TRTask实例，如果不存在，则返回nil
let task = downloadManager.fetchTask(URLString)

// 开始下载
// 如果调用suspend暂停了下载，可以调用这个方法继续下载
downloadManager.start(URLString)

// 暂停下载
downloadManager.suspend(URLString)

// 取消下载，没有下载完成的任务会被移除，但保留没有下载完成的缓存文件，已经下载完成的不受影响
downloadManager.cancel(URLString)

// 移除下载，已经完成的任务也会被移除，没有下载完成的缓存文件会被删除，可以选择是否保留已经下载完成的文件
downloadManager.remove(URLString, completely: false)

// 除了可以对单个任务进行操作，TRManager也提供了对所有任务同时操作的API
downloadManager.totalStart()
downloadManager.totalSuspend()
downloadManager.totalCancel()
downloadManager.totalRemove(completely: false)
```



### TRManager

TRManager是下载任务的管理者，管理当前模块所有下载任务，要使用Tiercel进行下载，必须要先创建TRManager实例。Tiercel没有设计成单例模式，如果需要多个下载模块，可以创建多个不同的TRManager实例。

```swift
///  初始化方法
///
/// - Parameters:
///   - name: 设置TRManager实例的名字，区分不同的下载模块，每个模块中下载相关的文件会保存到对应的沙盒目录
///   - MaximumRunning: 下载的最大并发数
///   - isStoreInfo: 是否把下载任务的相关信息持久化到沙盒，如果是，则初始化完成后自动恢复上次的任务
public init(_ name: String? = nil, MaximumRunning: Int? = nil, isStoreInfo: Bool = false) {
    // 实现的代码... 
}
```

TRManager作为所有下载任务的管理者，也可以设置回调

```swift
// 回调闭包的参数是TRManager实例，可以得到所有相关的信息
// 回调闭包都是在主线程运行
// progress 闭包：只要有一个任务正在下载，就会触发
// success 闭包：只有一种情况会触发：
//    所有任务都下载成功(取消和移除的任务会被移除然后销毁，不再被manager管理) ，这时候manager.status == .completed
// failure 闭包：只要manager.status != .completed，就会触发：
//    1. 调用全部暂停的方法，或者没有等待运行的任务，也没有正在运行的任务，这时候manager.status == .suspended
//    2. 所有任务都结束，但有一个或者多个是失败的，这时候manager.status == .failed
//    3. 调用全部取消的方法，或者剩下一个任务的时候把这个任务取消，这时候manager.status == .canceled
//    4. 调用全部移除的方法，或者剩下一个任务的时候把这个任务移除，这时候manager.status == .removed
downloadManager.progress { (manager) in
    let progress = manager.progress.fractionCompleted
    print("downloadManager运行中, 总进度：\(progress)")
    }.success { (manager) in
         print("所有下载任务都成功了")
    }.failure { (manager) in
         if manager.status == .suspended {
            print("所有下载任务都暂停了")
        } else if manager.status == .failed {
            print("存在下载失败的任务")
        } else if manager.status == .canceled {
            print("所有下载任务都取消了")
        } else if manager.status == .removed {
            print("所有下载任务都移除了")
        }
}
```

**Tiercel的销毁**

```swift
// 由于Tiercel是使用URLSession实现的，session需要手动销毁，所以当不再需要使用Tiercel也需要手动销毁
// 一般在控制器中添加以下代码
deinit {
    downloadManager.invalidate()
}
```

TRManager的主要属性

```swift
// 设置内置日志打印等级，如果为none则不打印
public static var logLevel: TRLogLevel = .high
// 默认对networkActivityIndicator进行管理，可以取消
public static var isControlNetworkActivityIndicator = true
// 设置是否创建任务后马上下载，默认为是
public var isStartDownloadImmediately = true
// TRManager的状态
public var status: TRStatus = .waiting
// TRManager的缓存管理实例
public var cache: TRCache
// TRManager的进度
public var progress: Progress
// 设置请求超时时间
public var timeoutIntervalForRequest = 30.0
// 所有下载中的任务加起来的总速度
public private(set) var speed: Int64 = 0
// 所有下载中的任务需要的剩余时间
public private(set) var timeRemaining: Int64 = 0
// manager管理的下载任务，取消和移除的任务会被销毁，但操作是异步的，在回调闭包里面获取才能保证正确
public var tasks: [TRTask] = []
```



### TRDownloadTask

TRDownloadTask是Tiercel中的下载任务类，继承自TRTask。**在Tiercel中，URLString是下载任务的唯一标识，URLString代表着任务，如果需要对下载任务进行操作，则使用TRManager实例对URLString进行操作。**所以TRDownloadTask实例都是由TRManager实例创建，单独创建没有意义。

主要属性

```swift
// 保存到沙盒的下载文件的文件名，如果在创建的时候没有设置，则默认使用url的最后一部分
public internal(set) var fileName: String
// 下载任务对应的URLString
public var URLString: String
// 下载任务的状态
public var status: TRStatus = .waiting
// 下载任务的进度
public var progress: Progress = Progress()
// 下载任务的开始日期
public var startDate: TimeInterval = 0
// 下载任务的结束日期
public var endDate: TimeInterval = Date().timeIntervalSince1970
// 下载任务的速度
public var speed: Int64 = 0
// 下载任务的剩余时间
public var timeRemaining: Int64 = 0
// 下载文件路径
public var filePath: String
```

对下载任务操作，必须通过TRManager实例进行，不能用TRDownloadTask实例直接操作

- 开启
- 暂停
- 取消，没有完成的任务从TRManager实例中的tasks中移除，但保留没有下载完成的缓存文件，已经下载完成的任务不受影响
- 移除，已经完成的任务也会被移除，没有下载完成的缓存文件会被删除，已经下载完成的文件可以选择是否保留

**注意：对下载中的任务进行暂停、取消和移除操作，结果是异步回调的，在回调闭包里面获取状态才能保证正确**



### TRCache

TRCache是Tiercel中负责管理缓存下载任务信息和下载文件的类。TRCache实例一般作为TRManager实例的属性来使用，如果需要单独使用TRCache，那么只需要创建跟TRManager实例同样名字的TRCache实例即可操作对应模块的缓存信息和文件。Tiercel内置一个全局的`TRCache.default`单例，如果创建TRManager实例时`name`为`nil`，则对应的TRCache为`TRCache.default`。

```swift
/// 初始化方法
///
/// - Parameters:
///   - name: 设置TRCache实例的名字，一般由TRManager实例创建时传递
///   - isStoreInfo: 是否把下载任务的相关信息持久化到沙盒，一般由TRManager实例创建时传递
public init(_ name: String, isStoreInfo: Bool = false) {
    // 实现的代码...
}
```

主要属性

```swift
// 下载模块的目录路径
public let downloadPath: String

// 没有完成的下载文件缓存的目录路径
public let downloadTmpPath: String

// 下载完成的文件的目录路径
public let downloadFilePath: String
```

主要API分成几大类：

- 检查沙盒是否存在文件
- 移除跟下载任务相关的文件
- 保存跟下载任务相关的文件
- 读取下载任务相关的文件，获得下载任务相关的信息



## License

Tiercel is available under the MIT license. See the LICENSE file for more info.


