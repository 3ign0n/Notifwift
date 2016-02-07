# Notifwift

[![Version](https://img.shields.io/cocoapods/v/Notifwift.svg?style=flat)](http://cocoapods.org/pods/Notifwift)
[![License](https://img.shields.io/cocoapods/l/Notifwift.svg?style=flat)](http://cocoapods.org/pods/Notifwift)
[![Platform](https://img.shields.io/cocoapods/p/Notifwift.svg?style=flat)](http://cocoapods.org/pods/Notifwift)

Notifwift is the NS**Notif**icationCenter wrapper for S**wift**.

Notifwift resolves;

- cumbersome management of NSNotification observers
- ugly payload through userInfo `[String: AnyObject]` which cannot contain structs nor enum.
- long syntax to post/observe notifications


## Usage

```swift
    do {
        let nt = Notifwift()
        nt.observe(notificationName) { notification in
            print("Notifwift can observe NSNotification in simple way.", notification)
        }
        Notifwift.post(notificationName)
        //printed:
        // Notifwift can observe NSNotification in simple way. NSConcreteNotification 0x7fdfa0414b20 {name = Hoge}
    }
    
    Notifwift.post(notificationName)
    //printed nothing. Observers expire when the Notifwift instance(in this case) is destructed.
```

```swift
    let nt = Notifwift()
    nt.observePayload(notificationName) { (_, payload:String) in
        print("This closure observes nothing but NSNotification with String payload.", payload)
    }
    nt.observePayload(notificationName) { (_, payload:Int) in
        print("This closure observes nothing but NSNotification with Int payload.", payload)
    }
    
    Notifwift.post(notificationName, payload:"aaaa")
    //printed:
    // This closure observes nothing but NSNotification with String payload. aaaa
    
    Notifwift.post(notificationName, payload:1)
    //printed:
    // This closure observes nothing but NSNotification with Int payload. 1
```

```swift
    class Something {}
    class SubSomething: Something {}
    
    let nt = Notifwift()
    nt.observePayload(notificationName) { (_, p:Something) in
        print("Received Something.", p)
    }
    nt.observePayload(notificationName) { (_, p:SubSomething) in
        print("Received SubSomething. Yes, of course, Notifwift recognizes subtypes.", p)
    }
    
    Notifwift.post(notificationName, payload:Something())
    //printed:
    // Received Something. (Something #1)
    
    Notifwift.post(notificationName, payload:SubSomething())
    //printed:
    // Received Something. (SubSomething #1)
    // Received SubSomething. Yes, of course, Notifwift recognizes subtypes. (SubSomething #1)
```

```swift
    enum SomeResult {
        case Success(String)
        case Fail(NSError)
    }
    let nt = Notifwift()
    nt.observePayload(notificationName) { (_, p:SomeResult) in
        switch p {
        case .Success(let str):
            print("Any Type can be used as a payload", str)
        case .Fail(let err) where err.code == 403:
            print("not authorized")
        case .Fail(let err) where err.code == 404:
            print("not found")
        case .Fail(let err):
            print("Notifwift has a chemistry with Enum Associated Values.", err)
        }
    }
    
    Notifwift.post(notificationName, payload:SomeResult.Success("like this."))
    //printed:
    // Any Type can be used as a payload like this.

    Notifwift.post(notificationName, payload:SomeResult.Fail(NSError(domain: "", code: 0, userInfo: nil)))
    //printed:
    // Notifwift has a chemistry with Enum Associated Values. Error Domain= Code=0 "(null)"
```

```swift
    let obj1 = NSObject()
    let obj2 = NSObject()
    let nt = Notifwift()
    nt.observe(notificationName) { _ in
        print("Received from all objects")
    }
    nt.observe(notificationName, from: obj1) { _ in
        print("Received from obj1 only")
    }
    nt.observe(notificationName, from: obj2) { _ in
        print("Received from obj2 only")
    }
    
    Notifwift.post(notificationName, from: obj1)
    //printed:
    // Received from all objects
    // Received from obj1 only
    
    Notifwift.post(notificationName, from: obj2)
    //printed:
    // Received from all objects
    // Received from obj2 only
```


## Installation

Notifwift is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod "Notifwift"
```

## Author

[takasek](https://twitter.com/takasek)

## License

Notifwift is available under the MIT license. See the LICENSE file for more info.