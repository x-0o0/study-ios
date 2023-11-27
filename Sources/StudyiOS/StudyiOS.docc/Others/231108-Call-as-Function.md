# call-as-function 알아보기

객체의 메서드를 호출하지 않고 객체 자체를 함수로 사용하는 방법.

## 1. 서론

Pointfree.co 의 [ComposableArchitecure](https://github.com/pointfreeco/swift-composable-architecture) 라이브러리에서 제공하는 `Send` 타입이 `CallAsFunction` 을 구현하고 있어 다음과 같이 쓸 수 있다고 안내되어 있습니다.<sup>[1](#footnote_1)</sup>

```swift
return .run { send in
    send(.started)
    defer { send(.finished) }
    for await event in self.events {
        send(.event(event))
    }
}
```

이 때 `send`는 함수처럼 쓰고 있지만 타입을 보면 함수 타입이 아닌 `Send` 타입의 객체입니다.
객체를 함수처럼 쓸 수 있는 이유는 `Send`가 call-as-function 을 구현하고 있기 때문입니다.

이 문서에서 call-as-function이 무엇인지 다뤄보겠습니다.

## 2. 알아보기

### 2.1. 사용 방법

다음 내용은 Swift.org 의 내용을 번역하였습니다.<sup>[2](#footnote_2)</sup>

call-as-function 메서드 이름은 `callAsFunction()` 또는 `callAsFunction(` 로 시작하여 인자가 있는 형태를 사용합니다.

예를 들어, `callAsFunction(_:_:)` 과 `callAsFunction(something:)` 둘 다 call-as-function 메서드 이름으로 사용가능 합니다.

다음 함수 호출은 모두 동일합니다.
```swift
struct CallableStruct {
    var value: Int
    func callAsFunction(_ number: Int, scale: Int) {
        print(scale * (number + value))
    }
}

let callable = CallableStruct(value: 100)
callable(4, scale: 2)
callable.callAsFunction(4, scale: 2)
// 둘 다 208 를 출력합니다.
```

call-as-function 메서드는 다음 두 가지 사이에서 적절한 절충안을 만들어 균형을 유지시킵니다.
- 타입 시스템에 얼마나 많은 정보를 인코딩할 지
- 런타임 동안 얼마나 많은 동적 행동이 가능할 지

### 2.2. 왜 사용하는가?

객체를 함수처럼 호출할 수 있는 것은 상태를 가진 계산 프로그램, 파서 또는 컴퓨팅 객체를 표현하고자 할 때 유용함을 제공합니다. 이는 복잡한 수학 연산 및 머신 러닝에서 특히 흔하며 특정 객체가 일부 상태를 유지하고 하나의 메서드만을 구현하는 경우가 있습니다.

donnywals.com 에서 다음과 같은 예시를 통해 설명하고 있습니다. 실제 파이썬에서 자주 볼 수 있는 패턴 중 하나인 함수에 렌더링 객체를 전달하는 패턴을 Swift 언어로 보여주며 설명하고 있습니다.

```swift
protocol Route {
    associatedtype Output
}

func registerHandler<R: Route>(_ route: R, _ handler: (R) -> R.Output) {
    return renderer(route)
}
```

`registerHandler` 는 `Route` 프로토콜을 준수하는 객체를 전달받고, 이를 `handler` 에서 인수로 전달합니다. 이 함수를 다음과 같이 호출할 수 있습니다.

```swift
registerHandler(homeRoute) { route in
    /* 여기서 아웃풋을 생성하기 위해 수많은 작업을 수행해야 함 */

    return output
}
```
간단한 `renderer`의 경우 이 방식은 문제가 없지만, 파이썬 패턴을 현재 스위프트로 보여주고 있음을 고려하여, 파이썬의 맥락에서, 이러한 코드는 웹서버에서 실행됩니다.
`route`는 사용자가 요청한 웹페이지의 URL(또는 경로)와 동등합니다. `registerHandler`의 클로저는 사용자에 의해 연결된 라우트가 요청될 때마다 호출됩니다. 처리해야할 작업이 많은 경우 간단한 클로저만으로 충분치 않을 수 있습니다. **`route`를 처리하는데 사용되는 객체에는 데이터베이스 연결, 캐싱, 인증 등등 여러 기능이 있어야 합니다.**
이 모든 것을 클로저에 포함시키는 것은 좋은 아이디어가 아닙니다. 클로저에 포함시키는 것 대신 `route`를 처리하는 하나의 객체를 사용하는 것이 좋습니다. 이 때 call-as-function을 사용하는 것이 매우 유용합니다.

```swift
struct HomeHandler {
    let database: Database
    let authenticator: Authenticator

    // ...

    func callAsFunction<R: Route>(_ route: R) -> R.Output {
        /* 여기서 아웃풋을 생성하기 위해 수많은 작업을 수행해야 함 */

        return output
    }
}

let homeHander = HomeHandler(
    database: database, 
    authenticator: authenticator
)

registerHandler(for: homeRoute, homeHandler)
```

이렇게 객체를 클로저를 받는 함수에 그대로 전달할 수 있습니다. `homeHandler.callAsFunction` 과 같이 메서드를 직접 넘길 수도 있으나 call-as-function 을 사용하기 때문에 어떤 메서드를 호출해야하는지 알 필요가 없어 사용도 쉽고, 읽기도 쉽습니다.

## 3. 결론

서론에서 보았던 Pointfree.co 의 ComposableArchitecture 의 예시코드를 다시 살펴 보겠습니다.

```swift
return .run { send in
    send(.started)
    defer { send(.finished) }
    for await event in self.events {
        send(.event(event))
    }
}
```

이때 클로져가 전달하는 실행인자가 함수가 아니라 `Send` 라는 객체입니다. 하지만 `Send` 가 `callAsFunction` 을 구현하고 있기 때문에 아래와 같이 함수처럼 사용할 수 있습니다.

```swift
send(.started)
```

따라서, call-as-function 은 객체의 메서드를 호출하지 않고 객체 자체를 함수로 사용하는 방법을 제공합니다.

## 4. 참조

### 4.1 용어 정리

| 국문 | 영문 | 설명 |
| --- | --- | --- |
| 인자, 매개변수 | Parameter | 함수를 정의할 때 필요한 변수 이름, 함수의 전달되는 값을 넘겨받을 때 쓰이는 변수 |
| 인수, 실행인자 | Argument | 함수를 호출할 때 실제로 넘어가는 변수 값 |

### 4.2 참고 문헌

<a name="footnote_1">1<a>: [https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/send](https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/send)

<a name="footnote_2">2<a>: [CallsAsFunction, swift.org](https://docs.swift.org/swift-book/ReferenceManual/Declarations.html#ID622)
