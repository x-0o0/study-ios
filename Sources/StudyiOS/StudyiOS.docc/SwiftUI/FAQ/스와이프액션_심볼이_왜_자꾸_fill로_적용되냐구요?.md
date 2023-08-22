# 스와이프 액션의 심볼이 왜 fill 로만 적용될까

## 결론 미리보기

> **Apple 왈** "For labels or images that appear in swipe actions, SwiftUI automatically applies the *fill* symbol variant, as shown above."

스유는 자동으로 fill 심볼로 적용한다.

## 개요

```swift
.swipeAction(...)
```

```swift
func swipeActions<T>(
    edge: HorizontalEdge = .trailing,
    allowsFullSwipe: Bool = true,
    content: () -> T
) -> some View where T : View
```

스유에서 스와이프 액션은 정말 사용하기 쉽습니다. 하지만 사용하다보면 아래의 경우 두 케이스 전부 동일하게 fill 이미지가 적용되는 것을 확인할 수 있습니다.

```swift
.swipeActions(edge: .leading) {
    Button (action: bookmark) {
        Image(systemName: isBookmark ? "bookmark" : "bookmark.fill") // 둘 다 "bookmark.fill" 로 적용된다.
    }
}
```

그 이유는 스유는 자동으로 fill 심볼로 적용하기 때문이다. 애플이 그렇게 만들었다. 애플이 그런거다. 애플을 탓하자.

[관련 애플 개발 문서](https://developer.apple.com/documentation/SwiftUI/View/swipeActions(edge:allowsFullSwipe:content:))에서 아래와 같은 내용을 찾아볼 수 있다.

> For labels or images that appear in swipe actions, SwiftUI automatically applies the *fill* symbol variant, as shown above.
