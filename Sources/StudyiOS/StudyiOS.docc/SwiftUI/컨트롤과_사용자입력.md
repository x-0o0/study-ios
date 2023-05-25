# 컨트롤과 사용자입력

## `TextField`

### `@State` 

데이터 값이 변하면 뷰를 업데이트를 해야하는 경우, 즉 뷰에 영향을 주는 프로퍼티는 `@State` 속성을 사용하여 선언한다.

`TextField` 는 `UITextField` 에 상응하는 뷰이다.

입력이 들어올때마다 텍스트 값이 변경되고 이 변경이 UI에 반영되어야 하므로 `TextField` 에 들어가는 텍스트는 반드시 `@State` 로 선언한다.

```swift
@State private var name: String = ""

TextField("placeholder", text: $name)
```
