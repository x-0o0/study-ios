# task vs onAppear

## 결론부터 보기

task 에서는 비동기 처리에 적합하고, 뷰가 사라지면 스유가 알아서 취소해준다. onAppear는 자동으로 취소해주지 않으므로, onDisappear에서 취소 처리를 따로 해줘야한다.
뷰가 뜰 때 처리하고자 하는 코드가 자동으로 취소를 요구하는 비동기 코드면 task를 아니면 onAppear.

## 개요

### task(priority:_:)

> **[애플문서](https://developer.apple.com/documentation/swiftui/view/task(priority:_:)#:~:text=If%20the%20task%20doesn%E2%80%99t%20finish%20before%20SwiftUI%20removes%20the%20view%20or%20the%20view%20changes%20identity%2C%20SwiftUI%20cancels%20the%20task.):**
> If the task doesn’t finish before SwiftUI removes the view or the view changes identity, SwiftUI cancels the task.

뷰가 사라지거나 identity 가 변경되려고 할 때 task 가 안 끝나있으면 스유가 알아서 취소해줌

### onAppear
> **[애플문서](https://developer.apple.com/documentation/swiftui/view/onappear(perform:)#:~:text=The%20exact%20moment%20that%20SwiftUI%20calls%20this%20method%20depends%20on%20the%20specific%20view%20type%20that%20you%20apply%20it%20to%2C%20but%20the%20action%20closure%20completes%20before%20the%20first%20rendered%20frame%20appears.)**
> The exact moment that SwiftUI calls this method depends on the specific view type that you apply it to, but the action closure completes before the first rendered frame appears.

이 메소드를 적용하는 뷰에 따라 정확한 호출 시점은 다르지만 클로져는 렌더링된 프레임이 뜨기전에 종료됩니다.
