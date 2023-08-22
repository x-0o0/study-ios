# Part 6. SwiftUI bindings + TCA

## 왜 공부하는가?

SwiftUI 에서 `State` 변수에 대해 `$text` 와 같이 바인딩해서 사용하던 것처럼, 
TCA 를 사용할 경우 `ObservedObject` 인 `viewStore` 를 가지고 뷰계산에 필요한 state 값을 뷰와 바인딩 해야하는 상황이 
필수로 존재한다. 꼭 알아야하는 내용이기 때문에 한번 빠르게(?) 알아보자!

> **정보** [공식 문서](https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/bindings)

## 개요

SwiftUI 에 있는 많은 API 들은 앱의 `state`와 `view` 간의 양방향 통신을 설정하기 위해 바인딩을 사용합니다. TCA 는 이러한 통신에 앱의 `store` 를 사용하여 바인딩을 생성하는 여러가지 도구들을 제공합니다.

## Ad hoc 바인딩
`store`와 함께 통신하는 바인딩을 생성하는 가장 간단한 도구는 바로 `binding(get:send:)` 입니다. 이 도구는 두개의 클로져를 다룹니다.
- 어떻게 `state`를 바인딩 값으로 변형할지 묘사하는 클로져
- 어떻게 바인딩값을 `store`에 전달하기 위해 `action`으로 변형할지 묘사하는 클로져
예를 들어, 사용자가 햅틱 피드백을 활성화 했는지를 체크하는 도메인을 갖는 `reducer`가 있다고 가정해보자. 우선, 다음과 같이 `state`에 `Bool` 프로퍼티를 정의할 수 있다.
```swift
struct Settings: ReducerProtocol {
    strcut State: Equatable {
        var isHapticFeedbackEnabled = true
    }
}
```
그 다음, 외부에서 이 `state`를 변형할 수 있도록 하기 위해(예를 들어 토글), 이에 상응하는 업데이트를 전달할 수 있는 `action`을 정의해야합니다.
```swift
struct Settings: ReducerProtocol {
    enum Action {
        case isHapticFeedbackEnabledChanged(Bool)
        // ...
    }
}
```
`reducer` 가 이 `action`을 다룰 때, 다음과 같이 `state` 를 업데이트 할 수 있습니다.
```swift
struct Settings: ReducerProtocol {
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        switch action {
        case let .isHapticFeedbackEnabledChanged(isEnabled):
            state.isHapticFeedbackEnabled = isEnabled
            return .none
        }
    }
}
```
마지막으로, 도메인으로부터 바인딩을 가져와서 토글기능이 TCA 기능과 소통하도록 할 수 있습니다.
```swift
struct SettingsView: View {
    let store: StoreOf<Settings>
    
    var body: some View {
        WithViewStore(self.store) { viewStore in
            Form {
                Toggle(
                    "Haptic feedback",
                    isOn: viewStore.binding(
                        get: \.isHapticFeedbackEnabled,
                        send: { .isHapticFeedbackEnabledChanged($0) }
                    )
                )
            }
        }
    }
}
```
## state, action, reducer 바인딩하기
ad hoc 바인딩을 가져오는 것은 번거로울 수 있는 수동적인 단계들이 많이 필요합니다. 특히, 많은 바인딩에 의해 돌아가는 많은 컨트롤들이 있는 화면의 경우엔 더욱 그렇습니다. 이러한 이유로, TCA 는 `reducer`의 도메인과 로직에 적용되어 바인딩과정을 더 쉽게하는 도구 컬렉션을 제공합니다.
예를 들어, 설정 화면은 다음과 같이 `state`를 모델링한다고 가정해봅시다.
```swift
struct Settings: ReducerProtocol {
    struct State: Equatable {
        var digest = Digest.daily
        var displayName = ""
        var enableNotifications = false
        var isLoading = false
        var protectMyPosts = false
        var sendEmailNotifications = false
        var sendMobileNotifications = false
    }
}
```
이 필드들 중 대부분은 뷰에 의해 수정될 수 있습니다. 그 말은 즉슨, TCA 에서는 각 필드들마다 `store` 에 전달될 수 있는 상응하는 `action` 들이 필요합니다. 전형적으로 이는 필드 별로 `case` 를 생성하여 `enum` 형태로 구성할 수 있습니다.
```swift
struct Settings: ReducerProtocol {
    enum Action {
        case digestChanged(Digest)
        case displayNameChanged(String)
        case enableNotificationsChanged(Bool)
        case protectMyPostsChanged(Bool)
        case sendEmailNotificationsChanged(Bool)
        case sendMobileNotificationsChanged(Bool)
    }
}
```
아직 끝난게 아닙니다. `reducer` 안에서 반드시 이 각각의 `action`들을 다뤄 각 필드의 `state` 를 간단히 교체하게 해야합니다.
```swift
struct Settings: ReducerProtocol {
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        switch action {
        case let digestChanged(digest):
            state.digest = digest
            return .none

        case let displayNameChanged(displayName):
            state.displayName = displayName
            return .none

        case let enableNotificationsChanged(isOn):
            state.enableNotifications = isOn
            return .none

        case let protectMyPostsChanged(isOn):
            state.protectMyPosts = isOn
            return .none

        case let sendEmailNotificationsChanged(isOn):
            state.sendEmailNotifications = isOn
            return .none

        case let sendMobileNotificationsChanged(isOn):
            state.sendMobileNotifications = isOn
            return .none
        }
    }
}
```
이는 충분히 단순하게 만들 수 있는 많은 boilerplate(재사용 가능한 코드조각) 들을 보여주고 있습니다. 운좋게도, 우리는 `BindingState`, `BindableAction` 그리고, `BindingReducer` 를 사용하여 이러한 boilerplate를 획기적으로 제거할 수 있습니다.
첫번째로, `state` 바인딩 가능한 값들에 전부 `BindingState` 프로퍼티 래퍼를 사용합니다.
```swift
struct State: Equatable {
    var isLoading = false
    @BindingState var digest = Digest.daily
    @BindingState var displayName = ""
    @BindingState var enableNotifications = false
    @BindingState var protectMyPosts = false
    @BindingState var sendEmailNotifications = false
    @BindingState var sendMobileNotifications = false
}
```
이러한 필드들은 picker, toggle, textfield 와 같은 SwiftUI 컨트롤에서 직접 바인딩할 수 있습니다.
이때 알아둬야 하는 것이, `isLoading` 프로퍼티는 바인딩 되지 않기 때문에 `BindingState` 어노테이션을 붙이지 않았다는 것입니다.
이는 뷰가 직접 해당 값을 변경하지 못하도록 합니다.
그 다음, `action` 타입이 `BindableAction` 을 합니다. 이를 위해 변경가능한 필드에 대한 개별 `action` 들은 `BindingAction` 제너릭을 갖는 하나의 단일 케이스로 합칩니다. 이 제너릭 타입은 `reducer` 의 `state` 를 갖습니다.
```swift
enum Action: BindableAction {
    case binding(BindingAction<State>)
}
```
그 다음, `BindingReducer` 를 사용하여 reducer를 더 간단히 할 수 있습니다.
```swift
struct Settings: ReducerProtocol {
    var body: some ReducerProtocol<State, Action> { 
        BindingReducer()
    }
}
```
> **메모**
> 
> 여러 예시 프로젝트들 보면서 뜬금없이 `Reducer` 에 `body` 가 있는 모습을 볼때마다 이게 뭐지 했는데 `Binding` 에 관한거라는 것을 알게되었다..!!!

