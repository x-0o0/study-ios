# 멀티캐스트 델리게이트 패턴

## 간단 요약

`addDelegate(_:)`, `removeDelegate(_:)` 를 쓰는 경우가 이 패턴이다.

## 개요

<img width="649" alt="Screen Shot 2022-06-09 at 10 30 37 PM" src="https://user-images.githubusercontent.com/53814741/172859108-ddc04b32-b3d6-4810-a133-5eee43ccdb71.png">

보통 델리게이트 패턴은 하나의 delegate 객체만 필요하여 아래와 같은 형태일 것이다.

```swift
class SomeClassA {
    weak var delegate: Delegate?
    
    func notify() {
        delegate.didNotify()
    }
}

class SomeViewController: UIViewController, Delegate {
    let someInstance = SomeClassA()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        someInstance.delegate = self
    }
    
    func didNotify() { ... }
}
```

하지만 만약 delegate 객체가 여러개가 필요하다면? 이런 경우 멀티캐스트 델리게이트 패턴 (Multicast delegate pattern)을 쓴다.

### Delegate Protocol

델리게이트 프로토콜.

```swift
// MARK: - Delegate Protocol
public protocol EmergencyResponding {
  func notifyFire(at location: String)
  func notifyCarCrash(at location: String)
}
```

### Multicast Delegate

`delegate` 구현체들을 배열로 보관하고 있을 클래스. 구현체들의 타입은 모르지만, 어쨋든 어떤 하나의 델리게이트 프로토콜을 준수할거라는 것만 알고있으면 됨.

```swift
public class MulticastDelegate<ProtocolType> {
  private var delegateWrappers: [DelegateWrapper]
  
  public var delegates: [ProtocolType] {
    delegateWrappers.compactMap { $0.delegate as? ProtocolType }
  }

  init(delegates: [ProtocolType] = []) {
    delegateWrappers = delegates
      .map { DelegateWrapper($0 as AnyObject) }
  }
  
  public func addDelegate(_ delegate: ProtocolType) {
    let wrapper = DelegateWrapper(delegate as AnyObject)
    delegateWrappers.append(wrapper)
  }

  public func removeDelegate(_ delegate: ProtocolType) {
    guard let index = delegateWrappers.firstIndex(where: { $0.delegate === (delegate as AnyObject) }) else { return }
    delegateWrappers.remove(at: index)
  }
  
  public func invokeDelegates(_ closure: (ProtocolType) -> ()) {
    delegates.forEach { closure($0) }
  }
  
  // MARK: - DelegateWrapper
  // 2
  private class DelegateWrapper {
    weak var delegate: AnyObject?
    init(_ delegate: AnyObject) {
      self.delegate = delegate
    }
} }
```

### Delegates

`Delegate` 를 구현하고 있는 클래스들 (ConcreteDegate)

```swift
// MARK: - Delegates
public class FireStation: EmergencyResponding {
  public func notifyFire(at location: String) { ... }
  public func notifyCarCrash(at location: String) { ... }
}

public class PoliceStation: EmergencyResponding {
  public func notifyFire(at location: String) { ... }
  public func notifyCarCrash(at location: String) { ... }
}
```

### Delegating Object

`MutlicastDelegate` 를 가질 프로퍼티. e.g., `ViewController`

```swift
// MARK: - Delegating Object
public class DispatchSystem {
  let multicastDelegate = MulticastDelegate<EmergencyResponding>()
}
```

**Uses**

```swift
let dispatch = DispatchSystem()

var policeStation: PoliceStation = PoliceStation()
var fireStation: FireStation = FireStation()

dispatch.multicastDelegate.addDelegate(policeStation)
dispatch.multicastDelegate.addDelegate(fireStation)
```

## 참고

> **Note** 
> 
> Apple introduced a new Multicast type in the Combine framework in Swift 5.1. 
> This is different than the MulticastDelegate introduced in this chapter. 
> It allows you to handle multiple Publisher events. 
> In such, this could be used as an alternative to the multicast delegate pattern as part of a reactive achitecture.
> 
> Multicast is an advanced topic in the Combine framework, and it's beyond the scope of this chapter. 
> If you'd like to learn more about Combine, check out our book about it, 
> Combine: Asynchronous Programming with Swift (http://bit.ly/ swift-combine).
