# NavigationLink의 Disclosure Indicator 숨기기

## 해결방법

List 의 Row를 나타내는 `MyRow` 가 있다고 가정할 때 `ZStack` 으로 `NavigationLink` 와 분리 시키고 `NavigationLink` 를 `opacity` 를 `0`으로 적용합니다.

```swift
// Row
ZStack {
    NavigationLink {
        // destination
    } label: {
        EmptyView()
    }
    .opacity(0)
    
    MyRow(item: item)
}
```
