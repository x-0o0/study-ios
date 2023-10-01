# State 패턴

런타임 동안 하나의 객체의 행동을 변경할 수 있도록 해주는 패턴.

## 개요

> State 란 주어진 객체가 주어진 시간동안 어떻게 행동해야하는지 설명하는 데이터의 집합이다.

## 종류

<img width="735" alt="Screen Shot 2022-05-07 at 11 09 27 PM" src="https://user-images.githubusercontent.com/53814741/167257869-94b57675-b795-486c-bb91-cc505215676b.png">

이 패턴은 3가지 부분으로 나뉜다.

### **컨텍스트 (context)**

현재의 상태를 갖고 행동이 바뀌는 객체이다.

```swift
class TrafficLight {
  var currentState: TrafficLightState
  
  init() {
    self.currentState = RedLightState()
  }
  
  func transition(to state: TrafficLightState) {
    currentState = state
    currentState.apply(to: self)
  }
}
```

### **상태 프로토콜(state protocol)**

필요한 함수와 프로퍼티들을 정의한다. 보통은 프로토콜 자리에 베이스 상태 클래스를 대신 사용하기도 한다. 이렇게 하면 프로토콜과는 달리 저장된 프로퍼티를 가질 수 있다.

베이스 클래스를 사용하더라도 이를 인스턴스로 생성하진 않는다. 서브클래스 되는 목적으로만 사용된다. 다른 언어에서는 abstract 클래스가 이에 해당한다.

```swift
protocol TrafficLightState: AnyObject {
  var delay: TimeInterval { get }
  func apply(to context: TrafficLight)
}
```

### 콘크리트 상태들 (Concrete states)

상태 프로토콜 또는 베이크 클래스를 준수한다. 컨텍스트는 이런 상태값을 갖고 있지만 명확한 타입으로 지정하지는 않는다. (아래 예시 참고)

```swift
var state: StateProtocol = ConcreteState1() // 🆗

var state: ConcreteState1 = ConcreteState1() // ❌
```

컨텍스트는 폴리모피즘이라는 것을 사용하여 행동을 바꾼다. 콘텍스트 상태는 컨텍스트가 해야할 액션을 정의합니다. 새로운 행동이 필요하면 새로운 콘크리트 상태를 정의 하면 됩니다.

```swift
class GreenTrafficLightState {
  let delay: TimInterval = 1.0
  
  func apply(to context: TrafficLight) {
    queue.ayncAfter(deadline: DispatchTime.now() + dely) {
      context.transition(to: RedTrafficLightState())
    }
  }
}

class RedTrafficLightState {
  let delay: TimInterval = 2.0
  
  func apply(to context: TrafficLight) {
  
  }
}
```

그럼 `context.currentState` 값은 어디서 바꿔야 하는가? 놀랍게도 스테이트 패턴은 그거에 대한 해답을 제공하진 않는다. 이것은 이 패턴에 대한 장점인 동시에 단점이기도 하다.

## 통화

caller -> 전화걸고 ->  Offer -> connect
callee -> 전화를 수락하거나 거절하기 -> Answer -> connect

```swift

class DirectCall { // context
    var currentState: CallState
    
    func changeState(to newstate: CallState) {
    }
    
    func end() {
        // 메
    }
}

protocol CallState {
    func makeCall(context: DirectCall)
    func cancel(context: DirectCall)
    func accept(context: DirectCall)
    func decline(context: DirectCall)
    func end(context: DirectCall)
}

class OngoingCallState {
    func cancel(context: DirectCall) {
        context.end()
    }
}

class OfferState

```


## 요점

- 스테이트 패턴은 런타임 동안 객체가 행동을 바꿀 수 있도록 한다.
- 컨텍스트, 상태 프로토콜, 콘크리트 상태들 로 구성된다.
- **컨텍스트**는 `currentState` 를 갖고 있는 객체고 **상태 프로토콜**은 필요한 메소드와 프로퍼티를 정의힌다. 그리고 **콘크리트 상태**들은 상태 프로토콜을 준수하고 런타임동안 바뀌는 실제 행동을 구현한다.
- 스테이트 패턴은 상태들 간의 변화에 대한 로직을 어디에 둬야할지 말해주진 않는다. 컨텍스트, 콘크리트 상태 어느 쪽이던 유연한 디자인에 적합하게 선택하면 된다.
