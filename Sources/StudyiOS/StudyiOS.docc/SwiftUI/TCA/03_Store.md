# Part 3. Store

앱을 동작하는 런타임 자체를 나타내는 오브젝트. 앱과 소통하는 뷰들에 전달된다. (즉 뷰에 직접적으로 쓰인다)

## 왜 공부하는가?
`StoreOf` 를 보자마자 첫번째 든 생각은 네이밍이 뭐 이렇지? 였다. 
그러면 `Store` 도 있을 것 같은데(있다) 왜 공식 예제에서는 `Store` 가 아닌 `StoreOf` 를 쓸까 하는 생각이 들어서
이번엔 `Store`를 공부해보고자 한다.

> **궁금증에 대한 답변 미리보기** `StoreOf` 는 `Store` 선언시 `State`, `Action` 둘다 사용할 필요없이 `ReducerProtocol`를 준수하는 `Reducer`만 사용하는 `typealias`.

> **참고링크** [Store | Documentation](https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/store/)

## 선언

```swift
final class Store<State, Action>
```

여기서 `Store`는 `State`와 `Action`을 갖는다는 것을 알 수 있다.

## 개요

앱의 루트에서 하나의 Store 를 설정하는 것이 일반적이다.

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            RootView(
                store: Store(
                    initialState: AppReducer.State(),
                    reducer: { AppReducer() }
                )
            )
        }
    }
}
```
그런 다음 `scope(state:action:)` 메소드를 사용하여 하위 뷰에 전달할 수 있는 더 특화된 `Store` 들을 도출한다.

### Scoping ⭐️

`scope(state:action:)` 는 `Store` 에서 가장 중요한 동작이다. 
이 메소드는 하나의 `Store`를 하위 `State` 와 `Action`을 다루는 새로운 `Store`로 변형시켜준다.

예를 들어, 만약 앱이 탭뷰로 구성되어 있고, 각 탭에는 활동, 검색, 프로필이 있다고 가정해보자.
이 경우, 우리는 도메인을 다음과 같이 모델링 할 수 있다.

```swift
struct State {
    var activity: Activity.State  // State for 활동
    var profile: Profile.State    // State for 프로필
    var search: Search.State      // State for 검색
}

enum Action {
    case activity(Activity.Action)  // Action for 활동
    case profile(Profile.Action)    // Action for 프로필
    case search(Search.Action)      // Action for 검색
}
```

전체 앱 도메인에서 각 하위 도메인을 위한 Store로 변형하기 위해 `scope(state:action:)` 을 사용해보면 다음과 같다.

```swift
struct AppView: View {
    let store: StoreOf<AppReducer>
    
    var body: some View {
        TabView {
            ActivityView(
                store: store.scope(
                    state: \.activity
                    action: App.Action.activity
                )
            )
            .tabItem { Text("Activity") }
            
            SearchView(
                store: store.scope(
                    state: \.search
                    action: App.Action.search
                )
            )
            .tabItem { Text("Search") }
            
            ActivityView(
                store: store.scope(
                    state: \.profile
                    action: App.Action.profile
                )
            )
            .tabItem { Text("Profile") }
        }
    }
}
```

## Thread safety

⭐️ `Store` 클래스는 **thread-safe** 하지 않습니다.

⭐️ 때문에, `Store`(모든 스콥된 store들과 도출된 `ViewStore`들도 포함) 와의 모든 상호작용은 **store가 생성된 동일 쓰레드에서 완료**되어야 합니다.
SwiftUI 를 사용하는 경우 모든 상호작용들은 **메인**쓰레드에서 완료되어야 합니다.

`Store` 가 thread-safe 하지 않는 이유는 *store로 action이 전달될 때, reducer가 현재 state에서 돌고 있고* 이러한 프로세스로 인해 다중 쓰레드에서 완료될 수 없습니다.

물론, lock이나 queue를 사용하여 thread-safe 하게 처리할 수 있습니다. 하지만 매우 복잡합니다.
- 만약 `DispatchQueue.main.async` 를 사용하여 완료하는 경우, 이미 메인쓰레드에 있더라도 thread hop 이 일어날 수 있습니다. 이는 SwiftUI 에서 애니메이션 블록과 같은 sync한 작업이 필요할 때, 예상치 못한 동작을 야기할 수 있습니다.
- 메인쓰레드에 있으면 즉시 작업을 수행할 수 있도록 schduler를 생성할 수 있습니다. 그렇지 않은 경우 `DispatchQueue.main.async` 를 사용합니다. (Combine Scheduler 의 [UIScheduler](https://github.com/pointfreeco/combine-schedulers/blob/main/Sources/CombineSchedulers/UIScheduler.swift) 참고)

> **INFO** actor hop: 만약 어떤 쓰레드가 한 액터의 작업을 시작하기 위해 다른 액터의 작업을 중지하는 경우 이걸 actor hopping 이라고 부릅니다.

위와 같은 복잡성에 대한 이유로, 모든 action 들은 같은 쓰레드에서 전달되어야 합니다. 이러한 요구사항은 `URLSession` 과 다른 애플 API 가 디자인 되는 방식과 동일합니다.
이러한 API 들은 가장 편한 아무 쓰레드에 output을 전달하는 경향이 있기 때문에 이를 다시 메인 쓰레드로 보내는 것은 너희들 책임임~

⭐️ 이러한 디자인 방식에 따라, TCA 에서는 반드시 메인 쓰레드에서 전달되어야 합니다. 만약 output을 메인쓰레드가 아닌 다른 쓰레드에서 전달하고 있는
effect를 사용하고 있다면, 반드시 `.receive(on:)` 를 사용하여 강제로 메인쓰레드로 오도록 해야합니다.

# StoreOf ⭐️

편의성을 위한 `typealias`

```swift
typealias StoreOf<R> = Store<R.State, R.Action> where R: ReducerProtocol
```
`Store` 쓰려면 State, Action 둘다 지정해야하는데,
```swift
let store: Store<Feature.State, Feature.Action>
```
만약 `ReducerProtocol`를 준수하는 객체가 있다면 `StoreOf` 를 써서 아래와 같이 간단하게 쓸 수 있다.
```swift
let store: StoreOf<Feature>
```

















