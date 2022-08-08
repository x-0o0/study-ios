# State 와 데이터 플로우 (1)

## SwiftUI 의 긍정적인 부분

- 선언적이다: UI를 구현할 필요 없이 선언만 하면 된다.
- 함수형이다: 같은 상태면 UI 결과는 늘 동일하다. 즉 UI는 상태에 따른 함수이다.
- 반응형이다: 상태가 변하면 스유는 자동으로 UI를 업데이트 한다.

## State

struct로 상태를 선언한다면?
- value type이라 mutating error 발생
  - body는 자신이 포함된 sturct를 mutate 할 수 없음

class 로 상태를 선언한다면?
- 상태값은 변하나 뷰 업데이트가 안됨

### PropertyWrapper

struct로 상태를 선언하면 문제점은 mutating error 발생이다.
body는 자신이 포함된 sturct를 mutate 할 수 없다.
해결방법: mutating 프로퍼티를 ref type(class)으로 감싸면 됨 (propertyWrapper)

```swift
class State<T> {
  var wrappedValue: T
  
  init(initialValue value: T) {
    self.wrappedValue = initialValue
  }
}
```

```swift
struct ScoreView: View {
  var _numberOfAnswer = State<Int>(initialValue: 0)
  
  var body: some View {
    Button(action: answer) {
      HStack {
        Text(...)
        
        Spacer()
    }
  }
  
  func answer() {
    self._numberOfAnswer.wrappedValue += 1
  }
}
```

다른 인스턴스를 참조하는 방식으로 이제 State 안의 value를 변경할 수 있다.
뷰도 업데이트 된다.

#### 애플의 얘기도 들어보자

[ Developer Doc - State](https://developer.apple.com/documentation/swiftui/state)

> **Apple왈** property wrapper 타입으로 SwiftUI에 의해 관리되는 값을 읽고 쓸 수 있다.
> 
> SwiftUI는 state로 선언된 모든 프로퍼티의 저장소를 관리한다. 
> 상태값이 변하면 뷰는 기존 모양을 무효화하고 body를 재계산한다.
> 뷰를 위한 *Single Source of Truth* 로 state를 사용하면 된다.

#### 사용

```swift
@State var numberOfAnswered = 0

...
  
func answer() {
  numberOfAnswer += 1
}
```
컴파일러가 알아서 wrappedValue 업데이트로 translate

**결론**: state로 선언된 프로퍼티만 뷰에 영향을 줄 수 있다.

### 바인딩 (`$`)

상태 변수는 값이 변경될때마다 UI 업데이트를 트리거하는 용도로만 쓰이진 않는다.

UIKit에서는 값 변화에 대한 알림을 받으려면 delegate를 쓰거나 textfield에 edit 이벤트를 subscribe 해야했다.

그리고 수동으로 UI를 업데이트 해야했다. 예를 들어 버튼을 활성/비활성하거나 에러를 보여주거나.

스유에서는 이 과정이 보다 쉽다. 스유에서는 컴포넌트들이 데이터를 *소유* 하지 않고 데이터를 *참조*를 하기 때문에 모델에 변경이 있으면 자동으로 UI를 업데이트 하게 된다.

이 때 Binding이라는 걸 사용한다.

> **바인딩**: 바인딩은 데이터를 저장하는 *프로퍼티*와 데이터를 *변경*하고 보여주는 *뷰* 간의 양방향 커넥션이다.
> 바인딩은 직접 데이터를 저장하지 않고 source of truth를 준수하는 저장된 프로퍼티를 연결한다.

```swift
@State var name: String = ""

...

TextField("Type your name...", text: $name)

Text(name)
```

```swift
// 데이터가 아닌 참조를 전달
@Binding var numberOfAnswer: Int
```

```swift
ScoreView(
  numberOfQuestions: 5,
  numberOfAnswered: $numberOfAnswered
)
// ScoreView로 바인딩 전달 -> 동일 데이터에 접근 가능
```

## 요점

- 뷰가 소유하는 데이터로 프로퍼티를 생성할때는 `@State` 를 사용할 것. 이는 프로퍼티 값이 변하면 UI를 자동으로 재 렌더링 할 수 있도록 한다.
- `@Binding` 을 사용하여 상태 프로퍼티 같은 프로퍼티를 생성할 수 있고 데이터는 다른 곳(상위 뷰의 state 프로퍼티 또는 observable object)에서 저장/소유 된다.
```
