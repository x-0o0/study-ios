# 테스팅과 디버깅

## 테스트 개요
테스트는 크게 아래의 3가지로 구분 됩니다.

**1. 유닛 테스트 (Unit test)**

하나의 함수에 대해 입력에 따른 결과를 체크하는 테스트입니다. 오직 유닛테스트 하나당 하나의 함수/코드만을 테스트 해야하며
기본적으로 밀리 세컨드 단위로 실행되어야 합니다.

**2. 통합 테스트 (Integration test)**

서로 다른 코드들이 서로 잘 맞물려 동작하는지를 체크하는 테스트 입니다. 외부 API 에 대한 테스트도 포함합니다.

**3. UI 테스트 (UI test)**

유저가 보게 될 앱 화면에 대한 테스트 입니다. 유저 인터랙션과 그에 따른 유저 인터페이스 동작을 체크합니다.

## SwiftUI 앱 디버깅

```swift
Button(action: calculate} {
    Text("버튼입니다")
}

func calculate() {
    
}
```

### 브레이크포인트

```swift
Button(action: calculate} {
🏷   Text("버튼입니다")
}

func calculate() {
🏷  
}
```

이렇게 브레이크포인트를 잡고 앱을 실행합니다. Xcode13 이전 버전에서는 preview에서 디버그 모드를 실행할 수 있었지만 Xcode13 부터는 preview에서 불가능합니다... 퇴보하는 SwiftUI...🙄

👇🏼 프리뷰에서 디버깅하기 (Xcode13 미만만 가능합니다)

<img width="30%" alt="Screen Shot 2022-05-02 at 5 32 41 PM" src="https://user-images.githubusercontent.com/53814741/166206807-f8652e23-1534-4907-8d0d-ae79a971075e.png">

암튼 Xcode13 에서는 앱을 실행해야 합니다.

calculate 함수의 브레이크포인트에서 콘솔에 아래의 명령을 실행하면 다음의 결과를 얻을 수 있습니다,

```swift
(11db) po _memory
🔽 State<Double>
   - _value : 0.0
   🔽 _location : Optional<AnyLocation<Double>>
      🔽 some : <StoredLocation<Double>: 0x600003fbf1e0>
```

### `_printChanges()` 사용하기

```swift
var body: some View {
  let _ = Self._printChanges()
  ...
}
```
iOS 15 이상부터 위와 같이 `_printChanges()` 함수를 사용하여 `body` 안의 상황을 모니터링 할 수 있다.

해당 함수는 뷰와 다시 그려진 프로퍼티의 이름을 출력한다.

## UI 테스트

UI tests 타겟을 추가합니다. 추가 방법을 모르신다면 아래 `details`를 참고하세요.

<details>
  <img width="469" alt="Screen Shot 2022-05-07 at 4 48 24 PM" src="https://user-images.githubusercontent.com/53814741/167244585-11e18a5f-e3bc-4f81-ac28-cd39a3453e62.png">
</details>

테스트 코드에 기본 내용은 아래의 `details` 를 참고하세요.

<details>
  
  ```swift
  override func setUpWithError() throws
  ```
  각각의 테스트 함수가 실행되기 전마다 호출됩니다.
  
  ```swift
  override func tearDownWithError() throws
  ```
  각각의 테스트 함수가 끝날 때마다 호출됩니다.
  
  ```swift
  continueAfterFailure = false
  ```
  테스트 도중 실패 케이스가 발생하면 즉각 멈춥니다.
</details>

각각의 UI 테스트는 막 시작한 앱의 상태로 시작합니다. 하지만 이것이 각각의 실행때마다 앱 상태가 초기화된다는 의미는 아닙니다. 
`setUpWithError()`, `tearDownWithError()` 를 통해 테스트 동안 발생한 변화를 원래대로 돌려놓도록 하거나 필요한 사전 세팅을 합니다.

```swift
func testPressMemoryPlusAtAppStartShowZeroInDisplay() {
  let app = XCUIApplication()
  app.launch()
}
```

테스트 실행 후 왼쪽 하단에 빨간 동그라미를 클릭하면 시뮬레이터에 앱이 실행되는데, 이때 버튼이나 어떤 액션을 취하면 테스트 코드안에 기록됩니다.

```swift
let app = XCUIApplication()
app.button["M+"].tap()
app.windows
  .children(matching: .other).element
  .children(matching: .other).element
  .children(matching: .other).element
  .children(matching: .other).element
  .tap()
```

이걸 아래와 같이 수정합니다.

