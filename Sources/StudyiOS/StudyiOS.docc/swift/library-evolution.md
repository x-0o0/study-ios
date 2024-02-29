# 라이브러리 에볼루션 (Library Evolution)

## 개요

Swift 5.1 이상부터 다음 두가지 기능이 제공됩니다.
1. 모듈 안정성 (Module Stability)
   - 서로 다른 버전의 컴파일러에서 빌드된 스위프트 모듈들이 하나의 앱에서 함께 쓰일 수 있게 해주는 기능
2. 라이브러리 에볼루션 지원 (Library Evolution Support, 이하 "에볼루션")
   - 바이너리 프레임워크 개발자들이 바이너리가 이전 버전과 호환가능하게 유지하면서 프레임워크의 API 에 추가적인 변화를 만들도록 허용
  
모듈 안정성 기능은 라이브러리 에볼루션 지원 기능을 필요로 합니다.

배포를 위해 바이너리 프레임워크를 빌드하는 경우 일반적으로 이 두가지 기능을 활성화하게 될 것입니다.
> **NOTE**
>
> 바이너리 프레임워크는 동적(dynamic) 라이브러리라고 이해하면 다음 내용들을 이해하기 수월합니다.
> 스위프트 패키지는 기본적으로 정적(static) 라이브러리 입니다.

## 라이브러리 에볼루션 지원 활성화 시키는 경우
기본적으로 에볼루션 기능은 꺼져 있습니다.
빌드와 배포를 항상 하는 프레임워크의 경우 (예: 스위프트 패키지, 또는 앱 내부의 바이너리 프레임워크) 에볼루션 기능을 쓰지 말아야합니다.

에볼루션 기능은 오직 클라이언트 와는 별도로 프레임워크가 빌드되고 업데이트 되는 경우에만 사용해야합니다.
예를 들어 옛 버전의 프레임워크로 빌드한 앱이 다시 컴파일 하지 않고 새버전의 프레임워크에서 돌아갈 수 있는 경우가 이에 해당합니다.

에볼루션 기능을 켜는 것은 프레임워크의 퍼포먼스 형식을 바꾸고 소스 비호환 언어 변화를 도입합니다.

에볼루션 없이 빌드한 프레임워크는 바이너리 호환성을 보장해주지 않기 때문에 에볼루션 기능을 키게 되면 바이너리가 호환되지 않는 변경이 됩니다.

> **INFO**
>
> 예전에 다음과 같은 의존성을 가진 프로젝트에서 SDK 내 수정된 인터페이스에 대한 signature를 못 찾아 크래시가 난적이 있다.
>
> 앱 --의존--> Sendbird UIKit --의존---> Chat SDK
>
> 이때 Sendbird UIKit은 구버전을 그대로 사용하고, Chat SDK 만 업데이트 했는데,
> Sendbird UIKit은 구버전의 Chat SDK의 interface를 사용하고 있었지만, 새 버전의 Chat SDK 에서 해당 인터페이스가 수정되면서 시그니처가 사라져었다.

## 라이브러리 에볼루션 지원 활성화 하기
### Xcode
Xcode 를 사용해서 개발하고 있다면 프레임워크 타겟에서 `BUILD_LIBRARY_FOR_DISTRIBUTION` 빌드 세팅을 설정합니다. 이 설정은 모듈 안정성과 라이브러리 에볼루션 기능 둘 다 켜줍니다.
> **IMPORTANT**
> 
> 반드시 Debug, Release 모드 둘 다 이 설정을 사용해야합니다.

