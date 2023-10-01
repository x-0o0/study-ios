# Long Press 적용시 스크롤이 안되는 이슈

## 해결방법 미리보기

```swift
MyView()
  .onTapGesture { } // 👈 추가 (empty closure)
  .onLongPressGesture { ... }
```

## 이슈

ChatUI 오픈소스 프로젝트에서 메세지뷰에서 터치할 경우 메세지 리스트 스크롤이 안된다는 이슈가 들어왔다.
`onLongPressGesture` 가 적용된 뷰의 경우 다른 제스쳐에 대해 정의하지 않으면 동작을 하지 않는 것 같다.

## 스택오버플로우

[링크](https://stackoverflow.com/questions/59440283/longpress-and-list-scrolling?rq=1)
> I asked this on the Apple Developer Forum as well and got a solution for the problem. If the view defines an onTapGesture handler before onLongPressGesture, the list will be scrollable while supporting long press on the individual items.
> The onTapGesture handler can be empty as long as it is declared first.

