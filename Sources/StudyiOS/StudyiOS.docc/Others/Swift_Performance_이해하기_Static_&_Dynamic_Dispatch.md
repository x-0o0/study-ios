# Swift Performance 이해하기 (Static & Dynamic Dispatch)

## 개요

> **참고** [Understanding Swift Performance | WWDC16](https://developer.apple.com/videos/play/wwdc2016/416/)

## Method Dispatch

- static dispatch
  - **컴파일 타임**에 어떤 구현이 실행될 건지 알 수 있음
  - 컴파일러가 최적화를 할 수 있으므로 속도, 성능이 더 좋다
  - `value type` (오버라이딩이 불가하기 때문) - `struct`, `enum`
  - `final`
- dynamic dispatch: **런타임**에 어떤 구현이 실행될 건지 알 수 있음
  - `class`
  - `protocol` (프로토콜 고유의 vtable를 가짐 - Witness Table)

> **vtable 이란**
> 함수들의 포인터들을 배열로 저장하고 있는 Dynamic dispatch를 위한 가상 디스패치 테이블(Virtual Dispatch Table)
>
> 아래에 더 자세한 설명이 있음

### Dynamic Dispatch 줄이기

> [Reducing Dynamic Dispatch | Swift Docs](https://github.com/apple/swift/blob/main/docs/OptimizationTips.rst#reducing-dynamic-dispatch) 

스위프트는 옵씨와는 달리 기본적으로 매우 다이나믹한 언어이다. 이는 개발자로 하여금 다이나믹함을 줄여 런타임 퍼포먼스를 향상 시킬 수 있다.

**방법1: `final` 키워드 쓰기**

```swift
final class A { 
    // 이 클래스에 있는 선언들은 전부 오버라이드 불가
    var array1: [Int]
    func doSomething() { ... }
}

class B {
    // 오버라이드 불가
    final var array1: [Int]
    // 오버라이드 가능
    var array2: [Int]
}

func useA(_ a: A) {
    // 다이나믹 디스패치를 거치지 않고 바로 A의 프로퍼티 에 접근 가능
    a.array1[i] = ...
    // 가상 디스패치를 거치지 않고 바로 A의 메소드에 접근 가능
    a.doSomething()
}

func useB(_ b: B) {
    // 다이나믹 디스패치 거치지 않고 프로퍼티에 바로 접근 가능
    b.arrayi[i] = ...
    // 다이나믹 디스패치를 거쳐서 프로퍼티 접근
    b.array2[i] = ...
}
```

**방법2: 파일 외부에서 접근할 필요 없는 경우, `private` 또는 `fileprivate` 를 사용하여 선언하기**

이 경우, 오버라이딩이 불가하다는 것을 컴파일러가 알기 때문에 자동으로 `final` 키워드로 추론할 수 있습니다.

```swift
private class C {
    func doSomething() { ... }
}

class D {
    fileprivate var myPrivateVar: Int
}

func useC(_ c: C) {
    c.doSomething()
    // 클래스 c가 선언된 파일에 서브클래스가 없기 때문에
    // 컴파일러가 메소드에 대한 가상 콜을 제거하여
    // 해당 메소드를 직접 호출 가능.
}

func useD(_ d: D) -> Int {
    // 바로 접근 가능
    return d.myPrivateVar
}
```

**방법3: WMO이 활성화되어 있는 경우, 모듈 밖에서 접근할 필요 없는 건 `internal` 로 선언**

WMO는 컴파일러가 한번에 모듈의 모든 소스들을 컴파일 하도록 합니다. internal로 선언된 것들은 모듈 밖에서 보이지 않기 때문에,
옵티마이저(최적화도구)가 자동으로 잠재적으로 오버라이딩 할 수 있는 모든 선언들을 탐지하여 `final`로 간주합니다.

> **WMO 란?**
>
> Whole Module Optimization 의 약자.
> 스위프트는 기본적으로 각각의 파일을 따로따로 컴파일 합니다.이는 Xcode 가 여러개의 파일을 나란히 컴파일 할 수 있도록 하여 컴파일 속도를 빠르게 합니다.
> 그러나 각각의 파일을 따로따로 컴파일 하는 것은 몇몇의 컴파일러 최적화가 이뤄지지 않습니다.
>
> 스위프트는 마치 하나의 파일에 전부 들어가 있는 것처럼 프로그램 전체를 한번에 컴파일 할 수 있고, 마치 하나의 컴파일 단위만 있는 것처럼 프로그램을 최적화 할 수 있습니다.
> 이 모드는 `swiftc` 커맨드라인에 `-whole-module-optimization` 플래그를(또는 Xcode 빌드세팅의 `Whole Module Optimization`) 사용하여 활성화할 수 있습니다.
> 이 모드는 컴파일 시간이 길어지지만, 돌아가는 속도는 빨라질 수 있습니다.



### Virtual Dispatch

스위프트는 클래스마다 vtable(virtual dispatch table) 이라는 것을 유지합니다. vtable은 함수 포인터들의 배열로 표현되며,
서브 클래스가 메소드를 호출할 때, 이 배열을 참조하여 실제 호출할 함수를 결정합니다(indirect call)
이 과정들은 전부 런타임에 결정되기 때문에 Static dispatch 에 비해 추가적인 연산이 필요할 수 밖에 없고, 컴파일러가 최적화하기에 유리하지 않음.