> **NOTE**
> [Binary frameworks in Swift | WWDC19](https://developer.apple.com/wwdc19/416) 영상 참고

### 컴파일러에서 직접 호출
`swiftc` 를 직접 사용하고 있다면, 커맨드 라인이나 빌드 시스템 둘 중 한 곳에서 `-enable-library-evolution` 과 `-emit-module-interface` 플래그를 전달하면 됩니다.
```bash
$ swiftc Tack.swift Barn.swift Hay.swift \
  -module-name Horse \
  -emit-module -emit-library -emit-module-interace \
  -enable-library-evolution
```
위의 명령어를 돌리면, `Horse.swiftinterface` 라는 이름의 모듈 인터페이스 파일과 `libHorse.dylib`(macOS) 또는 `libHorse.so`(리눅스) 공유 라이브러리가 생성됩니다.
> **INFO**
>
> `dylib` 는 동적라이브러리(Dynamic Library) 를 의미합니다.

## 라이브러리 에볼루션 모델

라이브러리 에볼루션은 프레임워크에 특정 변경이 발생해도 바이너리 호환성을 깨지지 않게 합니다.
> **NOTE**
>
> 이와 같이 새버전이 구버전과 소스 호환성 & 바이너리 호환성 둘 다 유지하는 경우 이 프레임워크는 "탄력이 있다(resilient)" 라고 표현합니다.

### ABI 퍼블릭 선언
ABI 퍼블릭 선언은 다른 스위프트 모듈에서 참조할 수 있는 선언을 말합니다.
- 모든 `public` 은 ABI-public
- `@usableFromInline` 속성을 사용해 선언하는 경우 public이 아니더라도 ABI-public. @inline 코드에서 참조할 수 있음을 의미합니다.

### ABI 프라이빗
ABI-public 이 아니라고 명시하는 방식으로 선언하는 것을 ABI 프라이빗(ABI-private)이라고 합니다.
`@usableFromInline` 속성이 없는 `private`, `fileprivate`, `internal` 이 이에 해당합니다.

### @frozen
`@frozen` 속성도 라이브러리 에볼루션과 관련이 있습니다. 이 속성은 ABI 퍼블릭 struct 나 enum의 바이너리 인터페이스를 변경 시켜서 더 자세한 구현 내용을 노출 시킵니다.

### 탄력적 변경(Resilient changes)의 예시
- ABI 프라이빗 선언은 추가/제거/변경이 맘대로 가능한게 일반적인 원칙. 명시적으로 ABI 퍼블릭으로 선언된 것만 프레임워크의 바이너리 인터페이스에 포함됨.
- 소스 파일 내의 상단(top-level) 선언부들은 재정렬될 수 있고, 동일 프레임워크 내에서 다른 소스 파일로 이동할 수 있음. 타입 내 멤버들(프로퍼티, 메소드) 또는 extension 은 재정렬될 수 있음. 단, `@frozen` 을 사용해 선언된 stored property와 struct, enum 내의 enum case 들에 한해서 예외 적용

예를 들어 아래 탑레벨 함수와 두개의 메소드는 바이너리 호환성을 깨뜨리지 않고 재정렬될 수 있습니다.
```swift
// top-level 함수
public func sum<T: Sequence>(_ seq: T) -> Int where T.Element == Int {
   return array.reduce(0, (+))
}

// 메소드
open class NetworkHandle {
   open func open() { }
   open func close() { }
}
```
대조적으로,`@frozen` enum 에서 두개의 `case`는 재정렬 되지 않습니다. 하지만 메소드는 재정렬이 될 수 있습니다. 그리고 메소드와 `case` 의 상대적인 순서는 변경될 수 있습니다.
```swift
@frzen public enum Shape {
   case rect(w: Int, h: Int)
   case circle(radius: Int)

   public func area() -> Int { ... }
   public func circumference() -> Int { ... }
}
```

- 소스파일의 top level 에 선언을 추가할 수 있습니다.
- class, struct, enum 타입에 `@frozen` 이 사용되지 않으면 멤버들을 추가할 수 있습니다. 만약 `@frozen` 을 사용하는 타입이면, stored 프로퍼티 나 enum case 들이 추가될 수 없습니다. 다른 종류의 멤버들은 제약없이 추가될 수 있습니다.
- 변경할 수 없는(immutable) 프로퍼티는 mutable 이 될 수 있습니다. 프로퍼티에 대한 바이너리 인터페이스는 접근자 기능(accessor functions) 의 집합이기 때문에 새로운 선언를 추가하는 것과 동등하게 mutability를 도입할 수 있습니다 (setter)

예시: computed property인 `fahrenheit`` 를 주목하자
```swift
public struct Temperature {
   public var celsius: Int
   public var fahrenheit: Int { (celsius * 9) / 5 + 32 }
}
```
새 버전에서 `fahrenheit` 에 `setter` 추가 가능
```swift
public struct Temperature {
   public var celsius: Int
   public var fahrenheit: Int {
      get { (celsius * 9) / 5 + 32 }
      set { celsius = ((newValue - 32) * 5) / 9}
   }
}
```
- 새로운 프로토콜 요구사항이 프로토콜의 extension에서 기본 구현 부를 제공하면 추가가 될 수 있다.
```swift
public protocol PointLike {
   var x: Int { get }
   var y: Int { get }
}
```
새 버전은 `z` 라는 새 프로토콜 요구사항을 추가할 수 있음. 단, 기본 구현부를 제공해야함.
```swift
public protocol PointLike {
   var x: Int { get }
   var y: Int { get }
   var z: Int { get }
}

extension PointLike {
   public var z: Int { 0 }
}
```
새 associated type 추가하는 것도 default를 명시해주면 바이너리 호환이 가능함.
```swift
public protocol PointLike {
   var x: Int { get }
   var y: Int { get }
   var z: Int { get }

   associatedtype Magnitude = Double

   var magnitude: Magnitude { get }
}
```