```swift
let app = XCUIApplication()
app.launch()

let memoryButton = app.buttons["M+"]
memoryButton.tap()
```

### 유저 인터랙션 읽기

유저 인터랙션에 의한 변화를 감지하고 싶은 뷰에 아래의 속성을 추가합니다.
```swift
  .accessibilityIdentifier("display")  //raywenderlich 에는 `accessibility(identifier:)` 로 되어있음
```

다시 테스트로 돌아와서 아래와 같이 코드를 추가합니다.

```swift
let app = XCUIApplication()
app.launch()

let memoryButton = app.buttons["M+"]
memoryButton.tap()  // display 의 숫자 리셋

let display = app.staticTexts["display"]
let displayText = display.label
XCTAssert(displayText == "0")
```

```swift
let display = app.staticTexts["display"]
```
`XCUIElement` 를 생성합니다.

```swift
let displayText = display.label
```
라벨의 텍스트를 테스트하기 위해 `label`에 접근합니다.

```swift
XCTAssert(displayText == "0")
```

### 복잡한 테스트 짜기

```swift
func testAddingTwoDigits() {
  let app = XCUIApplication()
  app.launch()
  
  let threeButton = app.buttons["3"]
  threeButton.tap()
  
  let addButton = app.buttons["+"]
  addButton.tap()
  
  let fiveButton = app.buttons["5"]
  fiveButton.tap()
  
  let equalButton = app.buttons["="]
  equalButton.tap()
  
  // 3 + 5 =
  
  let display = app.staticTexts["display"]
  let displayText = display.label
  XCTAssert(displayText == "8.0")
}
```

### 유저 인터랙션 시뮬레이션

```swift]
var body: some View {
// 1. 제스쳐 추가
  let memorySwipe = DragGesture(minimumDistance: 20)
    .onEnded { _ in
      memory = 0.0
    }
    
  HStack { 
    Spacer()
    
    Text(...)
      .accessibilityIdentifier("memoryDisplay") // id 추가
      .gesture(memorySwipe)  // 제스쳐 적용
    
    Text(...)
  }
}
```

```swift
func testSwipeToClearMemory() {
  let app = XCUIApplication()
  app.launch()
  
  app.buttons["3"].tap()
  app.buttons["5"].tap()
  app.buttons["M+"].tap()
  
  let memoryDisplay = app.staticTexts["memoryDisplay"]
  // 1. `exists` 프로퍼티는 `XCUIElement` 값이 존재할 때 `true` 가 됩니다. memory display 가 눈에 보이지 않으면 assert는 실패합니다.
  XCTAssert(memoryDisplay.exists)
  // 2. `swipeLeft()` 는 호출된 element 에 스와이프 액션을 제공합니다. 이외에도 `swipeRight()`, `swipeUp()`, swipeDown()` 도 있습니다.
  memoryDisplay.swipeLeft()
  // 3.제스쳐 이후 `memory` 는 `0` 으로 세팅되고 액션이 메모리 화면을 숨기기 때문에 `exists` 값은 `false` 가 되어야 합니다.
  XCTAssertFalse(memoryDisplay.exists)
}
```

이 외에도 추가적인 공통 속성들이 있습니다.

- `.isHittable`: element 가 존재하고 유저가 탭할 수 있는 경우를 말합니다. element가 존재하더라도 화면 밖에 없으면 탭할 수 없기 때문에 `ishittable` 값이 `false` 가 됩니다.
- `.typeText()`: 호출한 컨트롤에 텍스트를 입력하는 것처럼 동작합니다.
- `.press(forDuration:)`: 특정 시간 동안 한 손가락 터치를 수행할 수 있도록 합니다.
- `.press(forDuration:thenDragTo:)` `swipe` 메소드는 제스쳐 속도에 대해 보장해주지 않습니다. 이 함수는 더 정밀한 드래그 액션을 할 수 있도록 합니다.
- `.waitForExistence()`: element 가 스크린에 보이지 않을 때 일시정지합니다.

> 더 많은 속성이 궁금하시다면 [링크](https://developer.apple.com/documentation/xctest/xcuielement) 를 클릭해주세요.

## 요약
- - -

- 평소처럼 브레이크 포인트 잡고 디버깅할 수 있다.
- 뷰 요소가 고정된 텍스트 값을 갖지 않는 경우 `accessiblityIdenitifier(_:)` 을 추가하여 테스트 할 수 있다.
- `Self._printChanges()` 를 사용하여 뷰가 다시 그려질 때의 변화를 모니터링 할 수 있다.