바인딩하는 action 들은 `binding(_:fileId:line:)` 을 바인딩하는 `state` 의 key path와 함께 호출하여 구조화하고 `store`로 전송할 수 있습니다.
```swift
TextField("Display name", text: viewStore.binding(\.$displayName)
```
만약, 이러한 바인딩에 추가적인 기능을 더해야 한다면, `reducer` 는 주어진 key path에 대한 `action` 들을 패턴 매칭할 수 있습니다.
```swift
var body: some ReducerProtocol<State, Action> {
    BindingReducer()
    
    Reduce { state, action in
        case .binding(\.$displayName):
            //  validate display name
        case .binding(\.$enableNotifications):
            // 인증 요청 effect 리턴
    }
}
```
`action` 을 바인딩 하는 것은 일반 `action` 들이 테스팅 되는 것과 거의 동일한 방법으로 테스트 될 수 있습니다.
바인딩이 어떻게 바뀌었는지 설명하는 특정 `action` 을 전달하는 대신(예: `.displayNameChanged("Blob")`, 
key path 가 어느 값으로 세팅 될 지 설명하는 `BindingAction` 의 `action` 를 전달하면 됩니다. (예: `.set(\.$displayName, "Blob")`)
```swift
let store = TestStore(initialState: Settings.State()) {
    Settings()
}
store.send(.set(\.$displayName, "Blob")) {
    $0.displayName = "Blob"
}
store.send(.set(\.$protectMyPosts, true)) {
    $0.protectMyPosts = true
}
```





