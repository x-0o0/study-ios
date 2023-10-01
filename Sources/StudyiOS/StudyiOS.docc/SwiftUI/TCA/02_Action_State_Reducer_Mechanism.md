# Part 2. 이벤트 발생 시 Action, State, Reducer 의 동작과정

> **참고** https://0urtrees.tistory.com/359

TCA 설명에서 ObservableObject와 StateObject, ObservedObject 등이 언급되지 않아서, 
애플이 SwiftUI에 사용하라고 만든 것을 왜 안 쓰는데 과연 적합한 패턴인가? 아니면 TCA 내부에서 사용 중이라 개발단에서 필요없는 것인가?
하는 궁금증이 생겨 검색을 했다.


## View & Store

```swift
struct MyAppView: View {
    let store: StoreOf<TradeInfo>
}
````
> **정보** 각 View는 Store를 갖는다.

Store 는 State 와 Action 를 갖는다. 

(이전에 페이지에서 공부한 개념이므로 State 와 Action이 뭔지 따로 언급하지 않겠습니다)

| Input | Store | Output |
| ----- | ----- | ------ |
| User Interaction | Action -> Reducer -> State | Effect |
    
### WithViewStore

```swift
var body: some View {
    WithViewStore(store) { viewStore in
        // ...
    }
}
```
`WithViewStore` 
- `State` 변화에 따른 뷰 업데이트를 위해 사용.
- `Store` -> ObservableObject 인 `ViewStore` 로 변환

> **정보** `viewStore` 는 `ObservableObject` 를 채택하고 있다. 즉, 뷰 계산에 사용.

### State 변경을 위한 이벤트 받기

```swift
Button {
    viewStore.send(.reconnectWebSocket) // action case 전송
} label: {
    // Label
}
```

> **정보** `viewStore.send(_:)` 에 `Action`을 파라미터로 넣어 이벤트를 전송한다.

`viewStore.send(_:)` -> Action case 에 따라 Reducer 에서 적절한 동작 실행.

```swift
struct TradeInfo: ReducerProtocol {
    struct State: Equatable { … }
    enum Action: Equatable { … }
  
    func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        switch action {
        case .onAppear:
            return .concatenate(
                Effect(value: .setupLocalData), 
                Effect(value: .connectWebSocekt)
            )
        case .reconnectWebSocket:
            state.numberFactAlert = nil
            return .none
        }
    }
}
```

## Reducer

Reducer는 여러개가 있을 수 있다. e.g., AppReducer, MainReducer, A_Reducer, B_Reducer, C_Reducer, ...

이러한 Reducer들은 서로 분해하고 결합이 가능.

- pullback: local -> global 하게 사용할 수 있도록 변경
- combine: 하나의 reducer로 

