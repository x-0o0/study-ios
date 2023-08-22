# Part 5. ViewStore

`ViewStore`는 `state` 변화를 지켜보고, `action`을 보내는 객체. 뷰에서 가장 흔히 사용되지만 뷰 외에도 state 를 옵저빙하고 action를 전달해야하는 곳 어디든 이유만 타당하다면 쓸 수 있다.

## 왜 공부하는가?

[02_이벤트_발생_시_Action_State_Reducer_동작과정](https://github.com/jaesung-0o0/TIL_iOS/blob/main/SwiftUI/TCA/02_%EC%9D%B4%EB%B2%A4%ED%8A%B8_%EB%B0%9C%EC%83%9D_%EC%8B%9C_Action_State_Reducer_%EB%8F%99%EC%9E%91%EA%B3%BC%EC%A0%95.md) 에서 한번 공부하긴 했지만
놓친 개념이 없는지 확인하기 위해 공식 문서의 `ViewStore` 부분을 정독하고자 한다.
겸사겸사 `@dynamicMemberLookup` 도 공부하자!

## 선언

```swift
@dynamicMemberLookup final class ViewStore<ViewState, ViewAction>
```

```swift
typealias ViewStoreOf<R: ReducerProtocol> = ViewStore<R.State, R.Action>
```

```swift
let viewStore: ViewStore<Feature.State, Feature.Action>
let viewStore: ViewStoreOf<Feature>
```

### @dynamicMemberLookup

`@dynamicMemberLookup` 키워드를 사용해서 선언되어있는데, 이게 뭔지부터 알아보자...

`@dynamicMemberLookup` 
- Swift4.2 +
- class, struct, enum, protocol 에 사용가능 (프로토콜 처럼)
- `subscript(dynamicMemberLookup:)` 메소드를 구현해야함
- 만약 이 키워드를 사용하는 타입이 있다면, 런타임시 확인되는 임의의 name에 대해 `.`(dot) syntax를 사용할 수 있도록 제공해준다.

```swift
@dynamicMemberLookup
struct SolarSystem {
    var planets: [String: String]
    
    subscript(dynamicMember koreanName: String) -> String? {
        return self.planets[koreanName]
    }
    
}
```

```swift
var system = SolarSystem(planets: [
    "지구": "Earth",
    "달": "Moon"
])

let earth = system[dynamicMember: "지구"]
let earth = system.지구 // 마치 프로퍼티 가져오는 거처럼
let mars = system.화성 // nil
```

`ViewStore` 에서는 현재 상태를 나타내는 `ViewState` 에 대해 key path 방식으로 `subscript(dynamicMember:)` 이 적용되어 있다

## 개요

SwiftUI 에서는 `WithViewStore` 라는 **뷰**를 통해서 `ViewStore` 에 접근하는 것이 가장 보편적.
`WithViewStore`는 view store 객체를 다루고 뷰를 리턴하는 **클로져**와 **store** 를 가지고 초기화합니다.
```swift
var body: some View {
    WithViewStore(store, observe: { $0 }) { viewStore in
        VStack {
            Button("Increment") { viewStore.send(.incrementButtonTapped) }
        }
    }
}
```
`view store` 는 View, Scene, Command 그리고 `@ObservedObject` 프로퍼티 래퍼를 지원하는 다른 모든 컨텍스트에서 옵저빙 됩니다.
```swift
@ObservedObject var viewStore: ViewStore<State, Action>
```
> **팁**
>
> 만약, WithViewStore를 사용하는 뷰에서 컴파일 에러가 발생하면, `@ObservedObject` 를 사용해서 직접 view store를 옵저빙 해보는 것을 추천합니다.
> 이는 컴파일러에게 더 쉽게 느껴질거 거든요.

> **중요**⭐️⭐️⭐️
> `ViewStore` 클래스는 thread-safe 하지 않습니다. 그리고 view store 와의 모든 인터랙션들은 반드시 동일한 쓰레드에서 발생해야합니다.
> SwiftUI 라면 모든 인터랙션은 **전부 `main` 쓰레드**에서 발생해야합니다.
> Thread safe 에 대한 추가적인 내용은 3번째 공부내용을 참고하면 됩니다: [링크](https://github.com/jaesung-0o0/TIL_iOS/blob/main/SwiftUI/TCA/03_Store.md#thread-safety)

## 생성자

### `init(_:observed:send:removeDuplicates:)`

state 변화를 감지하는 store 를 가지고 view store를 생성합니다.

```swift
init<State, Action>(
    _ store: Store<State, Action>,
    observe toViewState: @escaping (State) -> ViewState,
    send fromViewAction: @escaping (ViewAction) -> Action,
    removeDuplicates isDuplicate: @escaping (ViewState, ViewState) -> Bool
)
```
| 파라미터 | 설명 | 타입 |
| ---- | ---- | ---- |
| store | store 객체 | `Store<State, Action>` |
| toViewState | `ViewState` 를 변화를 옵저빙할 `state` 로 변형 | `(State) -> ViewState` |
| fromViewAction | 어떤 `action`을 전달 할 수 있을지 묘사하는 `ViewAction` 의 변형 | `(ViewAction) -> Action` |
| isDuplicated | 2개의 `State` 서로 일치하는 지 결정하는 함수. 두 값이 같다면 뷰의 중복 계산이 제거됩니다 | `(ViewState, ViewState) -> Bool` |

퍼포먼스를 위해 `observe` 에서 `state` 를 작업에 필요한 최소한의 데이터로 변환하는 것을 권장합니다. 상위 레벨의 feature에서는 가장 중요하고, 하위(leaf) feature 에 대해서는 그렇게 중요하진 않습니다.

### `init(_:observe:send:)`

```swift
convenience init<State, Action>(
    _ store: Store<State, Action>,
    observe toViewState: @escaping (State) -> ViewState,
    send fromViewAction: @escaping (ViewAction) -> Action
)
```
`ViewState` 가 `Equatable`를 준수할 때 사용가능합니다. (`removeDeuplicates` 가 없어졌으니까)

> **깨닫음** 아.. 어쩐지.. 자꾸 `Equatable` 준수하라더라...

## ViewState

- `ViewStore/state` 는 현재 상태를 나타내는 `ViewState` 값.
- `@dynamicMemberLookup` 의 `dynamicMember` 대상

## ViewAction

### ViewStoreTask

아래에서 서술될 `send(_:)` 메소드의 리턴 타입으로, `action`이 전달될 때를 시작으로하는 `effect`의 라이프사이클을 나타낸다.

```swift
struct ViewStoreTask
```
| 변수 및 메소드 | 설명 | 타입 |
| --- | --- | --- |
| `isCancelled` | task 가 실행을 멈춰야하는지 나타내는 `Bool` 값 | `Bool` |
| `cancel() async` | 주요 비동기 작업을 취소하고 종료될때까지 기다린다 | `() -> Void async` |
| `finish() async` | 작업이 끝나기를 기다린다 | `() -> Void async` |

SwiftUI 의 `task` view modifier 와 같은 비동기성 컨텍스트에서 `effect`의 라이프사이클과 `cancellation`을 연결 지을 때 사용할 수 있습니다.

```swift
.task { 
    await viewStore
        .send(.task) // `ViewStoreTask`
        .finish()    // `Cancellation`
}
```

> **정보**
> 
> Swift의 `Task`와는 달리, 
> `ViewStoreTask`는 자동으로 현재의 비동기 컨텍스트와 task 사이에 cancellation 핸들러를 설정한다.

### `send(_:)`

```swift
@discardableResult func send(_ action: ViewAction) -> ViewStoreTask
```

리턴 값인 `ViewStoreTask` 는 액션을 전달할 때 실행되는 `effect` 의 라이프사이클을 나타냄.

> **중요**
> `ViewStore` 는 thread-safe 하지 않기 때문에 (잊을만 하면 나오니까 이정도면 외우자) 반드시 action 들은 메인쓰레드에서 전달되어야합니다.
> 만약 `reducer`(`action` 에 따라 `state`를 변경하는 객체) 가 무거운 계산업무 수행중이어서 백그라운드 스레드에서 액션을 전달하고 싶다면,
> 이를 `EffectTask` 로 감싸면 됩니다. `EffectTask` 는 백그라운드 쓰레드에서 돌아가기 때문에 이렇게 하면 결과를 다시 `store`(State + Reducer 를 모두 아우르는 개념) 에 전달할 수 있습니다. 

### `send(_:while:)`

`store`에 `action` 을 전달하고 나서 `state` 일부가 `true`가 될때까지 작업을 중지함.

```swift
@MainActor
func send(
    _ action: ViewAction,
    while predicate: @escaping (ViewState) -> Bool
) async
```
| 파라미터 | 설명 | 타입 |
| --- | --- | --- |
| action | 액션 | `ViewAction` |
| predicate | 얼마나 오랫동안 메소드를 중지할 지 결정하는 ViewState 에 대한 조건함수 | `(ViewState) -> Bool` |

이 메소드는 async/await 코드와 함께 사용하여 작업이 effect에서 수행되는 동안 비동기 작업을 중지시킬 수 있습니다.
가장 대표적인 예시로는 SwiftUI 의 `refreshable` 메소드가 있습니다. 이 메소드는 작업이 수행되는 동안 스크린에 로딩 indicator를 보여줍니다.

사용 예시로, 리스트에서 pull-to-refresh 제스쳐를 했을 때 네트워크를 통해 어떤 데이터를 가져온다고 가정해보자.
이 기능을 위한 도메인과 로직은 다음과 같이 모델링 될 것이다.
```swift
struct Feature: ReducerProtocol {
  @Dependency(\.fetch) var fetch

  func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
    switch action {
    case .pulledToRefresh:
      state.isLoading = true
      return .run { send in
        await send(.receivedResponse(TaskResult { try await self.fetch() }))
      }

    case let .receivedResponse(result):
      state.isLoading = false
      state.response = try? result.value
      return .none
    }
  }
}

extension Feature {
  struct State: Equatable {
    var isLoading = false
    var response: String?
  }
}

extension Feature {
  enum Action {
    case pulledToRefresh
    case receivedResponse(TaskResult<String>)
  }
}
```
여기서 알아야할 것은, state 의 `isLoading` 을 계속 지켜볼 것이라는 것이다. 이렇게 함으로서 네트워크 응답이 언제 수행중인지를 알 수 있다.
그리고 응답결과를 뷰에서는 `List` 를 통해 보여줄 것이다. 
만약에 화면이 뜨면, `.refreshable`  view modifier 를 사용하여 pull-to-refresh 제스처를 사용할 수 있다.
```swift
struct MyView: View {
  let store: Store<State, Action>

  var body: some View {
    WithViewStore(self.store, observe: { $0 }) { viewStore in
      List {
        if let response = viewStore.response {
          Text(response)
        }
      }
      .refreshable {
        await viewStore.send(.pulledToRefresh, while: \.isLoading)
      }
    }
  }
}
```
여기서 `.refreshable`를 주목하자
```swift
.refreshable {
  await viewStore.send(.pulledToRefresh, while: \.isLoading)
}
```
`send(_:while:)` 메소드를 사용하여 `isLoading` 이 `true` 인 동안 비동기 작업을 중지시켰다.
이 값이 `false` 가 될 때, 메소드는 재개되어 다음으로 넘어가면서 `.refreshable` 에 게 작업이 완료 되었음을 알리고
화면에서 로딩 indicator 가 사라질 것이다.

### `yield(while:)`

```swift
@MainActor
func yield(while predicate: @escaping (ViewState) -> Bool) async
```

| 파라미터 | 설명 | 타입 |
| ---- | ---- | ---- |
| predeicate | `ViewState` 에 대한 조건함수로 이 메소드가 비동기 작업시 얼마나 오랫동안 중지될지 결정 | `(ViewState) -> Bool` |

만약 `view store` 에 `action` 을 전달함과 동시에 중지시키고 싶다면 `send(_:while:)` 를 사용하면 됨


















